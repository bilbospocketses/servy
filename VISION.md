# Servy fork — Vision & Architecture

> **Servy — `bilbospocketses` fork.** Hard fork of [aelassas/servy](https://github.com/aelassas/servy) by Akram El Assas.
> This is the fork's **architecture-of-record**: where Servy is headed and what stands in the way. The plan is in [ROADMAP.md](ROADMAP.md); the rationale is in [NOTES.md](NOTES.md).
>
> ⚠️ **Forward-looking.** None of the target architecture below has been implemented. The fork today is upstream **Servy 8.4** + repository hardening.

## Mission

Take Servy's well-built service/daemon-management core off the Windows-only island: run it **cross-platform** (Linux, macOS), on **multiple architectures** (arm64), and eventually **merge it with Velopack** so a single tool both *delivers* and *runs* a background app.

## Current architecture (the starting point)

Servy is a .NET solution of eight source projects. All of it is Windows-bound today:

| Layer | Projects | Windows binding |
|---|---|---|
| Service host | `Servy.Service`, `Servy.Restarter` | Win32 Service Control Manager |
| Core | `Servy.Core` | Win32 SCM (`Native/NativeMethods.cs`, `Services/WindowsServiceApi.cs`), Windows Event Log (`Logging/EventLogReader.cs`), AD/gMSA accounts |
| Infrastructure | `Servy.Infrastructure` | SQLite (portable) — but `net10.0-windows` TFM |
| CLI | `Servy.CLI` | no inherent UI dependency — but `net10.0-windows` TFM |
| Desktop / UI / Manager | `Servy`, `Servy.UI`, `Servy.Manager` | **WPF** (Windows-only) |

Two hard facts are visible directly in the project files:

- **Every** project sets `TargetFramework = net10.0-windows` — including `Servy.Core` and `Servy.CLI`, which have no inherent Windows-UI dependency.
- **Every** project hardcodes `RuntimeIdentifier = win-x64`.

So the portability barrier is not just the obvious WPF apps — it is baked into the TFM and RID of the entire stack, including the core. Native service/restarter helpers are committed as build resources (e.g. `Servy.Service.exe`, `Servy.Restarter.exe` under each project's `Resources/`); a `handle.exe` utility is used for file-handle inspection. All x64 today.

## Dimension 1 — Cross-platform

**Today's barrier.** The service lifecycle is Win32 SCM — install / start / stop / query via `advapi32` P/Invoke in `NativeMethods` and `WindowsServiceApi`. Recovery and observability use the Windows Event Log (`EventLogReader`). Accounts assume Local System / domain / Active Directory / gMSA. The TFM is `net10.0-windows`.

**Target.**
- Re-target `Servy.Core` / `Servy.Service` / `Servy.CLI` to a platform-neutral `net10.0`.
- Introduce a service-host abstraction (e.g. `IServiceHost`) with backends for Win32 SCM (existing), **systemd** (Linux), and **launchd** (macOS).
- Abstract Event-Log logging behind the existing logging interfaces, with a journald / syslog / file backend off Windows.
- Abstract identity (AD/gMSA → Unix users/groups).
- WPF apps (`Servy`, `Servy.UI`, `Servy.Manager`) stay Windows-only for now; a cross-platform UI (Avalonia/MAUI) or a CLI-first experience on Linux/macOS is a later decision.

**Open questions.** How much of `Servy.Core` is cleanly portable versus entangled with Win32? Port `Core` in place behind interfaces, or build a cross-platform sibling library and bridge to it? (See [ROADMAP.md](ROADMAP.md).)

## Dimension 2 — Multi-architecture (arm64)

**Today's barrier.** `RuntimeIdentifier = win-x64` is hardcoded in every project. The embedded native helpers and the `handle.exe` utility are x64.

**Target.** Add `win-arm64`, then `linux-arm64` / `osx-arm64` alongside the cross-platform work. Produce per-architecture builds of the embedded helpers; resolve AOT constraints; extend code-signing to the new architectures.

**Open questions.** AOT versus framework-dependent for the native helpers; how to vendor or produce per-architecture native dependencies without bloating the repository.

## Dimension 3 — Velopack integration

Servy runs the background app; [Velopack](https://github.com/bilbospocketses/velopack) — a sister hard fork — delivers and updates it. The horizon is to merge the two halves so one tool installs-as-service/daemon **and** self-updates.

**Models under consideration.**
- Servy becomes a Velopack subsystem (Velopack owns delivery and calls Servy for service/daemon install).
- Servy stays independent, with a Velopack adapter bridging the two.

The other half of this conversation lives in the [velopack fork](https://github.com/bilbospocketses/velopack)'s own system-aware-context work.

## Upstream relationship

Upstream Servy is actively maintained and unapologetically Windows-only — that is its identity, not an oversight. This fork tracks upstream and will contribute genuinely-upstreamable fixes back where appropriate; the cross-platform / multi-arch / Velopack vision lives only here, with no expectation that upstream adopts it.

## Packaging

The fork publishes no binaries or packages yet. Released, signed Windows binaries and the WinGet / Chocolatey / Scoop packages come from upstream, under upstream's identity. A fork packaging identity (manifests, installers, signing) is deferred until the fork has its own releases — see [ROADMAP.md](ROADMAP.md).
