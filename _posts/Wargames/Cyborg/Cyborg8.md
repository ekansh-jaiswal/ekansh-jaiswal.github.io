---
title: "Cyborg8 Writeup"
author: ekansh
date: 2026-10-10 05:23:15 +0530
categories: [Wargames, Cyborg]
tags: [underthewire, cyborg]
published : true
---

## Objective :

The password for cyborg9 is the Internet zone that the picture on the desktop was downloaded from.

## Solution : 

The first thought in my mind came to the tool `exif` but then I remembered this is a Powershell challenge. 
So , I checked how to check an object's / file's metadata using powershell on Google. 

Turns out we have something known as a ADS (NTFS Alternate Data Stream) , which is basically a built in feature inside Windows , which allows a file to hold multiple , hidden sets of data
simultaneously. And it's quite interesting to use. It can even make a file show as size of 1KB while hiding a video of size 500MBs. 

Using the following article as reference : https://gist.github.com/chriselgee/bf41951d0b51d0ef9d2504a36921cd13

We run the following commands : 

```powershell 
PS C:\users\cyborg8\desktop> gi -Path .\1_qs5nwlcl7f_-SwNlQvOrAw.png -Stream *                                                                                                              
                                                                                                                                                                                            
                                                                                                                                                                                            
PSPath        : Microsoft.PowerShell.Core\FileSystem::C:\users\cyborg8\desktop\1_qs5nwlcl7f_-SwNlQvOrAw.png::$DATA                                                                          
PSParentPath  : Microsoft.PowerShell.Core\FileSystem::C:\users\cyborg8\desktop                                                                                                              
PSChildName   : 1_qs5nwlcl7f_-SwNlQvOrAw.png::$DATA                                                                                                                                         
PSDrive       : C                                                                                                                                                                           
PSProvider    : Microsoft.PowerShell.Core\FileSystem                                                                                                                                        
PSIsContainer : False                                                                                                                                                                       
FileName      : C:\users\cyborg8\desktop\1_qs5nwlcl7f_-SwNlQvOrAw.png                                                                                                                       
Stream        : :$DATA                                                                                                                                                                      
Length        : 60113                                                                                                                                                                       
                                                                                                                                                                                            
PSPath        : Microsoft.PowerShell.Core\FileSystem::C:\users\cyborg8\desktop\1_qs5nwlcl7f_-SwNlQvOrAw.png:Zone.Identifier                                                                 
PSParentPath  : Microsoft.PowerShell.Core\FileSystem::C:\users\cyborg8\desktop                                                                                                              
PSChildName   : 1_qs5nwlcl7f_-SwNlQvOrAw.png:Zone.Identifier                                                                                                                                
PSDrive       : C                                                                                                                                                                           
PSProvider    : Microsoft.PowerShell.Core\FileSystem                                                                                                                                        
PSIsContainer : False                                                                                                                                                                       
FileName      : C:\users\cyborg8\desktop\1_qs5nwlcl7f_-SwNlQvOrAw.png                                                                                                                       
Stream        : Zone.Identifier                                                                                                                                                             
Length        : 26                                                                                                                                                                          
                     
```

Here we can see 2 streams , 

1. `$DATA` : This is the Data from the actual PNG itself. So this stream contains the contents of the PNG file. 
2. `Zone.Identifier` : This is a small set of bytes which helps us track where this file originally came from. The zone ID can vary from 0-to-4. 

When trying to read the `Zone.Identifier` stream we get the following : 

```powershell
PS C:\users\cyborg8\desktop> Get-Content -Path .\1_qs5nwlcl7f_-SwNlQvOrAw.png -Stream Zone.Identifier                                                                                       
[ZoneTransfer]                                                                                                                                                                              
ZoneId=4                                                                                                                                                                                    
PS C:\users\cyborg8\desktop> 
```

Here `ZoneId=4` implies that the file came from a suspicious source which Windows doesn't likes. 
Also hence the password for the next level is `4`
