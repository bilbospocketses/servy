# Roadmap — Servy (`bilbospocketses` fork)

> Hard fork of [aelassas/servy](https://github.com/aelassas/servy). This is the **fork's** roadmap. The architecture behind each epic is in **[VISION.md](VISION.md)**; the rationale is in **[NOTES.md](NOTES.md)**.
>
> ⚠️ None of the fork epics below have started. The fork today tracks upstream Servy 8.4 + repository hardening.

## Inherited baseline (upstream Servy 8.4)

Servy already does all of this **on Windows**, inherited from upstream — this fork has not changed it:

- Run any app as a Windows service (working directory, startup type, priority, env vars, dependencies; Local System / local / domain / AD / gMSA accounts)
- Desktop app + Manager app + CLI + PowerShell module
- stdout/stderr logging with size- and date-based rotation; health checks + automatic recovery; pre-/post-launch and pre-/post-stop hooks
- Real-time CPU/RAM graphs, live console streaming, dependency-tree visualization
- Export/import configuration; failure notifications (Windows toast + email); package-manager distribution (WinGet / Chocolatey / Scoop)

For the complete upstream feature list and history, see [upstream's roadmap](https://github.com/aelassas/servy/blob/main/ROADMAP.md) and [CHANGELOG.md](CHANGELOG.md).

## Fork roadmap

Three epics. See [VISION.md](VISION.md) for the architecture and open questions behind each.

### Epic 1 — Cross-platform (Linux systemd, macOS launchd)
- [ ] Re-target `Servy.Core` / `Servy.Service` / `Servy.CLI` from `net10.0-windows` to a platform-neutral `net10.0` TFM
- [ ] Introduce a service-host abstraction over the Win32 SCM layer (`Servy.Core/Native/NativeMethods.cs`, `Services/WindowsServiceApi.cs`)
- [ ] systemd backend (Linux)
- [ ] launchd backend (macOS)
- [ ] Abstract Windows Event Log (`Logging/EventLogReader.cs`) and AD/gMSA identity behind platform-neutral logging/identity seams
- [ ] Decide the UI story (WPF apps stay Windows-only vs. a cross-platform toolkit vs. CLI-first elsewhere)

### Epic 2 — Multi-architecture (arm64)
- [ ] Add the `win-arm64` RID; then `linux-arm64` / `osx-arm64` alongside the cross-platform work
- [ ] Produce per-architecture builds of the embedded service/restarter helpers and `handle.exe` utility
- [ ] Resolve AOT and code-signing implications for the new architectures

### Epic 3 — Velopack integration
- [ ] Decide the model: Servy as a Velopack subsystem vs. independent Servy + a Velopack adapter
- [ ] Unify install/update (Velopack) with service/daemon management (Servy)
- [ ] Coordinate with the [velopack fork](https://github.com/bilbospocketses/velopack)

## Out of scope (deferred)

- **Packaging rebrand** (WinGet/Chocolatey/Scoop manifests, Inno Setup scripts, build scripts → fork identity) — gated on the fork having its own releases and signing. Until then, released binaries come from upstream.
