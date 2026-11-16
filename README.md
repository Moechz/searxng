# searxng-for-tos

> TerraMaster TOS 7 App Center — **Docker application** package for
> [SearXNG](https://github.com/searxng/searxng).

## What this is

A four-file TOS Docker application archive (`searxngdocker.tar.gz`) that installs
SearXNG on TOS 7 through the App Center:

- `config.ini` — TOS application metadata (`application_type: docker`, id `searxngdocker`)
- `searxngdocker.lang` — 23-language superset (TOS requires 14)
- `searxngdocker.svg` — application icon (657 bytes, 3 SVG elements)
- `docker-compose.yml` — one service, image `searxng/searxng:2026.11.8-74418092d`

## How it works

| | |
|---|---|
| Container image | `searxng/searxng:2026.11.8-74418092d` (Docker Hub, used unmodified) |
| Open | `http://<NAS-IP>:8036/` (container port 8080) |
| Runs as | non-root, `user: "1000:1000"` |
| Persistent data | `/etc/searxng` → `/Volume1/DockerAppData/searxngdocker/config` |
| Logs | `docker logs searxngdocker` (stdout/stderr) |

`/etc/searxng` (mounted from `DockerAppData/searxngdocker/config`) holds `settings.yml`: the instance name, the generated `secret_key`, the enabled engines and the limiter switch.

## Privacy

This application runs entirely on your NAS. **The developer does not collect, receive or
have access to any data processed by the application.** There is no telemetry, no analytics
and no account.

| Topic | Detail |
|---|---|
| Data processed | The application's configuration, database, caches and logs, plus any files you create with it. |
| Where it is stored | The application data folder (`/Volume1/DockerAppData/searxngdocker/`) on your NAS. Nothing is uploaded to the developer. |
| Retention period | Until **you** delete it. Logs are rotated or size-limited by the application; uninstalling keeps the data unless you tick “delete data”. |
| Your rights | **Access / export**: copy the data folder. **Correct**: change the values in the web UI. **Delete**: uninstall with “delete data”, or remove the folder after stopping the container. |
| Deletion channel | Delete the folder yourself, or email **moechz@163.com** (answered within 30 days). |
| Security measures | Container runs as uid 1000 (never root), no `privileged`, no host network, no `cap_add`, no Docker socket, one published port; image tag pinned and refreshed on upstream fixes. |
| Children | Not directed at children. |

**Full privacy policy:** [PRIVACY.md](PRIVACY.md) —
<https://github.com/Moechz/searxng/blob/main/PRIVACY.md> — and the website version at
<https://www.moechz.com/privacy.html>.

## Third-party services (API disclosure)

The developer operates no server in this path. The application contacts only the services
**you configure**, plus (optionally) an update check and the image registry.

| Service | Endpoint | Data sent | Destination / region |
|---|---|---|---|
| Services you configure in the app | the address you enter | only the data that feature needs | normally your own LAN or the provider you chose |
| Update check (if enabled) | the upstream project's release feed | current version string | GitHub, US |
| Image pull | `registry-1.docker.io` or your registry mirror | anonymous pull | Docker Hub, US / your mirror |

See `PRIVACY.md` §6 for the full table. Disable any integration in the app's settings if you
do not want it.

## Licence and attributions

| Component | Licence | Notes |
|---|---|---|
| SearXNG | AGPL-3.0 | upstream application, used unmodified inside the image |
| Container image `searxng/searxng:2026.11.8-74418092d` | see the image's own metadata | pulled from Docker Hub at install time |
| This packaging | GPL-3.0 (see `LICENSE`) | compose file, metadata, translations, icon |

“SearXNG”, “TerraMaster” and “TOS” are trademarks of their respective owners. This is an
independent community package by Moechz and is not affiliated with the upstream project or
TerraMaster. Full details: [NOTICE.md](NOTICE.md).

## Design highlights

- **Non-root.** `user: "1000:1000"`; verified on real hardware (the container reports uid 1000).
- **Fixed image tag, never `:latest`**, Docker Hub only, no `privileged`, no `network_mode: host`, and no Docker socket, no `cap_add`.
- **Health check** probing the service on container port 8080.
- **Deterministic build**: GNU tar with uid/gid 0 and zeroed timestamps, gzip without a timestamp, plus the review-standards self-check above.
- **Per-install secret.** The image generates `secret_key` into the mounted settings file on first start, so no secret is baked into the package.
- **Limiter off for LAN use.** `SEARXNG_LIMITER=false` keeps a family from hitting 429s; re-enable it before any public exposure.
- **No external services of its own.** SearXNG scrambles the request through the engines you enable; nothing is stored outside the container.

## Runtime file manifest

Everything the application writes on the NAS lives inside its own data
directory; the container-internal `/tmp` is discarded with the container. The
table lists what is created and how it behaves. Nothing else is written.

| Path | Purpose | Created when | Growth bound | Lifecycle |
|---|---|---|---|---|
| `/Volume1/DockerAppData/searxngdocker/config` | SearXNG application data (everything the container writes outside /tmp) | on first start / on demand | bounded by your own data | persistent; kept on uninstall unless you ask to delete it |

## Build

```bash
scripts/build.sh                  # -> out/searxngdocker.tar.gz + .sha256
scripts/build.sh 2026.11.8-2   # packaging iteration bump
scripts/build.sh 2026.11.8-1 aarch64
```

The build stages the four files, substitutes the version/platform placeholders,
removes CRLF/BOM/AppleDouble, then runs
`tools/verify.py`, a review-standards self-check (archive layout, JSON validity,
required fields, 14+ language coverage, version consistency, Docker-Hub-only
images, fixed tag, reserved/recommended host ports, non-root user, per-service
healthcheck/TZ/restart, data path below the application data root, never the
`/Volume*` wildcard, comment-free sequence blocks, `x-app-meta` position, no
literal secrets and the SVG size/element limits).

## Local verification

```bash
./TOSAppSelfTestingTool -c out/searxngdocker.tar.gz   # official spec check
bash test/install-and-verify.sh                  # on-device E2E (needs root)
```

> Docker applications cannot be side-loaded through the App Center's manual
> install page (it accepts `.deb` only); the official self-testing tool's `-i`
> mode or `docker compose` are the only ways to install one for testing.

## Store submission checklist

1. Public GitHub repo (code + README only).
2. Release **tag `v2026.11.8-1`**, assets `searxngdocker.tar.gz` +
   `searxngdocker.tar.gz.sha256` (Docker asset naming: `<app_id>.tar.gz`, **no
   platform suffix**).
3. Developer platform → Add Application: ID `searxngdocker`, package type Docker,
   repository URL, architecture `x86_64`.
4. Version Management → Add Version `2026.11.8-1` → automated validation → review.


## Limitations

- The web UI is plain HTTP; use a TLS reverse proxy before exposing it to the internet.
- With the limiter disabled, a public instance would be easy to abuse - only do that behind auth.
- Some engines are blocked or rate-limited from certain networks (the log lists the ones that failed to register); editing `settings.yml` lets you drop them.

---

Packaged for TOS 7 by Moechz. Upstream image used unmodified from Docker Hub.

## Support

- Issues and questions: <https://github.com/Moechz/searxng/issues>
- Email: moechz@163.com
- Privacy: [PRIVACY.md](PRIVACY.md) · Licence: [NOTICE.md](NOTICE.md)
