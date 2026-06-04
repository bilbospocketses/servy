# Servy fork — Scoop packaging (local notes)

> **Packaging rebrand is deferred** — see [ROADMAP.md](../../ROADMAP.md) and [VISION.md](../../VISION.md). `servy.json` here targets **upstream** Servy; the fork publishes no Scoop manifest yet. A fork Scoop identity is gated on the fork having its own releases.

## Local test
```
scoop install .\servy.json
scoop uninstall servy
```

Fix encoding if needed:
```powershell
[System.IO.File]::WriteAllText("servy.json", [System.IO.File]::ReadAllText("servy.json"), (New-Object System.Text.UTF8Encoding))
```
