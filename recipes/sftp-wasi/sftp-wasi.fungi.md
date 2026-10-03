---
fungi: service/v1
id: sftp-wasi

run:
  provider: wasmtime
  source:
    url: https://github.com/enbop/sftp-wasi/releases/download/v0.1.0/sftp-wasi.wasm
  env:
    SFTP_BIND_HOST: 127.0.0.1
    SFTP_PORT: "2222"
    SFTP_FS_ROOT: appdata/files
    SFTP_HOST_KEY: appdata/host_ed25519
  mounts:
    - from: $fungi.service.data
      to: appdata

publish:
  sftp:
    tcp:
      port: 2222
    client:
      kind: ssh
---

# SFTP WASI

Runs [sftp-wasi](https://github.com/enbop/sftp-wasi) as a long-running WASIp2
command component with its own SSH TCP listener. Requires Fungi's run-only
Wasmtime provider and WASI guest environment support (merged Fungi PR #79;
the old Fungi v0.7.1 release is not compatible).

## Usage

After this recipe is included in a published catalog:

```bash
fungi service apply my-sftp --recipe sftp-wasi --refresh --dry-run
fungi service apply my-sftp --recipe sftp-wasi --refresh --start
fungi service inspect my-sftp --verbose
fungi service connect my-sftp sftp
```

Before catalog publication, apply this `.fungi.md` file by path instead of
using `--recipe sftp-wasi`. Use modern OpenSSH SCP, an SFTP client, or SSHFS
against the address printed by Fungi, replacing `HOST` and `PORT` below:

```bash
scp -P PORT local.txt demo@HOST:/remote.txt
sftp -P PORT demo@HOST
sshfs -p PORT demo@HOST:/ /path/to/empty-mountpoint
```

The experimental default username and password are both `demo`. The listener
binds to `127.0.0.1:2222` on the service device. For multiple instances, copy
the recipe and change both `SFTP_PORT` and `publish.sftp.tcp.port` to a free
port per instance.

`client.kind: ssh` describes the transport. This server provides SFTP only:
there is no remote shell, SSH exec, legacy `scp -O`, or `scp -R` support.
WASIp2 reports fixed file modes; chmod/chown are accepted as no-ops.

## Data and Access

Files persist in `$fungi.service.data/files`. On first start, the server creates
an Ed25519 host key at `$fungi.service.data/host_ed25519`, outside the SFTP file
root, and reuses it across service and daemon restarts. This recipe does not
expose `$fungi.workspace` or require a host-path allowlist change. Back up files
before removing the service.

This is an experimental password-authenticated server with public demo
credentials, intended for trusted local/Fungi access. Do not expose its port
directly to untrusted networks. To change credentials, use a private local copy
of the recipe with `SFTP_USERNAME` and `SFTP_PASSWORD` under `run.env`; do not
commit real credentials to the catalog.

## Source

- Project: <https://github.com/enbop/sftp-wasi>
- Artifact URL: <https://github.com/enbop/sftp-wasi/releases/download/v0.1.0/sftp-wasi.wasm>
- Published checksum: <https://github.com/enbop/sftp-wasi/releases/download/v0.1.0/sftp-wasi.wasm.sha256>
- WASM SHA-256: `8b68741a7df2b4d4245611ee1a5789976375e87e8affe091dc0ce23333b9880d`

The checksum above records the verified release artifact; it is not an
enforced field in the service manifest.
