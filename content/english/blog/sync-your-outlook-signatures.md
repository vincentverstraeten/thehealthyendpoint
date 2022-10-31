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

My colleague gave me an interesting link from a blog from [Jose Gabriel Ortega C](https://j0rt3g4.medium.com/save-your-outlook-signatures-into-onedrive-and-never-lose-them-again-1337fc1924b6). I really liked his idea of saving the signature in Onedrive. While i examined his code i decided i could make it a bit more modern with proactive remediation and write it in Powershell code.

My proactive remediation script consists of a detection and a remediation script.

### What does my detection script do?

First it gets the paths of all the locations the Outlook signature could be.

The outlook signatured is saved in this folder by Outlook.

```Powershell
#Windows directory
%APPDATA%\Roaming\Microsoft\Signatures 
```


