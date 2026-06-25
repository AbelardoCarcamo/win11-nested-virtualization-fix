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
````md
## Additional Troubleshooting

In some systems, disabling Hyper-V and VBS through Windows may not be sufficient. Certain firmware (BIOS/UEFI) security features can cause Windows to continue loading virtualization-based security components even after the standard fix has been applied.

### Verify Hypervisor Status

After applying the fix and rebooting, run:

```cmd
systeminfo
````

If the output still reports:

```text
A hypervisor has been detected.
```

additional firmware or security settings may still be enabling virtualization-based security.

### Additional BIOS/UEFI Checks

Review the following settings if available on your platform:

* Virtualization Technology (VT-x / AMD-V)
* SVM Mode (AMD systems)
* IOMMU / AMD-Vi
* Kernel DMA Protection
* Secure Boot
* Trusted Execution Technology (TXT) on Intel platforms
* Other firmware security features related to virtualization-based security

Availability and naming vary by vendor, platform generation, and firmware version.

### Additional Boot Configuration Verification

Confirm the current bootloader configuration:

```cmd
bcdedit /enum {current}
```

Verify that:

```text
hypervisorlaunchtype    Off
```

is present.

If not, reapply:

```cmd
bcdedit /set {current} hypervisorlaunchtype off
```

and reboot.

### Credential Guard / Device Guard

Some environments may continue enforcing virtualization-based security through Credential Guard or Device Guard policies.

Check current status:

```powershell
Get-CimInstance Win32_DeviceGuard
```

or review:

```cmd
msinfo32
```

for VBS-related entries.

### Validation

Nested virtualization should only be enabled after:

* `systeminfo` no longer reports a detected hypervisor.
* VBS status reports disabled.
* VMware Workstation or VirtualBox can access VT-x/AMD-V directly.

### Tested Platforms

The procedure has been successfully tested on:

* AMD Ryzen 7 6000 Series platforms
* Intel Core 4th Generation platforms
* Intel Core 11th Generation platforms

Results may vary depending on BIOS version, OEM firmware customizations, and Windows security policies.

```
```

## Security Note

Disabling VBS and Memory Integrity removes a native Windows 11 process isolation layer. Apply this fix only in controlled local environments used for development, auditing, or security research.
