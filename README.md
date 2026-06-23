# Windows 11 Nested Virtualization Fix

Script and guide to resolve nested virtualization conflicts caused by VBS (Virtualization-Based Security) and Hyper-V on Windows 11, enabling Type 2 hypervisors such as VMware Workstation and VirtualBox to use hardware-assisted virtualization.

## Problem

Windows 11 retains hardware virtualization extensions (VT-x / AMD-V) at ring 0 through VBS and Core Isolation, even when Hyper-V is disabled via optional features. This prevents Type 2 hypervisors from activating nested virtualization.

## Automated Fix

1. Download `disable-hypervisor.bat`.
2. Right-click the file and select **Run as Administrator**.
3. Follow the on-screen prompt to manually disable Memory Integrity.
4. Restart the system.

## Manual Fix

### Step 1 — Disable Memory Integrity

Open **Windows Security > Device Security > Core Isolation Details** and toggle **Memory Integrity** off.

### Step 2 — Remove Hyper-V features

```cmd
dism /online /disable-feature /featurename:Microsoft-Hyper-V-All
dism /online /disable-feature /featurename:VirtualMachinePlatform
dism /online /disable-feature /featurename:HypervisorPlatform
dism /online /disable-feature /featurename:Microsoft-Windows-Subsystem-Linux
```

### Step 3 — Modify bootloader and registry

```cmd
bcdedit /set hypervisorlaunchtype off
reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\DeviceGuard" /v "EnableVirtualizationBasedSecurity" /t REG_DWORD /d 0 /f
```

Restart after completing all steps.

## Verification

After restarting, run:

```cmd
systeminfo
```

Confirm one of the following:

- **Virtualization-based security status** reports `Not enabled`, or
- **Hyper-V Requirements** lists all four entries as `Yes` with no `A hypervisor has been detected` message.

## Enabling Nested Virtualization in VMware

Once the hardware is free:

1. Open VMware Workstation with the VM powered off.
2. Go to **Virtual Machine Settings > Processors**.
3. Enable **Virtualize Intel VT-x/EPT or AMD-V/RVI**.

## Security Note

Disabling VBS and Memory Integrity removes a native Windows 11 process isolation layer. Apply this fix only in controlled local environments used for development, auditing, or security research.
