[![build](https://github.com/bilbospocketses/servy/actions/workflows/build.yml/badge.svg?branch=vnext)](https://github.com/bilbospocketses/servy/actions/workflows/build.yml)
[![test](https://github.com/bilbospocketses/servy/actions/workflows/test.yml/badge.svg?branch=vnext)](https://github.com/bilbospocketses/servy/actions/workflows/test.yml)
[![License](https://img.shields.io/github/license/bilbospocketses/servy)](LICENSE.txt)

# Servy &nbsp;<sub>·&nbsp;`bilbospocketses` fork</sub>

> **Hard fork of [aelassas/servy](https://github.com/aelassas/servy) by Akram El Assas**, extending Servy toward **cross-platform** operation, **multi-architecture** support, and **Velopack integration**. This fork's released Windows binaries still come from upstream — see [Relationship to upstream](#relationship-to-upstream).
>
> 📐 Direction lives in **[VISION.md](VISION.md)** (architecture-of-record) · **[ROADMAP.md](ROADMAP.md)** (plan) · **[NOTES.md](NOTES.md)** (rationale).

Servy lets you run any app as a native Windows service with full control over the working directory, startup type, process priority, logging, health checks, environment variables, dependencies, pre-/post-launch and pre-/post-stop hooks, and parameters. It is a full-featured alternative to NSSM, WinSW, and FireDaemon Pro, with a desktop app, a CLI, a PowerShell module, and a Manager app for monitoring services in real time.

That capability is **inherited from upstream Servy** — this fork has not changed it. What the fork adds is a *direction*: take Servy off the Windows-only island.

## Why this fork?

Upstream Servy is, by design and by its maintainer's stated identity, a **Windows-only** product — Windows Service Control Manager, Active Directory / gMSA accounts, the Windows Event Log, x64 only. It is excellent at that, and there is no upstream appetite for changing it.

This fork exists to take the same well-built service-wrapper core somewhere upstream's roadmap does not go:

1. **Cross-platform** — run the same service/daemon manager on **Linux (systemd)** and **macOS (launchd)**, not just Windows.
2. **Multi-architecture** — **arm64** alongside the existing x86/x64.
3. **Velopack integration** — merge packaging/install/update (**[Velopack](https://github.com/bilbospocketses/velopack)**, a sister fork) with service/daemon management (Servy) into a single tool.

The detailed architecture-of-record — what is Windows-bound today, what each target looks like, and the open questions — is in **[VISION.md](VISION.md)**. See **[NOTES.md](NOTES.md)** for the longer rationale.

> ⚠️ **Status — direction, not shipped capability.** This fork currently tracks upstream **Servy 8.4** and adds only repository hardening. **None** of the cross-platform / multi-arch / Velopack work has landed yet. These documents describe where the fork is going, not what it does today.

## Relationship to upstream

- **Code:** This fork is **code-identical to upstream Servy 8.4** (upstream `main` as of 2026-05-19), plus a single repository-lockdown commit. No product behavior has diverged yet.
- **Binaries:** **Released, code-signed Windows binaries come from upstream** — [aelassas/servy releases](https://github.com/aelassas/servy/releases) (signed by the SignPath Foundation) and the public **WinGet / Chocolatey / Scoop** packages, which are published under upstream's identity. **This fork has not published its own releases or packages.**
- **Docs:** The shared Windows feature set is documented in [upstream's wiki](https://github.com/aelassas/servy/wiki). Fork-specific direction lives in this repo (VISION / ROADMAP / NOTES / CHANGELOG).
- To run **this fork's** tree, [build from source](#getting-started-build-from-source).

## Getting started (build from source)

This fork publishes no binaries yet, so run it from source with the **.NET 10 SDK** (the exact pinned version is in `global.json`):

```powershell
git clone https://github.com/bilbospocketses/servy.git
cd servy
dotnet build Servy.sln -c Release
```

Build outputs land under each project's `bin/Release/` directory (the desktop app, `Servy.Manager`, the CLI, and the Windows service host). Everything targets `net10.0-windows` / `win-x64` today — see [VISION.md](VISION.md) for why, and what it takes to change that.

If you just want **Servy on Windows** (signed, released), use **upstream** instead:

```powershell
winget install servy      # installs upstream's signed build, not this fork
```

(or Chocolatey / Scoop — see upstream's [Installation Guide](https://github.com/aelassas/servy/wiki/Installation-Guide).)

## Quick example (CLI)

Servy's CLI (inherited from upstream) runs any app as a Windows service. For example, a Node.js server:

```powershell
servy-cli install `
  --name="MyService" `
  --path="C:\Program Files\nodejs\node.exe" `
  --startupDir="C:\MyServer" `
  --params="C:\MyServer\server.js"

servy-cli start --name="MyService"
```

More recipes for Python, Java, Go, and other stacks are in upstream's [Examples & Recipes](https://github.com/aelassas/servy/wiki/Examples-&-Recipes).

## What Servy does (inherited baseline)

The full Windows feature set carried over from upstream Servy 8.4:

* Clean desktop UI, plus a **Manager** app to monitor and manage all installed services
* Real-time CPU/RAM monitoring with live graphs; live stdout/stderr **Console**; service **dependency tree** visualization
* CLI and PowerShell module for scripting and CI/CD
* Run any executable as a Windows service with custom name, description, startup type, priority, working directory, environment variables, and dependencies
* Environment-variable expansion in parameters, process paths, and startup directories
* Run as Local System, local/domain accounts, Active Directory accounts, or gMSAs
* stdout/stderr redirection with size- and date-based log rotation
* Pre-launch, post-launch, pre-stop, and post-stop hooks (retries, timeout, failure handling)
* `Ctrl+C` for console apps (with descendant propagation), close-window for GUI apps, force-kill if unresponsive; orphan/zombie prevention
* Health checks and automatic recovery; log browse/search by level, date, keyword
* Export/Import service configurations; failure notifications via Windows toast and email
* Compatible with Windows 7–11 x64 and Windows Server editions

Full documentation for these features lives in [upstream's wiki](https://github.com/aelassas/servy/wiki).

## Documentation

| Fork direction (this repo) | Shared feature set (upstream) |
|---|---|
| [VISION.md](VISION.md) — architecture-of-record | [Installation Guide](https://github.com/aelassas/servy/wiki/Installation-Guide) · [Overview](https://github.com/aelassas/servy/wiki/Overview) |
| [ROADMAP.md](ROADMAP.md) — the plan | [Usage](https://github.com/aelassas/servy/wiki/Usage) · [FAQ](https://github.com/aelassas/servy/wiki/FAQ) |
| [NOTES.md](NOTES.md) — rationale | [CLI](https://github.com/aelassas/servy/wiki/Servy-CLI) · [PowerShell](https://github.com/aelassas/servy/wiki/Servy-PowerShell-Module) |
| [CHANGELOG.md](CHANGELOG.md) — fork + upstream history | [Examples & Recipes](https://github.com/aelassas/servy/wiki/Examples-&-Recipes) |

## Attribution

Servy was created by **[Akram El Assas](https://github.com/aelassas)** and remains an actively maintained upstream project at [aelassas/servy](https://github.com/aelassas/servy). This repository is an independent hard fork; all original copyright is retained.

Upstream's released binaries are code-signed by the [SignPath Foundation](https://signpath.org/). If you find Servy valuable, **support the original author**: [GitHub Sponsors](https://github.com/sponsors/aelassas) · [PayPal](https://www.paypal.me/aelassaspp) · [Buy Me a Coffee](https://www.buymeacoffee.com/aelassas).

## License

[MIT](LICENSE.txt) — original copyright retained from upstream Servy.
