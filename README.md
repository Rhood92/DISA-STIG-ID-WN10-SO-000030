# STIG ID: WN10-SO-000030 - Audit policy using subcategories must be enabled.

## Synopsis
This PowerShell script ensures that the Audit policy using subcategories is enabled.

## Notes
- **Author**: Richard Hood
- **LinkedIn**: [Richard Hood Jr.](https://www.linkedin.com/in/richard-hood-jr/)
- **GitHub**: [Rhood92](https://github.com/Rhood92)
- **Date Created**: 2025-04-2
- **Last Modified**: 2025-04-2
- **Version**: 1.0
- **CVEs**: N/A
- **Plugin IDs**: N/A
- **STIG-ID**: WN10-SO-000030
-   
## Tested On
- **Date(s) Tested**: 
- **Tested By**: 
- **Systems Tested**: 
- **PowerShell Ver.**: 

## Usage
Put any usage instructions here.

Example syntax:
```powershell
 # Requires administrative privileges
if (-not ([Security.Principal.WindowsPrincipal] [Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)) {
    Write-Host "This script requires administrative privileges. Please run PowerShell as an Administrator."
    exit 1
}

# Define registry path and value
$registryPath = "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa"
$valueName = "SCENoApplyLegacyAuditPolicy"
$requiredValue = 1

# Check if the registry key exists, create it if it doesn't
if (-not (Test-Path $registryPath)) {
    Write-Host "Registry path $registryPath does not exist. Creating it..."
    New-Item -Path $registryPath -Force | Out-Null
}

# Check the current value of SCENoApplyLegacyAuditPolicy
Write-Host "Checking current value of $valueName..."
$currentValue = $null
try {
    $currentValue = Get-ItemProperty -Path $registryPath -Name $valueName -ErrorAction Stop | Select-Object -ExpandProperty $valueName
    Write-Host "Current value of $valueName is: $currentValue"
} catch {
    Write-Host "The registry value $valueName does not exist. It will be created."
}

# Set the registry value to 1 if it doesn't already match
if ($currentValue -ne $requiredValue) {
    Write-Host "Setting $valueName to $requiredValue..."
    try {
        Set-ItemProperty -Path $registryPath -Name $valueName -Value $requiredValue -Type DWord -Force -ErrorAction Stop
        Write-Host "Successfully set $valueName to $requiredValue."
    } catch {
        Write-Host "Failed to set the registry value. Error: $_"
        exit 1
    }
} else {
    Write-Host "The value of $valueName is already set to $requiredValue. No changes needed."
}

# Verify the change
Write-Host "Verifying the change..."
try {
    $verifiedValue = Get-ItemProperty -Path $registryPath -Name $valueName -ErrorAction Stop | Select-Object -ExpandProperty $valueName
    if ($verifiedValue -eq $requiredValue) {
        Write-Host "Verification successful: $valueName is set to $verifiedValue."
    } else {
        Write-Host "Verification failed: $valueName is set to $verifiedValue, expected $requiredValue."
        exit 1
    }
} catch {
    Write-Host "Failed to verify the registry value. Error: $_"
    exit 1
}

# Inform the user to reboot for the change to take effect
Write-Host "Audit policy subcategory settings have been enabled. A reboot is required for the change to take effect." 
