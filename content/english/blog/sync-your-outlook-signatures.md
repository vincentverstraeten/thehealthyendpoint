+++
author = "Vincent"
categories = ["Microsoft 365"]
date = 2022-10-30T20:00:00Z
description = "Outlook signatures"
images = ["/images/syncoutllookaltered.png"]
tags = []
title = "Sync your Outlook signature with Onedrive"

+++
### Intro
Our Helpdesk was telling me that they had alot of calls when employees get a new device and they need to setup their outlook signature again.

My colleague gave me an interesting link from a blog from [Jose Gabriel Ortega C](https://j0rt3g4.medium.com/save-your-outlook-signatures-into-onedrive-and-never-lose-them-again-1337fc1924b6). 
I really liked his idea of saving the signature in Onedrive. 
While i examined his code i decided i could make it a bit more robust when giving out new devices to our employees.

My proactive remediation script consists of a detection and a remediation script.

### Detection script.

First it gets the paths of all the locations the Outlook signature could be and does some checks.

The signature is saved in Outlook here:
```Powershell
#Windows directory
%APPDATA%\Roaming\Microsoft\Signatures 
```
Or
The signature is backed up here in Onedrive and we need to set it back to Outlook.
```Powershell
#Windows directory
C:\Users\yourname\your-onedrive-folder\Signatures
```

We have 3 options in my detection script.

1. There is no Outlook signature present in Outlook %APPDATA% or in the signatures folder in Onedrive. 
This is probably a new employee that needs to setup its Outlook signature for the first time. 
We do nothing.

2. If there is a Outlook signature in Onedrive or Outlook %APPDATA% folder. 
We trigger the remediation script.

3. If there is a onedrive backup folder and a %APPDATA% folder. Your signature is synced between Onedrive and Outlook. Ofcourse we do not trigger the remediation script now.

### Remediation script.

