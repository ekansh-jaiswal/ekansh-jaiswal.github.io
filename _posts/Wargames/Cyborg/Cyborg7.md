---
title: "Cyborg7 Writeup"
author: ekansh
date: 2026-10-10 00:53:15 +0530
categories: [Wargames, Cyborg]
tags: [underthewire, cyborg]
published : true
---

## Objective :

The password for cyborg8 is the executable name of a program that will start automatically when cyborg7 logs in.

## Solution : 

By referring to the article from Microsoft Learn at : https://devblogs.microsoft.com/scripting/use-powershell-to-provide-startup-information/

```powershell
PS C:\users\cyborg7\desktop> Get-WmiObject Win32_StartupCommand | Select-Object Name,Command,User | Format-List                                                                             
PS C:\users\cyborg7\desktop> Get-CimInstance Win32_StartupCommand | Select-Object Name, Command, Location, User | Format-Table -AutoSize                                                    
PS C:\users\cyborg7\desktop>                                                                                                                                                                
```

Yeah that's very odd . . . 
Let's Use AI this time.
Turns out we need Administrator Privileges in order to get any output from the commands above. For a normal user case we need to use the following command : 
(Actually Had to try a lot of commands from the ones which AI gave me since many didn't work as shown below) 

```powershell
PS C:\users\cyborg7\desktop> Get-WmiObject Win32_StartupCommand | Select-Object Name,Command,User | Format-List                                                                             
PS C:\users\cyborg7\desktop> Get-CimInstance Win32_StartupCommand | Select-Object Name, Command, Location, User | Format-Table -AutoSize                                                    
PS C:\users\cyborg7\desktop> Get-ChildItem "$env:APPDATA\Microsoft\Windows\Start Menu\Programs\Startup"                                                                                     
PS C:\users\cyborg7\desktop> Get-ChildItem "$env:ProgramData\Microsoft\Windows\Start Menu\Programs\Startup" -ErrorAction SilentlyContinue                                                   
PS C:\users\cyborg7\desktop> Get-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" -ErrorAction SilentlyContinue                                                     
PS C:\users\cyborg7\desktop> Get-ItemProperty -Path "HKLM:\Software\Microsoft\Windows\CurrentVersion\Run" -ErrorAction SilentlyContinue                                                     
PS C:\users\cyborg7\desktop> Get-ScheduledTask | Where-Object { $_.Triggers.TriggerType -eq 9 } | Select-Object TaskName, State, TaskPath                                                   
PS C:\users\cyborg7\desktop> Get-ItemProperty -Path "HKLM:\Software\WOW6432Node\Microsoft\Windows\CurrentVersion\Run"                                                                       
                                                                                                                                                                                            
                                                                                                                                                                                            
Bitvise SSH Server Activation State Checker : "C:\Program Files\Bitvise SSH Server\BssActStateCheck.exe"                                                                                    
PSPath                                      : Microsoft.PowerShell.Core\Registry::HKEY_LOCAL_MACHINE\Software\WOW6432Node\Microsoft\Windows\CurrentVersion\Run                              
PSParentPath                                : Microsoft.PowerShell.Core\Registry::HKEY_LOCAL_MACHINE\Software\WOW6432Node\Microsoft\Windows\CurrentVersion                                  
PSChildName                                 : Run                                                                                                                                           
PSDrive                                     : HKLM                                                                                                                                          
PSProvider                                  : Microsoft.PowerShell.Core\Registry                                                                                                            
                                                                                                                                                                                            
                                                                                                                                                                                            
                                                                                                                                                                                            
PS C:\users\cyborg7\desktop> Get-ItemProperty -Path "HKLM:\Software\Microsoft\Windows\CurrentVersion\Run"                                                                                   
PS C:\users\cyborg7\desktop> whoami                                                                                                                                                         
underthewire\cyborg7                                                                                                                                                                        
PS C:\users\cyborg7\desktop>  
```
	
After trying soo many different commands only 1 of them worked. So let's understand why only "that" command worked while the rest failed? 
[ Explanation given by AI ] 

The `whoami` output shows you are navigating the UnderTheWire Cyborg wargame environment. In these CTF scenarios, creators deliberately hide target executables in secondary locations to test your understanding of Windows architecture.

Here is exactly why each method failed or succeeded:

**Why the WMI and CIM commands failed:**

* `Get-WmiObject Win32_StartupCommand`
* `Get-CimInstance Win32_StartupCommand`
**Reason:** The `Win32_StartupCommand` class is a legacy WMI provider. It is hardcoded to look at specific standard 64-bit registry keys and startup folders. It is notoriously "blind" to the 32-bit registry hive (`WOW6432Node`) on a 64-bit operating system. Because the target app was stored in the 32-bit hive, WMI completely missed it.

**Why the Startup Folder commands failed:**

* `Get-ChildItem "$env:APPDATA\...\Startup"`
* `Get-ChildItem "$env:ProgramData\...\Startup"`
**Reason:** These check the physical file system directories where users can drop shortcut (`.lnk`) files to make programs run at boot. The server administrator simply left these directories empty.

**Why the standard Registry commands failed:**

* `Get-ItemProperty -Path "HKCU:\...\Run"`
* `Get-ItemProperty -Path "HKLM:\...\Run"`
**Reason:** These query the native 64-bit registry paths for the Current User and Local Machine. They failed because there are genuinely no 64-bit startup applications configured on this server.

**Why the Scheduled Task command failed:**

* `Get-ScheduledTask | Where-Object ...`
**Reason:** Apps can bypass the registry entirely by using the Windows Task Scheduler to trigger an execution "At log on". The administrator did not set up any tasks this way.

**Why the WOW6432Node command WORKED:**

* `Get-ItemProperty -Path "HKLM:\Software\WOW6432Node\Microsoft\Windows\CurrentVersion\Run"`
**Reason:** WOW6432Node stands for "Windows 32-bit on Windows 64-bit". Because modern Windows is 64-bit, it creates a separate registry hive to isolate older 32-bit applications. When a 32-bit program (like the specific version of Bitvise installed on this server) tells Windows to run it at startup, Windows automatically redirects that request away from the main `Run` key and buries it inside `WOW6432Node`. You had to explicitly query this redirected path to find it.

Now let's check if the password returned is correct or not . Turns out this is not correct as well now. :( 

So let's manually dig into all this again. 
Googling and referring to docs at : 

1. https://support.microsoft.com/en-us/windows/experience/startup-boot/configure-startup-applications-in-windows
2. https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-itemproperty?view=powershell-7.6
3. https://learn.microsoft.com/en-us/powershell/scripting/samples/working-with-registry-entries?view=powershell-7.6
4. https://stackoverflow.com/questions/15511809/how-do-i-get-the-value-of-a-registry-key-and-only-the-value-using-powershell

So , after testing several sources for reference. I realized it's actually a problem of the level itself right now. 
Referring to a writeup at : https://benheater.com/underthewire-cyborg/

I realized this problem and reported it. While the answer for this level is `skynet` and the command for getting this output was : `Get-CimInstance Win32_StartupCommand`
