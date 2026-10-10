---
title: "Cyborg2 Writeup"
author: ekansh
date: 2026-10-09 04:02:15 +0530
categories: [Wargames, Cyborg]
tags: [underthewire, cyborg]
published : true
---

## Objective :

The password for cyborg3 is the host A record IP address for CYBORG718W100N PLUS the name of the file on the desktop.

## Solution : 

First we check for the name of the file present on the desktop as follows : 

```powershell
PS C:\users\cyborg2\desktop> Get-ChildItem                                                                                                                                                  
                                                                                                                                                                                            
                                                                                                                                                                                            
    Directory: C:\users\cyborg2\desktop                                                                                                                                                     
                                                                                                                                                                                            
                                                                                                                                                                                            
Mode                LastWriteTime         Length Name                                                                                                                                       
----                -------------         ------ ----                                                                                                                                       
-a----       10/26/2025   6:43 PM              0 _ipv4                                                                                                                                      
                                                                                                                                                                                            
                                                                                                                                                                                            
PS C:\users\cyborg2\desktop>   
```

We check for the contents of the file and we are presented with just blank characters. 
So moving ahead we search for the commands we could utilize to check the A record IP address for the given. 
Using the official Microsoft Docs at : https://learn.microsoft.com/en-us/powershell/module/dnsclient/resolve-dnsname?view=windowsserver2025-ps

We use the `Resolve-DnsName` function which basically performs a DNS name query resolution for the specified name.

```powershell
PS C:\users\cyborg2\desktop> Resolve-DnsName -Name CYBORG718W100N                                                                                                                           
                                                                                                                                                                                            
Name                                           Type   TTL   Section    IPAddress                                                                                                            
----                                           ----   ---   -------    ---------                                                                                                            
CYBORG718W100N.underthewire.tech               A      3600  Answer     172.31.45.167                                                                                                        
                                                                                                                                                                                            
                                                                                                                                                                                            
PS C:\users\cyborg2\desktop>  
```

Hence the password for the next level becomes : `172.31.45.167_ipv4`
