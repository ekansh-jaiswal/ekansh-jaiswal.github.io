---
title: "Cyborg12 Writeup"
author: ekansh
date: 2026-10-11 13:03:15 +0530
categories: [Wargames, Cyborg]
tags: [underthewire, cyborg]
published : true
---

## Objective :

The password for cyborg13 is the first four characters of the base64 encoded full path to the file that started the i_heart_robots service PLUS the name of the file on the desktop.

## Solution :

First we check for the file present on the desktop : 

```powershell
PS C:\users\cyborg12\desktop> gci                                                                                                                                                           
                                                                                                                                                                                            
                                                                                                                                                                                            
    Directory: C:\users\cyborg12\desktop                                                                                                                                                    
                                                                                                                                                                                            
                                                                                                                                                                                            
Mode                LastWriteTime         Length Name                                                                                                                                       
----                -------------         ------ ----                                                                                                                                       
-a----        8/30/2018  10:45 AM              0 _heart                                                                                                                                     
                                                                                                                                                                                            
                                                                                                                                                                                            
PS C:\users\cyborg12\desktop>   
```

Referring to docs at : https://learn.microsoft.com/en-us/powershell/scripting/samples/managing-services?view=powershell-7.6
We find / search for the required Service name as follows: 

```powershell
PS C:\users\cyborg12\desktop> Get-Service -Name *i_heart_robots*                                                                                                                            
                                                                                                                                                                                            
Status   Name               DisplayName                                                                                                                                                     
------   ----               -----------                                                                                                                                                     
Stopped  i_heart_robots     i_heart_robots                                                                                                                                                  
                                                                                                                                                                                            
                                                                                                                                                                                            
PS C:\users\cyborg12\desktop>  
```

After referring to the following StackOverflow question : https://stackoverflow.com/questions/59725591/powershell-get-service-detailed-description-of-the-windows-service
I got the following : 

```powershell
PS C:\users\cyborg12\desktop> Get-CimInstance Win32_Service | ?{$_.Name -like "i_heart_robots"} | select *                                                                                  
                                                                                                                                                                                            
                                                                                                                                                                                            
Name                    : i_heart_robots                                                                                                                                                    
Status                  : OK                                                                                                                                                                
ExitCode                : 1077                                                                                                                                                              
DesktopInteract         : False                                                                                                                                                             
ErrorControl            : Normal                                                                                                                                                            
PathName                : c:\windows\system32\cmd.exe                                                                                                                                       
ServiceType             : Own Process                                                                                                                                                       
StartMode               : Disabled                                                                                                                                                          
Caption                 : i_heart_robots                                                                                                                                                    
Description             : I be lovin some metal bots!                                                                                                                                       
InstallDate             :                                                                                                                                                                   
CreationClassName       : Win32_Service                                                                                                                                                     
Started                 : False                                                                                                                                                             
SystemCreationClassName : Win32_ComputerSystem                                                                                                                                              
SystemName              : UTW                                                                                                                                                               
AcceptPause             : False                                                                                                                                                             
AcceptStop              : False                                                                                                                                                             
DisplayName             : i_heart_robots                                                                                                                                                    
ServiceSpecificExitCode : 0                                                                                                                                                                 
StartName               : LocalSystem                                                                                                                                                       
State                   : Stopped                                                                                                                                                           
TagId                   : 0                                                                                                                                                                 
CheckPoint              : 0                                                                                                                                                                 
DelayedAutoStart        : False                                                                                                                                                             
ProcessId               : 0                                                                                                                                                                 
WaitHint                : 0                                                                                                                                                                 
PSComputerName          :                                                                                                                                                                   
CimClass                : root/cimv2:Win32_Service                                                                                                                                          
CimInstanceProperties   : {Caption, Description, InstallDate, Name...}                                                                                                                      
CimSystemProperties     : Microsoft.Management.Infrastructure.CimSystemProperties                                                                                                           
                                                                                                                                                                                            
                                                                                                                                                                                            
                                                                                                                                                                                            
PS C:\users\cyborg12\desktop>  
```
Hence the full path is : `c:\windows\system32\cmd.exe`
Let's Base64 encode this. 

```powershell
PS C:\users\cyborg12\desktop> [Convert]::ToBase64String([Text.Encoding]::UTF8.GetBytes($text))                                                                                              
Yzpcd2luZG93c1xzeXN0ZW0zMlxjbWQuZXhl                                                                                                                                                        
PS C:\users\cyborg12\desktop> [Convert]::ToBase64String([Text.Encoding]::UTF8.GetBytes("c:\windows\system32\cmd.exe"))                                                                      
Yzpcd2luZG93c1xzeXN0ZW0zMlxjbWQuZXhl                                                                                                                                                        
PS C:\users\cyborg12\desktop>  
```

But this output is wrong since I just noticed now that we had to use `Unicode` and not `UTF8`. 
So using the correct command we now get : 

```powershell
PS C:\users\cyborg12\desktop> [Convert]::ToBase64String([Text.Encoding]::Unicode.GetBytes("c:\windows\system32\cmd.exe"))                                                                   
YwA6AFwAdwBpAG4AZABvAHcAcwBcAHMAeQBzAHQAZQBtADMAMgBcAGMAbQBkAC4AZQB4AGUA                                                                                                                    
PS C:\users\cyborg12\desktop> 
```

Hence the correct answer is : `ywa6_heart`
