---
description: "Learn more about: Health Service settings"
title: Modify Health Service settings
manager: eldenc
ms.author: arduppal
ms.topic: article
author: arduppal
ms.date: 08/13/2021
---
# Modify Health Service settings

> Applies to: Azure Stack HCI, versions 22H2, 21H2, and 20H2; Windows Server 2022, Windows Server 2019, Windows Server 2016

The Health Service, first released in Windows Server 2016, improves the day-to-day monitoring and operational experience for clusters running Storage Spaces Direct.

Many of the parameters which govern the behavior of the Health Service are exposed as settings. You can modify these to tune the aggressiveness of faults or actions, turn certain behaviors on/off, and more.

Use the following PowerShell cmdlet to set or modify settings.

## Usage

```PowerShell
Get-StorageSubSystem Cluster* | Set-StorageHealthSetting -Name <SettingName> -Value <Value>
```

### Example

```PowerShell
Get-StorageSubSystem Cluster* | Set-StorageHealthSetting -Name "System.Storage.Volume.CapacityThreshold.Warning" -Value 70
```

## Common settings

Some commonly modified settings are listed below, along with their default values.

### Volume Capacity Threshold

```
"System.Storage.Volume.CapacityThreshold.Enabled"  = True
"System.Storage.Volume.CapacityThreshold.Warning"  = 80
"System.Storage.Volume.CapacityThreshold.Critical" = 90
```

### Pool Reserve Capacity Threshold

```
"System.Storage.StoragePool.CheckPoolReserveCapacity.Enabled" = True
```

### Physical Disk Lifecycle

```
"System.Storage.PhysicalDisk.AutoPool.Enabled"                             = True
"System.Storage.PhysicalDisk.AutoRetire.OnLostCommunication.Enabled"       = True
"System.Storage.PhysicalDisk.AutoRetire.OnUnresponsive.Enabled"            = True
"System.Storage.PhysicalDisk.AutoRetire.DelayMs"                           = 900000 (i.e. 15 minutes)
"System.Storage.PhysicalDisk.Unresponsive.Reset.CountResetIntervalSeconds" = 360 (i.e. 60 minutes)
"System.Storage.PhysicalDisk.Unresponsive.Reset.CountAllowed"              = 3
```

### Supported Components Document

The Health Service provides an enforcement mechanism to restrict the components used by Storage Spaces Direct to those on a Supported Components Document provided by the administrator or solution vendor. For more information, see [Supported Components Document](health-service-overview.md#supported-components-document).

### Firmware Rollout

```
"System.Storage.PhysicalDisk.AutoFirmwareUpdate.SingleDrive.Enabled"       = True
"System.Storage.PhysicalDisk.AutoFirmwareUpdate.RollOut.Enabled"           = True
"System.Storage.PhysicalDisk.AutoFirmwareUpdate.RollOut.LongDelaySeconds"  = 604800 (i.e. 7 days)
"System.Storage.PhysicalDisk.AutoFirmwareUpdate.RollOut.ShortDelaySeconds" = 86400 (i.e. 1 day)
"System.Storage.PhysicalDisk.AutoFirmwareUpdate.RollOut.LongDelayCount"    = 1
"System.Storage.PhysicalDisk.AutoFirmwareUpdate.RollOut.FailureTolerance"  = 3
```

### Platform / Quiescence

```
"Platform.Quiescence.MinDelaySeconds" = 120 (i.e. 2 minutes)
"Platform.Quiescence.MaxDelaySeconds" = 420 (i.e. 7 minutes)
```

### Metrics

```
"System.Reports.ReportingPeriodSeconds" = 1
```

### Debugging

```
"System.LogLevel" = 4
```

## Additional References

- [Health Service in Windows Server 2016](health-service-overview.md)
- [Storage Spaces Direct overview](/windows-server/storage/storage-spaces/storage-spaces-direct-overview)
