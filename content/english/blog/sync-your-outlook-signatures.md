+++
author = "Vincent"
categories = ["Microsoft 365"]
date = 2022-10-30T20:00:00Z
description = "Outlook signatures"
images = ["/images/syncoutllookaltered.png"]
tags = []
title = "Outlook signature sync with Onedrive"

+++
#### Intro
Microsoft is still working (2 years) on their own Outlook signature sync. Our helpdesk still gets alot of calls with the question how to get their Outlook signature back when they receive a new device. 

To reduce the workload i decided to find a solution for this signature problem.

My colleague gave me an interesting link from a blog from [Jose Gabriel Ortega C](https://j0rt3g4.medium.com/save-your-outlook-signatures-into-onedrive-and-never-lose-them-again-1337fc1924b6). 
I really liked his idea of saving the signature in Onedrive. 
While i examined his code i decided i could make it a bit more robust when giving out new devices to our employees.

My proactive remediation script consists of a detection and a remediation script.

#### The Detection script.

First it gets the paths of all the locations the Outlook signature could be and does some checks.

The signature is saved in Outlook here:
```Powershell
#Windows directory
%APPDATA%\Roaming\Microsoft\Signatures 
```
Or
The signature is backed up here in Onedrive and we need to set it back to Outlook.
```Powershell
#Onedrive directory
C:\Users\yourname\your-onedrive-folder\Signatures
```

We have 3 options in my detection script.

##### 1. 
There is no Outlook signature present in Outlook %APPDATA% or in the signatures folder in Onedrive. 
This is probably a new employee that needs to setup its Outlook signature for the first time. 
We do nothing.

##### 2. 
If there is a Outlook signature in Onedrive or Outlook %APPDATA% folder. 
We trigger the remediation script.

##### 3. 
If there is a onedrive backup folder and a %APPDATA% folder. Your signature is synced between Onedrive and Outlook. Ofcourse we do not trigger the remediation script now.

#### The Remediation script.

The remediation script makes a Onedrive signatures map or a Outlook signature folder. 
Then it will create a junction between Outlook and your Outlook signature map. 
The junction makes sure if you update your Outlook folder, the changes will update in your Outlook signature folder too!
```Powershell
#Make folder junction
cmd /c mklink /J $LocalOutlookSignature $SignatureOnedrive
```

Done! Your Outlook signature is backed up and will sync with your next device!

Use [this link](https://github.com/vincentverstraeten/Powershell-Scripts/tree/main/Proactive%20Remediations/Sync%20Outlook%20Signatures) to get the proactive remediation script from my github page.

Or copy paste the code below:

#### Detection.ps1
```Powershell
#detection.ps1
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
    if (!(test-path -Path "$SignatureOnedrive") -and !(test-path -Path "$SignatureOutlook") ) { 
        # Outlook Signature and Onedrive does not exist. Wait till Outlook signature is set in %APPDATA%.
        exit 0
    }
    if (!(test-path -Path "$SignatureOnedrive") -or !(test-path -Path "$SignatureOutlook") ) { 
        # Outlook Signature or Microsoft signature does not exist in Onedrive, remediation needed.
        exit 1
    }
    if (!(test-path -Path "$SignatureOutlook") -or ((Get-Item -Path $SignatureOutlook  -Force).LinkType -ne "Junction") ) { 
    # Outlook Junction Signature does not exist in Appdata/Microsoft/Outlook, remediation needed
    exit 1
   }
else {
        # Outlook Signature is synced, do not remediate.       
        exit 0
    }
}
catch{
    $errMsg = $_.Exception.Message
    Write-Error $errMsg
    exit 1
}
```

#### Remediation.ps1
```Powershell
#=============================================================================================================================
# Script Name: Backup Outlook Signature to onedrive
# Description: Symlink Outlook Signature to onedrive   New-Item -ItemType SymbolicLink -Path $LocalOutlookSignature  -Target $SignatureOnedrive -Force
# Notes: Vincent Verstraeten 08/2022     New-Item -Path $SignatureOnedrive -ItemType "directory" -Force

#=============================================================================================================================

# Main script
# get local user folders
$env:APPDATA

$LocalOutlookSignature = Join-Path $env:APPDATA -ChildPath "Microsoft/Signatures"
$LocalOutlookBackupSignature = Join-Path $env:APPDATA -ChildPath "Microsoft/Signatures_backup"
$OneDrivePath = [Environment]::GetEnvironmentVariable("ONEDRIVE",[EnvironmentVariableTarget]::User)   
$SignatureOnedrive = join-path -Path $OneDrivePath -ChildPath "Signatures"



if ((Test-Path -Path $LocalOutlookSignature -ErrorAction SilentlyContinue) -or ((Get-Item -Path $LocalOutlookSignature -Force -ErrorAction SilentlyContinue).LinkType -eq "Junction"))  {
    Copy-Item -Path $LocalOutlookSignature -Destination $LocalOutlookBackupSignature -recurse -Force
    Remove-Item -LiteralPath $LocalOutlookSignature -Force -Recurse
        if(!(Test-Path -Path $SignatureOnedrive)){
                mkdir $SignatureOnedrive
                # Make onedrive signatures map
         }
    cmd /c mklink /J $LocalOutlookSignature $SignatureOnedrive #Powershell command needs admin, only in preview windows(better use cmd here)
    Copy-Item -Path "$LocalOutlookBackupSignature\*" -Destination $LocalOutlookSignature -recurse -Force
    Remove-Item -LiteralPath $LocalOutlookBackupSignature -Force -Recurse
    # Copy over local signature data to onedrive
}

#Check if onedrive signatures map is present but local microsoft signature outlook is not there. Then copy Onedrive Signature to outlook signature.

if (!(Test-Path $LocalOutlookSignature)) {
    Copy-Item -Path $SignatureOnedrive -Destination $LocalOutlookBackupSignature -recurse -Force
    cmd /c mklink /J $LocalOutlookSignature $SignatureOnedrive
    Copy-Item -Path $LocalOutlookBackupSignature\* -Destination $SignatureOnedrive  -recurse -Force
    Remove-Item -LiteralPath $LocalOutlookBackupSignature -Force -Recurse
    # Copy onedrive to outlook signature
}

if (!((Get-Item -Path $LocalOutlookSignature  -Force).LinkType -eq "Junction")) {
    Remove-Item -LiteralPath $LocalOutlookSignature -Force -Recurse
    cmd /c mklink /J $LocalOutlookSignature $SignatureOnedrive
}
```







