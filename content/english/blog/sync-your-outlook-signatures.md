+++
author = "Vincent"
categories = ["Microsoft 365"]
date = 2022-10-30T20:00:00Z
description = "Outlook signatures"
images = ["/images/syncoutllookaltered.png"]
tags = []
title = "Sync your Outlook signature with Onedrive"

+++
Sync your outlook signatures

```PowerShell
#=============================================================================================================================
#
# Script Name: Outlook Signature
# Description: Set symlink between appdata\Microsoft\Signatures and onedrive\Signatures
# Written by: Vincent Verstraeten                      
#=============================================================================================================================

# Define Variables
$OneDrivePath = [Environment]::GetEnvironmentVariable("ONEDRIVE",[EnvironmentVariableTarget]::User)   
$SignatureOnedrive = join-path -Path $OneDrivePath -ChildPath "Signatures" 
$SignatureOutlook = Join-Path $env:APPDATA -ChildPath "Microsoft/Signatures"

try
{
    if (!(test-path -Path "$SignatureOnedrive") -or !(test-path -Path "$SignatureOutlook") ) { 
        Write-Host "Outlook Signature or Microsoft signature does not exist in Onedrive, remediation needed"
        exit 1
    }
    if (!(test-path -Path "$SignatureOutlook") -or ((Get-Item -Path $SignatureOutlook  -Force).LinkType -ne "Junction") ) { 
    Write-Host "Outlook Junction Signature does not exist in Appdata/Microsoft/Outlook, remediation needed"
    exit 1
   }
else {
        #do not remediate
        Write-Host "Outlook Signature is synced, do not remediate."        
        exit 0
    }
}
catch{
    $errMsg = $_.Exception.Message
    Write-Error $errMsg
    exit 1
}
```