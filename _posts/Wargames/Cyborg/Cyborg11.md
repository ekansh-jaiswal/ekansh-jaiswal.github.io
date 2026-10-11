---
title: "Cyborg11 Writeup"
author: ekansh
date: 2026-10-11 14:23:15 +0530
categories: [Wargames, Cyborg]
tags: [underthewire, cyborg]
published : true
---

## Objective :

The password for cyborg12 is located in the IIS log. The password is not Mozilla or Opera.

## Solution : 

Tried some of the following sources and ran their scripts but all of them failed mostly due to `insufficient perms` 

- https://quisitive.com/easy-iis-log-reading-with-powershell/
- https://stackoverflow.com/questions/26784883/get-iis-log-location-via-powershell

After referring to article https://stackoverflow.com/questions/6426375/iis-where-can-i-find-the-iis-logs
I managed to find the direcotry where the IIS logs are stored : 

```powershell
cd $env:SystemDrive\inetpub\logs\LogFiles                                                                                                               
PS C:\inetpub\logs\LogFiles> gci                                                                                                                                                            
                                                                                                                                                                                            
                                                                                                                                                                                            
    Directory: C:\inetpub\logs\LogFiles                                                                                                                                                     
                                                                                                                                                                                            
                                                                                                                                                                                            
Mode                LastWriteTime         Length Name                                                                                                                                       
----                -------------         ------ ----                                                                                                                                       
d-----        8/30/2018  10:45 AM                w3svc1                                                                                                                                     
                                                                                                                                                                                            
                                                                                                                                                                                            
PS C:\inetpub\logs\LogFiles> cd .\w3svc1                                                                                                                                                    
PS C:\inetpub\logs\LogFiles\w3svc1> gci                                                                                                                                                     
                                                                                                                                                                                            
                                                                                                                                                                                            
    Directory: C:\inetpub\logs\LogFiles\w3svc1                                                                                                                                              
                                                                                                                                                                                            
                                                                                                                                                                                            
Mode                LastWriteTime         Length Name                                                                                                                                       
----                -------------         ------ ----                                                                                                                                       
-a----        8/30/2018   5:52 AM        1099641 u_ex160413.log                                                                                                                             
                                                                                                                                                                                            
                                                                                                                                                                                            
PS C:\inetpub\logs\LogFiles\w3svc1>  
```

Upon a small inspection of the contents of the log we can see as follows : 

```powershell
PS C:\inetpub\logs\LogFiles\w3svc1> more .\u_ex160413.log                                                                                                                                   
#Software: Microsoft Internet Information Services 8.5                                                                                                                                      
#Version: 1.0                                                                                                                                                                               
#Date: 2016-04-13 04:14:01                                                                                                                                                                  
#Fields: date time s-sitename s-computername s-ip cs-method cs-uri-stem cs-uri-query s-port cs-username c-ip cs-version cs(User-Agent) cs(Cookie) cs(Referer) cs-host sc-status sc-substatus
 sc-win32-status sc-bytes cs-bytes time-taken                                                                                                                                               
2016-04-13 04:14:01 W3SVC1 Century 172.31.45.65 GET / - 80 - 172.31.45.65 HTTP/1.1 Mozilla/5.0+(compatible;+MSIE+9.0;+Windows+NT;+Windows+NT+6.2;+en-US) - - century.underthewire.tech 200 0
 0 925 158 3                                                                                                                                                                                
2016-04-13 04:14:01 W3SVC1 Century 172.31.45.65 GET / - 80 - 172.31.45.65 HTTP/1.1 Mozilla/5.0+(Windows+NT;+Windows+NT+6.2;+en-US)+Gecko/20100401+Firefox/4.0 - - century.underthewire.tech 
200 0 0 925 139 0                                                                                                                                                                           
2016-04-13 04:14:01 W3SVC1 Century 172.31.45.65 GET / - 80
[ . . . ]
```

Since the Password is not Morzilla or Opera , try to directly search for the string `Password` inside the file. 

```powershell
PS C:\inetpub\logs\LogFiles\w3svc1> Select-String -Path .\u_ex160413.log -Pattern "password"                                                                                                
                                                                                                                                                                                            
u_ex160413.log:2505:2016-04-13 04:14:12 W3SVC1 Century 172.31.45.65 GET / - 80 - 172.31.45.65 HTTP/1.1 LordHelmet/5.0+(CombTheDesert)+Password+is:spaceballs - - century.underthewire.tech  
200 0 0 925 118 0                                                                                                                                                                           
                                                                                                                                                                                            
                                                                                                                                                                                            
PS C:\inetpub\logs\LogFiles\w3svc1>   
```

And we are greeted with the password sucessfully!!. The password is `spaceballs`
