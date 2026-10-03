# Fungi Service Recipes

Experimental service manifest recipes for [Fungi](https://github.com/enbop/fungi).

This repository is a lightweight collection of runnable references and starting
points. It is not a curated service hub, marketplace, or security-reviewed
registry. Recipes can change, break, disappear, or move as the Fungi service
manifest format evolves.

## Release Assets

Tagged releases publish static assets that can be fetched over HTTP:

- `index.json`: recipe metadata and manifest paths.
- `<recipe-id>.fungi.md`: a directly fetchable service file with YAML front matter and Markdown notes.
- `SHA256SUMS`: checksums for the published assets.

The source layout intentionally mirrors the release layout:

```text
recipes/
  <recipe-id>/
    <recipe-id>.fungi.md
index.json
```

## Current Recipes

- `filebrowser-lite`: File Browser Lite through the Wasmtime runtime.
- `sftp-wasi`: SFTP, modern SCP, and SSHFS through the Wasmtime runtime, with service-owned persistent files.
- `socks5-wasip2`: an unauthenticated SOCKS5 TCP proxy through the Wasmtime runtime.
- `ssh-tunnel`: an existing SSH daemon exposed as a TCP tunnel service.
- `webdav`: WebDAV through the Wasmtime runtime.

Recipes support managed WASI applications and existing host TCP services.

> Starting with Fungi 0.8.0, Docker container management is no longer supported.

Start a container or host
application yourself, publish its port on the target device's loopback interface,
and connect it through a TCP service such as:

```yaml
fungi: service/v1
id: host-web
publish:
  web:
    tcp:
      host: 127.0.0.1
      port: 8080
    client:
      kind: web
      path: /
```

Applying, stopping or removing this service controls Fungi access; it does not
start, stop or delete the host process.

## Wasmtime Service Upgrades

The filebrowser-lite `lite-v0.4.0` and WebDAV `v0.2.0` recipes use long-running
WASIp2 command components with their own TCP listeners. They are validated with
the run-only Fungi core from [Fungi #79](https://github.com/enbop/fungi/pull/79).

Refreshing the catalog does not update installed services. To replace an old
HTTP component, use a Fungi build containing that change and explicitly apply
the updated recipe under the existing instance name:

```bash
fungi service apply my-files --recipe filebrowser-lite --refresh --dry-run
fungi service apply my-files --recipe filebrowser-lite --refresh --start
fungi service apply my-dav --recipe webdav --refresh --dry-run
fungi service apply my-dav --recipe webdav --refresh --start
```

These commands use the latest **published** catalog; a merged recipe PR becomes
available there only after a catalog release is tagged and its assets are built.
Use a local `.fungi.md` file to test a candidate before publication.

Using the same instance name preserves its local service ID and appdata. Both
recipes expose `$fungi.workspace`; retain any custom mounts in a local service
file if an existing installation stores files elsewhere. Removing `run.mode`
alone does not upgrade an incoming-handler component into a command component.

## Local Build

Build and validate the static release payload:

```bash
scripts/build-release.sh
```

The generated files are written to `dist/`.

## Adding A Recipe

1. Add `recipes/<recipe-id>/<recipe-id>.fungi.md`.
2. Put the structured service definition in YAML front matter and the human/AI notes in the Markdown body.
3. Add the recipe entry to `index.json`.
4. Run `scripts/build-release.sh`.

Keep entries experimental unless there is an explicit compatibility and review
process behind them.
