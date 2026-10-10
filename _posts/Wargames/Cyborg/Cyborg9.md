---
title: "Cyborg9 Writeup"
author: ekansh
date: 2026-10-11 01:23:15 +0530
categories: [Wargames, Cyborg]
tags: [underthewire, cyborg]
published : true
---

## Objective :

The password for cyborg10 is the first name of the user with the phone number of 876-5309 listed in Active Directory PLUS the name of the file on the desktop.

## Solution : 

We check for the file present on the desktop as follows : 

```powershell
PS C:\users\cyborg9\desktop> gci                                                                                                                                                            
                                                                                                                                                                                            
                                                                                                                                                                                            
    Directory: C:\users\cyborg9\desktop                                                                                                                                                     
                                                                                                                                                                                            
                                                                                                                                                                                            
Mode                LastWriteTime         Length Name                                                                                                                                       
----                -------------         ------ ----                                                                                                                                       
-a----        8/30/2018  10:45 AM              0 99                                                                                                                                         
                                                                                                                                                                                            
                                                                                                                                                                                            
PS C:\users\cyborg9\desktop>   
```

Now using the article present at : https://learn.microsoft.com/en-us/powershell/module/activedirectory/?view=windowsserver2016-ps


