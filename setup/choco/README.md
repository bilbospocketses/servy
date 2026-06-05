# Servy fork — Chocolatey packaging (local notes)

> **Packaging rebrand is deferred** — see [ROADMAP.md](../../ROADMAP.md) and [VISION.md](../../VISION.md). The fork publishes no Chocolatey package yet; the commands below reference **upstream** Servy's package identity and are kept as local-test reference. A fork Chocolatey identity is gated on the fork having its own releases and signing.

## Local test (pack + install from a local source)
```
choco pack
choco install servy -s . -y
choco uninstall servy -s . -y
```

## Inspect Servy uninstall entries
```powershell
Get-ItemProperty `
  HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*, `
  HKLM:\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*, `
  HKCU:\Software\Microsoft\Windows\CurrentVersion\Uninstall\* |
  Where-Object { $_.DisplayName -like "Servy*" } |
  Select-Object DisplayName, DisplayVersion, UninstallString | Format-Table -AutoSize
```
