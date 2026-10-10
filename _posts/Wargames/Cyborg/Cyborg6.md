---
title: "Cyborg6 Writeup"
author: ekansh
date: 2026-10-10 00:47:15 +0530
categories: [Wargames, Cyborg]
tags: [underthewire, cyborg]
published : true
---

## Objective :

The password for cyborg7 is the decoded text of the string within the file on the desktop.

## Solution : 

We check for the file and it's content present on the desktop : 

```powershell
PS C:\users\cyborg6\desktop> gci                                                                                                                                                            
                                                                                                                                                                                            
                                                                                                                                                                                            
    Directory: C:\users\cyborg6\desktop                                                                                                                                                     
                                                                                                                                                                                            
                                                                                                                                                                                            
Mode                LastWriteTime         Length Name                                                                                                                                       
----                -------------         ------ ----                                                                                                                                       
-a----         5/9/2026   7:23 PM             32 cypher.txt                                                                                                                                 
                                                                                                                                                                                            
PS C:\users\cyborg6\desktop> more .\cypher.txt                                                                                                                                              
YwB5AGIAZQByAGcAZQBkAGQAbwBuAA==                                                                                                                                                            
                                                                                                                                                                                            
PS C:\users\cyborg6\desktop>   
```

This text clearly looks like a base64 encoded string. (Because of the `==` at the end of the string) 
Now there are many ways to decrypt this Base64 String. 
We can do either of the following : 

1. Use any online available base64 decryptor for this. Such as CyberChef
2. Use the built-in Powershell tool for this by referring to the article at : https://stackoverflow.com/questions/15414678/how-to-decode-a-base64-string

```powershell
PS C:\users\cyborg6\desktop> [System.Text.Encoding]::ASCII.GetString([System.Convert]::FromBase64String($text))                                                                             
c y b e r g e d d o n                                                                                                                                                                       
PS C:\users\cyborg6\desktop> 
```

Hence the password for the next level is : `cybergeddon`
