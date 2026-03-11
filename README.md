# MECM Lab Fix

This repository contains the CM1 dual-NIC routing fix used to stabilize internet traffic while keeping private/corp connectivity.

## Files
- `fix-cm1-routing.ps1` - one-shot PowerShell fix script
- `fix-cm1-routing.README.md` - detailed breakdown and rollback guidance

## Quick Run
Run from an **Administrator PowerShell** session:

```powershell
powershell -ExecutionPolicy Bypass -File .\fix-cm1-routing.ps1
```

## Expected Result
- Only the internet NIC keeps the default route
- Internet NIC metric is lower than corp NIC metric
- Traceroute first hop is the internet gateway