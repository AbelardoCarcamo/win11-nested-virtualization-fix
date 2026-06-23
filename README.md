@echo off
net session >nul 2>&1
if %errorLevel% neq 0 (
    echo [ERROR] Run this script as Administrator.
    echo Right-click the file and select "Run as Administrator".
    pause
    exit /b
)

title Windows 11 Nested Virtualization Fix
color 0A

echo ==============================================================
echo         WINDOWS 11 NESTED VIRTUALIZATION FIX
echo ==============================================================
echo.
echo [1/4] Opening Windows Security - Device Security...
echo MANUAL STEP: Disable "Memory Integrity" if enabled, then return here.
echo.
start windowsdefender://device-security
pause

echo.
echo [2/4] Disabling optional Windows features (Hyper-V, WSL, etc.)...
dism /online /disable-feature /featurename:Microsoft-Hyper-V-All /norestart
dism /online /disable-feature /featurename:VirtualMachinePlatform /norestart
dism /online /disable-feature /featurename:HypervisorPlatform /norestart
dism /online /disable-feature /featurename:Microsoft-Windows-Subsystem-Linux /norestart

echo.
echo [3/4] Setting hypervisor launch type to off in bootloader...
bcdedit /set hypervisorlaunchtype off

echo.
echo [4/4] Disabling VBS via registry...
reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\DeviceGuard" /v "EnableVirtualizationBasedSecurity" /t REG_DWORD /d 0 /f

echo.
echo ==============================================================
echo   DONE. Restart the system to apply all changes.
echo ==============================================================
echo.
set /p REBOOT="Restart now? (Y/N): "
if /I "%REBOOT%"=="Y" shutdown /r /t 5
exit
