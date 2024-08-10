# Hosts File Management Script Documentation

## Overview

This PowerShell script allows you to manage entries in your Windows `hosts` file. You can use this script to add, remove, or list entries in the `hosts` file.

## Features

- **Add Entry**: Add a new IP address and hostname pair to the `hosts` file.
- **Remove Entry**: Remove an existing entry based on the hostname.
- **List Entries**: List all active (non-commented) entries in the `hosts` file.

## Prerequisites

- **Windows PowerShell**: The script is designed to run in PowerShell on a Windows machine.
- **Administrative Privileges**: The script needs to be run with administrator privileges to modify the `hosts` file.

## Usage

### Parameters

- **`-Action`**: The action to perform. Options are `add`, `remove`, and `list`.
- **`-IPAddress`**: The IP address to add (required for the `add` action).
- **`-Hostname`**: The hostname to add or remove.

### Example Commands

#### Add a New Entry

To add a new entry to the `hosts` file:

\```powershell
.\Manage-Hosts.ps1 -Action add -IPAddress 127.0.0.1 -Hostname editor.local
\```

#### Remove an Entry

To remove an entry from the `hosts` file:

\```powershell
.\Manage-Hosts.ps1 -Action remove -Hostname editor.local
\```

#### List All Entries

To list all non-commented entries in the `hosts` file:

\```powershell
.\Manage-Hosts.ps1 -Action list
\```

## Script Details

### Script: `Manage-Hosts.ps1`

\```powershell
# Main script parameters
param (
    [string]$Action,
    [string]$IPAddress,
    [string]$Hostname
)

# Define the path to the hosts file
$hostsFilePath = "$env:SystemRoot\System32\drivers\etc\hosts"

# Function to add a new entry to the hosts file
function Add-HostsEntry {
    param (
        [string]$IPAddress,
        [string]$Hostname
    )

    # Read the content of the hosts file
    $hostsContent = Get-Content $hostsFilePath

    # Check if the entry already exists
    if ($hostsContent -notmatch "^\s*$IPAddress\s+$Hostname\s*$") {
        # Add the new entry to the hosts file on a new line
        "`r`n$IPAddress`t$Hostname" | Add-Content -Path $hostsFilePath
        Write-Host "Added '$Hostname' with IP '$IPAddress' to the hosts file."
    } else {
        Write-Host "The hostname '$Hostname' already exists in the hosts file."
    }
}

# Function to remove an entry from the hosts file
function Remove-HostsEntry {
    param (
        [string]$Hostname
    )

    # Read the content of the hosts file
    $hostsContent = Get-Content $hostsFilePath

    # Filter out the line containing the specified hostname
    $updatedContent = $hostsContent | Where-Object { $_ -notmatch "^\s*\d{1,3}(\.\d{1,3}){3}\s+$Hostname\s*$" }

    # Save the updated content back to the hosts file
    Set-Content -Path $hostsFilePath -Value $updatedContent

    Write-Host "Removed '$Hostname' from the hosts file."
}

# Function to list all entries in the hosts file
function List-HostsEntries {
    Write-Host "Current entries in the hosts file:"
    Get-Content $hostsFilePath | Where-Object { $_ -notmatch '^#' -and $_ -notmatch '^\s*$' }
}

# Determine the action to take
switch ($Action) {
    "add" {
        if (-not $IPAddress -or -not $Hostname) {
            Write-Host "Usage: .\Manage-Hosts.ps1 -Action add -IPAddress <IP> -Hostname <hostname>"
            exit
        }
        Add-HostsEntry -IPAddress $IPAddress -Hostname $Hostname
    }
    "remove" {
        if (-not $Hostname) {
            Write-Host "Usage: .\Manage-Hosts.ps1 -Action remove -Hostname <hostname>"
            exit
        }
        Remove-HostsEntry -Hostname $Hostname
    }
    "list" {
        List-HostsEntries
    }
    default {
        Write-Host "Usage: .\Manage-Hosts.ps1 -Action {add|remove|list} [-IPAddress <IP>] [-Hostname <hostname>]"
    }
}
\```

## Common Issues

### Execution Policy Error

If you receive an error about the execution policy preventing the script from running:

1. Open PowerShell as Administrator.
2. Run the following command to allow script execution:
   \```powershell
   Set-ExecutionPolicy RemoteSigned
   \```
3. Retry running the script.

### Removing an Entry Removes Entire File

If the script mistakenly removes all content from the `hosts` file, the issue is likely due to the way the script handled the line removal. The latest version of the script addresses this issue by using `Where-Object` to filter out only the matching line.

### Restoring the Default `hosts` File

If your `hosts` file is cleared, you can recreate it by pasting the following default content:

\```plaintext
# Copyright (c) 1993-2009 Microsoft Corp.
#
# This is a sample HOSTS file used by Microsoft TCP/IP for Windows.
#
# This file contains the mappings of IP addresses to host names. Each
# entry should be kept on an individual line. The IP address should
# be placed in the first column followed by the corresponding host name.
# The IP address and the host name should be separated by at least one
# space.
#
# Additionally, comments (such as these) may be inserted on individual
# lines or following the machine name denoted by a '#' symbol.
#
# For example:
#
#      102.54.94.97     rhino.acme.com          # source server
#       38.25.63.10     x.acme.com              # x client host

# localhost name resolution is handled within DNS itself.
#       127.0.0.1       localhost
#       ::1             localhost
\```

## Conclusion

This PowerShell script provides a simple and effective way to manage your Windows `hosts` file, allowing you to easily add, remove, and list entries. Always ensure you have a backup of your `hosts` file before making significant changes, especially if you're using the script to remove entries.

