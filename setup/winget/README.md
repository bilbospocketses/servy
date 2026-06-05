# Servy fork — WinGet packaging (local notes)

> **Packaging rebrand is deferred** — see [ROADMAP.md](../../ROADMAP.md) and [VISION.md](../../VISION.md). The manifests under `manifests/a/aelassas/Servy/` are **upstream's** (publisher `aelassas.Servy`) and are kept as reference. A fork WinGet identity is gated on the fork having its own releases and signing.

## Validate / test against the upstream manifests
```
winget validate .\manifests\a\aelassas\Servy\8.3\
winget install --manifest .\manifests\a\aelassas\Servy\8.3\
```
