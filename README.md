# Ultimate Windows Performance Enhancement Script
if (!([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)) {
    Write-Host "You need to run this script as an administrator!" -ForegroundColor Red
    exit
}

Write-Host "Starting the ultimate Windows performance optimization script..." -ForegroundColor Yellow

# PART 1: Core Parking and CPU Throttling Fix
Write-Host "Optimizing CPU core parking and throttling settings..." -ForegroundColor Yellow

# Disable CPU core parking
$guid = "0cc5b647-c1df-4637-891a-dec35c318583"
powercfg /setacvalueindex scheme_current sub_processor $guid 0
powercfg /setdcvalueindex scheme_current sub_processor $guid 0
powercfg -setactive scheme_current

# Maximize CPU performance and disable CPU throttling
$procThrottleMin = "893dee8e-2bef-41e0-89c6-b55d0929964c"
$procThrottleMax = "bc5038f7-23e0-4960-96da-33abaf5935ec"
powercfg /setacvalueindex scheme_current sub_processor $procThrottleMin 100
powercfg /setacvalueindex scheme_current sub_processor $procThrottleMax 100
powercfg -setactive scheme_current

# PART 2: Memory and Virtual Memory Optimizations
Write-Host "Optimizing memory management and virtual memory..." -ForegroundColor Yellow
$memoryPath = "HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management"

# Disable Paging Executive to keep core processes in RAM
New-ItemProperty -Path $memoryPath -Name "DisablePagingExecutive" -Value 1 -Force

# Use RAM aggressively for file caching and minimize paging
New-ItemProperty -Path $memoryPath -Name "LargeSystemCache" -Value 1 -Force
New-ItemProperty -Path $memoryPath -Name "IoPageLockLimit" -Value 524288 -Force # 512MB I/O cache

# Optimize virtual memory (pagefile)
New-ItemProperty -Path $memoryPath -Name "PagingFiles" -Value "C:\pagefile.sys 2048 4096" -Force

# PART 3: Disk I/O and NTFS Optimizations
Write-Host "Optimizing disk I/O performance and NTFS..." -ForegroundColor Yellow

# Disable NTFS last access time updates
New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "NtfsDisableLastAccessUpdate" -Value 1 -Force

# Disable NTFS 8.3 name creation for faster file access
New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "NtfsDisable8dot3NameCreation" -Value 1 -Force

# PART 4: Power Management Optimization
Write-Host "Disabling power-saving features for maximum performance..." -ForegroundColor Yellow

# Enable Ultimate Performance power scheme
powercfg -duplicatescheme e9a42b02-d5df-448d-aa00-03f14749eb61

# Disable hard disk sleep
powercfg /change disk-timeout-ac 0

# Disable hibernation
powercfg -h off

# Disable USB selective suspend
$usbSuspendGuid = "2a737441-1930-4402-8d77-b2bebba308a3"
powercfg /setacvalueindex scheme_current sub_usb $usbSuspendGuid 0
powercfg /setdcvalueindex scheme_current sub_usb $usbSuspendGuid 0
powercfg -setactive scheme_current

# PART 5: Fix for DoSvc Permissions
Write-Host "Stopping and disabling DoSvc (Delivery Optimization) service..." -ForegroundColor Yellow

# Stop and disable DoSvc (Delivery Optimization)
sc stop DoSvc
sc config DoSvc start= disabled
