# Why this fork?

> **Servy — `bilbospocketses` fork.** A hard fork of [aelassas/servy](https://github.com/aelassas/servy) by Akram El Assas. This document explains why the fork exists and what it is trying to become. For the architecture-of-record see **[VISION.md](VISION.md)**; for the plan see **[ROADMAP.md](ROADMAP.md)**.

## The upstream project

Servy is an excellent Windows service wrapper. Akram El Assas built it to solve the real limitations of `sc.exe`, NSSM, and WinSW — custom working directories, monitoring and health checks, pre-/post-launch and pre-/post-stop hooks, a clean desktop UI, CLI and PowerShell automation, a Manager app, and service-dependency-tree visualization. It is a genuinely capable, actively maintained tool.

Upstream Servy is also, by design and by its maintainer's clear and consistent signal, **Windows-only**: it is built on the Windows Service Control Manager, Active Directory / gMSA accounts, the Windows Event Log, and x64. That is its identity, not an oversight — it aims to be the best service wrapper *on Windows*, and it succeeds.

## Why fork it?

This fork wants three things that upstream will not pursue. It takes the same well-built service-wrapper core and points it off the Windows-only island:

### 1. Cross-platform
The same robust install-as-service, monitor, recover, and restart model — but on **Linux (systemd)** and **macOS (launchd)**, not only Windows. A background app should be manageable the same way wherever it runs.

### 2. Multi-architecture
**arm64**, alongside the existing x86/x64 — for ARM servers, Apple Silicon, and ARM Windows.

### 3. Merge with Velopack
Servy manages the *running* side of a background app — install it as a service/daemon, watch it, recover it. [Velopack](https://github.com/bilbospocketses/velopack) (a sister hard fork) manages the *delivery* side — package, install, and update. Those are two halves of "ship and run a background app." The fork's long horizon is to merge them into a single tool that both delivers and runs.

## Relationship and approach

This fork respects upstream and tracks it closely. Where it finds genuinely upstreamable fixes, it will contribute them back through upstream's preferred channels. But the cross-platform / multi-arch / Velopack vision has no home upstream — it lives only here. There is no expectation that upstream adopts this direction, and none is asked of it.

## What this is not (yet)

To be honest about status: **none of the vision has shipped.** Today the fork is code-identical to upstream **Servy 8.4** plus repository hardening. Released, signed Windows binaries still come from upstream. These documents describe a direction and an architecture-of-record — not capability the fork has today. See [VISION.md](VISION.md) for what stands between here and there.
