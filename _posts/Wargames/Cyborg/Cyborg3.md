---
title: "Cyborg3 Writeup"
author: ekansh
date: 2026-10-09 04:12:15 +0530
categories: [Wargames, Cyborg]
tags: [underthewire, cyborg]
published : true
---

## Objective :

The password for cyborg4 is the number of users in the Cyborg group within Active Directory PLUS the name of the file on the desktop.

## Solution : 

We search for the file on the desktop and we get : 

```powershell
PS C:\users\cyborg3\desktop> gci                                                                                                                                                            
                                                                                                                                                                                            
                                                                                                                                                                                            
    Directory: C:\users\cyborg3\desktop                                                                                                                                                     
                                                                                                                                                                                            
                                                                                                                                                                                            
Mode                LastWriteTime         Length Name                                                                                                                                       
----                -------------         ------ ----                                                                                                                                       
-a----        2/26/2022   2:14 PM              0 _objects                                                                                                                                   
                                                                                                                                                                                            
                                                                                                                                                                                            
PS C:\users\cyborg3\desktop>  
```

`gci` is basically the short form for `Get-ChildItem` btw. 
Now we search for all AD commands available and we find the following : 

```powershell
Get-ADGroupMember                 Cmdlet    ActiveDirectory           Get-ADGroupMember...   
```

Using this command from it's examples : 

```powershell
PS C:\users\cyborg3\desktop> $A = Get-ADGroupMember "Cyborg"                                                                                                                                
PS C:\users\cyborg3\desktop> $A.count                                                                                                                                                       
88                                                                                                                                                                                          
PS C:\users\cyborg3\desktop>  
```

Hence the password for the next level becomes `88_objects`
