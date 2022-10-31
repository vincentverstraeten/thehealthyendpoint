+++
author = "Vincent"
categories = ["Microsoft 365"]
date = 2022-10-30T20:00:00Z
description = "Outlook signatures"
images = ["/images/syncoutllookaltered.png"]
tags = []
title = "Sync your Outlook signature with Onedrive"

+++
Our Helpdesk was telling me that they had alot of calls when employees get a new device and they need to setup their outlook signature again.

My colleague gave me an interesting link from a blog from Jose Gabriel Ortega C. I really liked his idea of saving the signature in Onedrive. While i examined his code i decided i could make it a bit more modern with proactive remediation and write it in Powershell code. 

My proactive remediation script consists of a detection and a remediation script. 

What does my detection script do?

First it gets the paths of all the locations the Outlook signature could be.

The outlook signatured is saved in this folder by Outlook:
'''PowerShell
%APPDATA%\Romaing\Microsoft\Signatures
'''




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