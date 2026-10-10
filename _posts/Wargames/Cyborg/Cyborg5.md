---
title: "Cyborg5 Writeup"
author: ekansh
date: 2026-10-09 23:47:15 +0530
categories: [Wargames, Cyborg]
tags: [underthewire, cyborg]
published : true
---

## Objective :

The password for cyborg6 is the last name of the user who has logon hours set on their account PLUS the name of the file on the desktop.

## Solution : 

The name of the file on the desktop is as follows : 
```powershell
PS C:\users\cyborg5\desktop> gci                                                                                                                                                            
                                                                                                                                                                                            
                                                                                                                                                                                            
    Directory: C:\users\cyborg5\desktop                                                                                                                                                     
                                                                                                                                                                                            
                                                                                                                                                                                            
Mode                LastWriteTime         Length Name                                                                                                                                       
----                -------------         ------ ----                                                                                                                                       
-a----        8/30/2018  10:45 AM              0 _timer                                                                                                                                     
                                                                                                                                                                                            
                                                                                                                                                                                            
PS C:\users\cyborg5\desktop>   
```

Now moving on to checking the user for logon hours set on his/her account. 
Using the article from StackOverflow : https://stackoverflow.com/questions/73629128/how-to-list-users-with-logonhours-denied
But this didn't work directly. For the article to be useful I first had to filter out all the users available myself. 

So I ran the following : 

```powershell
PS C:\users\cyborg5\desktop> $A = Get-ADUser -Filter *                                                                                                                                      
PS C:\users\cyborg5\desktop> $A.count                                                                                                                                                       
1080                                                                                                                                                                                        
PS C:\users\cyborg5\desktop> 
```

Whoa that's a lot lot of users to handle . . 
After multiple unsuccessfull attempts to fetch , I finally managed to find the list as follows : 

```powershell
PS C:\users\cyborg5\desktop> Get-ADUser -Filter * -Properties LogonHours | Format-Table -Property Name,LogonHours   
Name                      LogonHours                                                                                                                                                        
----                      ----------                                                                                                                                                        
Administrator             {255, 255, 255, 255...} 
[ . . . ] 
Rowray, Benny             {0, 0, 0, 0...}
[ . . . ]
```

Had to scroll a long way to find this. 
Hence the password for the next level is : `renny_timer`
