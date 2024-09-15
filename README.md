# Performance-No-Limits
Windows No Limits Core Performance
# Ultimate Windows Performance Enhancement Script
# WARNING: This script is highly aggressive and will disable many Windows features to maximize performance.
# It should be used in environments where performance is prioritized over features and appearance.

# Ensure the script is run as Administrator
if (!([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)) {
    Write-Host "You need to run this script as an administrator!" -ForegroundColor Red
    exit
}

Write-Host "Starting the ultimate Windows performance optimization script..." -ForegroundColor Yellow

# PART 1: Kernel-Level CPU Scheduling, Thread Management, and Interrupt Handling

Write-Host "Optimizing CPU scheduling, thread management, and interrupt handling..." -ForegroundColor Yellow
# Disable CPU core parking (ensures all cores are active at all times)
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Power\PowerSettings\0cc5b647-c1df-4637-891a-dec35c318583" -Name "Attributes" -Value 0  # Expose core parking settings
powercfg -setacvalueindex SCHEME_MIN SUB_PROCESSOR 0cc5b647-c1df-4637-891a-dec35c318583 0  # Disable core parking
powercfg -setactive SCHEME_MIN  # Apply the ultimate performance scheme

# Maximize CPU performance and disable CPU throttling completely
powercfg /setacvalueindex SCHEME_MIN SUB_PROCESSOR PROCTHROTTLEMIN 100
powercfg /setacvalueindex SCHEME_MIN SUB_PROCESSOR PROCTHROTTLEMAX 100

# Prioritize foreground processes and reduce latency in context switches
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\PriorityControl" -Name "Win32PrioritySeparation" -Value 26

# Increase timer resolution for high-priority threads
New-ItemProperty -Path "HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\Kernel" -Name "LowPriorityProcessorIdle" -Value 1 -Force
New-ItemProperty -Path "HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\Kernel" -Name "TimeCriticalThreadBoost" -Value 1 -Force

# Enable IRQ affinity and distribute interrupts across CPU cores
Set-ItemProperty -Path "HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\Kernel" -Name "IRQLAffinity" -Value 1 -Force

# PART 2: Deep Memory and Virtual Memory Optimizations

Write-Host "Optimizing memory management and virtual memory for extreme performance..." -ForegroundColor Yellow
# Disable Paging Executive (forces Windows to keep core processes in RAM instead of paging them to disk)
New-ItemProperty -Path "HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management" -Name "DisablePagingExecutive" -Value 1 -Force

# Force Windows to aggressively use RAM for file caching and reduce paging file usage
New-ItemProperty -Path "HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management" -Name "LargeSystemCache" -Value 1 -Force
New-ItemProperty -Path "HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management" -Name "IoPageLockLimit" -Value 524288  # 512MB I/O cache

# Optimize virtual memory settings for minimal paging
New-ItemProperty -Path "HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management" -Name "PagingFiles" -Value "C:\pagefile.sys 2048 4096" -Force  # 2GB min, 4GB max

# PART 3: Optimize Disk I/O Performance and NTFS Settings

Write-Host "Optimizing disk I/O performance and NTFS file system..." -ForegroundColor Yellow
# Disable NTFS last access time updates to reduce unnecessary disk writes
New-ItemProperty -Path "HKLM\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "NtfsDisableLastAccessUpdate" -Value 1 -Force

# Disable NTFS 8.3 name creation (for legacy compatibility), speeds up file access
New-ItemProperty -Path "HKLM\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "NtfsDisable8dot3NameCreation" -Value 1 -Force

# Boost NTFS performance by increasing file system cache size
New-ItemProperty -Path "HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management" -Name "IoPageLockLimit" -Value 1048576  # 1GB I/O cache

# PART 4: Power Management Optimization

Write-Host "Disabling all power-saving features and maximizing system performance..." -ForegroundColor Yellow
# Enable Ultimate Performance power scheme
powercfg -duplicatescheme e9a42b02-d5df-448d-aa00-03f14749eb61

# Disable CPU throttling and frequency scaling
powercfg /setacvalueindex SCHEME_MIN SUB_PROCESSOR PERFMODE 0  # Set CPU boost mode to aggressive

# Disable hard disk sleep (ensures the drive stays active at all times)
powercfg /change disk-timeout-ac 0

# Disable hibernation (frees up disk space and ensures faster boot times)
powercfg -h off

# Disable USB selective suspend (ensures USB devices stay powered on)
powercfg /setacvalueindex SCHEME_MIN SUB_USB SUB_USBSS 0

# PART 5: Network Stack and Latency Optimizations

Write-Host "Optimizing the network stack for high throughput and low latency..." -ForegroundColor Yellow
# Disable Nagle's algorithm (reduces TCP latency for faster networking)
$interfaces = Get-ChildItem "HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\Interfaces"
foreach ($interface in $interfaces) {
    New-ItemProperty -Path $interface.PSPath -Name "TcpNoDelay" -Value 1 -Force
}

# Increase the TCP window size to improve network throughput
New-ItemProperty -Path "HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters" -Name "TcpWindowSize" -Value 524288 -Force  # 512 KB window size
New-ItemProperty -Path "HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters" -Name "GlobalMaxTcpWindowSize" -Value 524288 -Force

# Disable auto-tuning of TCP for more consistent network performance
New-ItemProperty -Path "HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters" -Name "EnableTCPA" -Value 0 -Force  # Disable TCP auto-tuning

# PART 6: Strip Visual Effects and Animations for Maximum Responsiveness

Write-Host "Disabling all visual effects and animations for the snappiest UI..." -ForegroundColor Yellow
# Set the system for best performance by disabling visual effects
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\VisualEffects" -Name "VisualFXSetting" -Value 2 -Force

# Disable window and taskbar animations
Set-ItemProperty -Path "HKCU:\Control Panel\Desktop" -Name "DragFullWindows" -Value 0 -Force
Set-ItemProperty -Path "HKCU:\Control Panel\Desktop" -Name "MenuShowDelay" -Value 0 -Force  # Instant menu display
Set-ItemProperty -Path "HKCU:\Control Panel\Desktop" -Name "WindowAnimation" -Value 0 -Force

# Disable Aero Peek and transparency effects
New-ItemProperty -Path "HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced" -Name "DisablePreviewDesktop" -Value 1 -Force  # Disable Aero Peek
New-ItemProperty -Path "HKCU\Software\Microsoft\Windows\CurrentVersion\Themes\Personalize" -Name "EnableTransparency" -Value 0 -Force  # Disable transparency

# Disable taskbar animations for instant task switching
New-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced" -Name "TaskbarAnimations" -Value 0 -Force

# PART 7: Aggressive Service Disabling and Startup Optimization

Write-Host "Disabling unnecessary services and background tasks..." -ForegroundColor Yellow
$services = @(
    "SysMain",  # Superfetch
    "WSearch",  # Windows Search
    "DiagTrack",  # Diagnostic Tracking Service
    "DoSvc",  # Delivery Optimization
    "Fax",  # Fax service
    "XblAuthManager",  # Xbox Live Auth Manager
    "XblGameSave",  # Xbox Game Save
    "WMPNetworkSvc",  # Windows Media Player Network Sharing
    "Spooler",  # Print Spooler (if no printer is used)
    "MapsBroker",  # Offline Maps service
    "RetailDemo",  # Retail Demo service
    "BluetoothUserService",  # Bluetooth service (if not used)
    "IKEEXT",  # IPsec Keying Modules
    "OneSyncSvc",  # Sync Host
    "BcastDVRUserService",  # Game Broadcasting
    "MessagingService"  # Messaging Service (if not used)
)

foreach ($service in $services) {
    Set-Service -Name $service -StartupType Disabled
    Stop-Service -Name $service -Force
    Write-Host "Disabled $service service" -ForegroundColor Green
}

# PART 8: Disable Background Apps and Telemetry

Write-Host "Disabling background apps and telemetry services..." -ForegroundColor Yellow
# Disable Windows telemetry (tracking and reporting)
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\DataCollection" -Name "AllowTelemetry" -Value 0 -Force

# Disable Windows Error Reporting (fewer popups, reduced CPU usage)
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\Windows Error Reporting" -Name "Disabled" -Value 1 -Force

# Disable feedback notifications and diagnostics
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Siuf\Rules" -Name "NumberOfSIUFInPeriod" -Value 0 -Force  # Disable feedback requests

# Disable background apps from running
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\AppPrivacy" -Name "LetAppsRunInBackground" -Value 2 -Force  # Prevent all background apps from running

# PART 9: Final System Optimizations for Speed

Write-Host "Applying final system optimizations for maximum speed..." -ForegroundColor Yellow
# Remove startup delay
New-Item -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\Serialize" -Force | Out-Null
New-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\Serialize" -Name "StartupDelayInMSec" -Value 0 -Force

Write-Host "Ultimate performance optimizations complete! Reboot your system for all changes to take effect." -ForegroundColor Green
