# STIG ID: WN10-AC-000035 - Passwords must, at a minimum, be 14 characters.

## Synopsis
This PowerShell script ensures that the The Passwords must, at a minimum, be 14 characters.

## Notes
- **Author**: Richard Hood
- **LinkedIn**: [Richard Hood Jr.](https://www.linkedin.com/in/richard-hood-jr/)
- **GitHub**: [Rhood92](https://github.com/Rhood92)
- **Date Created**: 2025-04-2
- **Last Modified**: 2025-04-2
- **Version**: 1.0
- **CVEs**: N/A
- **Plugin IDs**: N/A
- **STIG-ID**: WN10-AC-000035
  
## Tested On
- **Date(s) Tested**: 
- **Tested By**: 
- **Systems Tested**: 
- **PowerShell Ver.**: 

## Usage
Put any usage instructions here.

Example syntax:
```powershell
# STIG ID: WN10-AC-000035
# Title: Passwords must, at a minimum, be 14 characters.

# PowerShell script to set the minimum password length to 14 characters (STIG WN10-AC-000035)

# PowerShell script to set the minimum password length to 14 characters (STIG WN10-AC-000035)

# Requires administrative privileges
if (-not ([Security.Principal.WindowsPrincipal] [Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)) {
    Write-Host "This script requires administrative privileges. Please run PowerShell as an Administrator."
    exit 1
}

# Define paths for temporary files
$exportFile = "$env:TEMP\secpol.cfg"
$updatedFile = "$env:TEMP\secpol_updated.cfg"
$logFile = "$env:TEMP\secpol_log.txt"

# Export the current security policy to a temporary file
Write-Host "Exporting current security policy to $exportFile..."
$exportResult = secedit /export /cfg $exportFile /log $logFile 2>&1
if ($LASTEXITCODE -ne 0) {
    Write-Host "Failed to export security policy. Exit code: $LASTEXITCODE"
    Write-Host "Export error details:"
    Get-Content $logFile -ErrorAction SilentlyContinue
    exit 1
}

# Check if the export file exists
if (-not (Test-Path $exportFile)) {
    Write-Host "Export file not found at $exportFile. Ensure you have the necessary permissions."
    exit 1
}

# Read the exported security policy file
Write-Host "Modifying security policy..."
$secpolContent = Get-Content $exportFile

# Debug: Display the current MinimumPasswordLength value
$currentLength = $secpolContent | Select-String "MinimumPasswordLength"
Write-Host "Current MinimumPasswordLength: $currentLength"

# Update the MinimumPasswordLength value to 14
$newContent = $secpolContent -replace "MinimumPasswordLength = \d+", "MinimumPasswordLength = 14"

# Save the updated configuration to a new file
$newContent | Set-Content $updatedFile

# Debug: Verify the updated file content
$updatedLength = (Get-Content $updatedFile | Select-String "MinimumPasswordLength").ToString()
Write-Host "Updated MinimumPasswordLength in file: $updatedLength"

# Import the updated security policy
Write-Host "Applying updated security policy..."
$importResult = secedit /configure /db $env:windir\security\database\secedit.sdb /cfg $updatedFile /log $logFile /quiet 2>&1
if ($LASTEXITCODE -ne 0) {
    Write-Host "Failed to apply the updated security policy. Exit code: $LASTEXITCODE"
    Write-Host "Import error details:"
    Get-Content $logFile -ErrorAction SilentlyContinue

    # Fallback: Try using 'net accounts' to set the minimum password length
    Write-Host "Attempting fallback method using 'net accounts'..."
    try {
        net accounts /minpwlen:14
        if ($LASTEXITCODE -eq 0) {
            Write-Host "Successfully set the minimum password length to 14 using 'net accounts'."
        } else {
            Write-Host "Fallback method failed. Exit code: $LASTEXITCODE"
            exit 1
        }
    } catch {
        Write-Host "Error using 'net accounts': $_"
        exit 1
    }
} else {
    Write-Host "Successfully applied the updated security policy using 'secedit'."
}

# Clean up temporary files
Write-Host "Cleaning up temporary files..."
Remove-Item $exportFile -ErrorAction SilentlyContinue
Remove-Item $updatedFile -ErrorAction SilentlyContinue
Remove-Item $logFile -ErrorAction SilentlyContinue

# Verify the change
Write-Host "Verifying the change..."
$check = (net accounts | Select-String "Minimum password length").ToString()
Write-Host $check

# Inform the user to log off or reboot for the change to take effect
Write-Host "A logoff or reboot may be required for the change to take effect." 
