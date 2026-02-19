# Thermal SELinux Fix for iKKO MindOne

Magisk module that fixes a vendor SELinux policy bug on the iKKO MindOne (Skyroam/Solis platform, MediaTek MT6789) preventing `thermal_core` from reading thermal zones 13–15.

## Problem

The vendor created custom SELinux types for three thermal zone sysfs directories and their temp files, but never added allow rules for `thermal_core` to access them:

| Zone | Type | SELinux dir type | SELinux file type |
|------|------|-----------------|-------------------|
| 13 | md | `sys_thermalzone_o` | `sys_md_ntcinfo` |
| 14 | ap_ntc | `sys_thermalzone_p` | `sys_ap_ntcinfo` |
| 15 | ltepa_ntc | `sys_thermalzone_q` | `sys_pa_ntcinfo` |

This causes `thermal_core` to fail reading these zones at startup, logging `Failed to find thermal zone index of ap_ntc` every ~3 seconds. AP thermal throttling, modem thermal protection, and LTE PA thermal protection are all non-functional as a result.

## Fix

The module adds 6 SELinux allow rules via Magisk's `sepolicy.rule`:

```
allow thermal_core sys_thermalzone_o dir { search read getattr open }
allow thermal_core sys_thermalzone_p dir { search read getattr open }
allow thermal_core sys_thermalzone_q dir { search read getattr open }
allow thermal_core sys_md_ntcinfo file { read getattr open }
allow thermal_core sys_ap_ntcinfo file { read getattr open }
allow thermal_core sys_pa_ntcinfo file { read getattr open }
```

## Install

1. Download `thermal_selinux_fix.zip` from [Releases](../../releases)
2. Install via Magisk app or `magisk --install-module thermal_selinux_fix.zip`
3. Reboot

## Tested on

- iKKO MindOne firmware v2.112.5.92(1204)
- Android 15 (SDK 35)
- Magisk 30.6
