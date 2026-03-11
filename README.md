# MECM Lab Fix

This repo contains a PowerShell fix for a dual-NIC CM1 VM where internet traffic was intermittently slow unless the private NIC was disconnected.

## Problem
The VM had two active NICs:
- `Ethernet` (corp/private network)
- `Ethernet 2` (internet/NAT network)

A stale default route on the private NIC and equal automatic interface metrics could cause Windows to evaluate the wrong path first. That introduced delays for internet-bound traffic.

## Fix Summary
The script [`fix-cm1-routing.ps1`](./fix-cm1-routing.ps1) does three things:

1. Removes the default route (`0.0.0.0/0`) from the private/corp NIC (`Ethernet`) in both:
   - Active route table
   - Persistent route table
2. Disables automatic interface metrics.
3. Pins manual metrics so internet is always preferred:
   - `Ethernet 2` (internet): metric `10`
   - `Ethernet` (corp/private): metric `500`

Result: internet-related traffic exits through the intended internet gateway instead of competing with the private path.

## Run
Use an **Administrator PowerShell** session:

```powershell
powershell -ExecutionPolicy Bypass -File .\fix-cm1-routing.ps1
```

## What Good Looks Like
After running, output should show:
- Only one default IPv4 route, on `Ethernet 2`
- Interface metrics:
  - `Ethernet 2` = `10`
  - `Ethernet` = `500`
- `tracert` to `8.8.8.8` first hop is the internet gateway (for this lab: `192.168.16.1`)

## Verify Manually
```powershell
Get-NetRoute -AddressFamily IPv4 -DestinationPrefix 0.0.0.0/0
Get-NetIPInterface -AddressFamily IPv4 | Sort-Object InterfaceMetric
tracert -d -h 4 8.8.8.8
```

## Rollback
```powershell
Set-NetIPInterface -InterfaceAlias "Ethernet 2" -AddressFamily IPv4 -AutomaticMetric Enabled
Set-NetIPInterface -InterfaceAlias "Ethernet"   -AddressFamily IPv4 -AutomaticMetric Enabled

# Optional: re-add private NIC default route only if your design requires it
New-NetRoute -DestinationPrefix "0.0.0.0/0" -InterfaceAlias "Ethernet" -NextHop "10.0.0.254" -PolicyStore PersistentStore
```

## Notes
- This fix targets routing/metric behavior only.
- If apps are still slow after this, check DNS forwarding/lookup behavior on your corp DNS path.
- Detailed script walkthrough: [`fix-cm1-routing.README.md`](./fix-cm1-routing.README.md)