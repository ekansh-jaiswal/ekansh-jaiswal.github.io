---
title: "Cyborg13 Writeup"
author: ekansh
date: 2026-10-11 15:53:15 +0530
categories: [Wargames, Cyborg]
tags: [underthewire, cyborg]
published : true
---

## Objective :

The password cyborg14 is the number of days the refresh interval is set to for DNS aging for the underthewire.tech zone PLUS the name of the file on the desktop.

## Solution :

We fetch the file name present on the desktop : 

```powershell
PS C:\users\cyborg13\desktop> gci                                                                                                                                                           
                                                                                                                                                                                            
                                                                                                                                                                                            
    Directory: C:\users\cyborg13\desktop                                                                                                                                                    
                                                                                                                                                                                            
                                                                                                                                                                                            
Mode                LastWriteTime         Length Name                                                                                                                                       
----                -------------         ------ ----                                                                                                                                       
-a----        8/30/2018  10:45 AM              0 _days                                                                                                                                      
                                                                                                                                                                                            
                                                                                                                                                                                            
PS C:\users\cyborg13\desktop>     
```

And now for check the DNS aging for the given domain , using the docs at : https://learn.microsoft.com/en-us/powershell/module/dnsserver/get-dnsserverzoneaging?view=windowsserver2025-ps

```powershell
PS C:\users\cyborg13\desktop> Get-DnsServerZoneAging -Name underthewire.tech                                                                                                                
                                                                                                                                                                                            
                                                                                                                                                                                            
ZoneName             : underthewire.tech                                                                                                                                                    
AgingEnabled         : True                                                                                                                                                                 
AvailForScavengeTime : 9/21/2018 10:00:00 AM                                                                                                                                                
RefreshInterval      : 22.00:00:00                                                                                                                                                          
NoRefreshInterval    : 7.00:00:00                                                                                                                                                           
ScavengeServers      :                                                                                                                                                                      
                                                                                                                                                                                            
                                                                                                                                                                                            
                                                                                                                                                                                            
PS C:\users\cyborg13\desktop>  
```

Hence the answer is : `22_days`
