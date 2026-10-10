---
title: "Cyborg1 Writeup"
author: ekansh
date: 2026-10-09 03:52:15 +0530
categories: [Wargames, Cyborg]
tags: [underthewire, cyborg]
published : true
---

## Objective :

The password for cyborg2 is the state that the user Chris Rogers is from as stated within Active Directory.

## Solution : 

Let's use the `Get-Help` command to check for any methods / functions to access the users in AD . 

```powershell
PS C:\users\cyborg1\desktop> get-help "User"                                                                                                                                                
                                                                                                                                                                                            
Name                              Category  Module                    Synopsis                                                                                                              
----                              --------  ------                    --------                                                                                                              
Get-SlackUser                     Function  PSSlack                   ...                                                                                                                   
Get-SlackUserGroup                Function  PSSlack                   ...                                                                                                                   
Get-SlackUserMap                  Function  PSSlack                   ...                                                                                                                   
New-ADUser                        Cmdlet    ActiveDirectory           New-ADUser...                                                                                                         
Get-ADUser                        Cmdlet    ActiveDirectory           Get-ADUser...                                                                                                         
Remove-ADUser                     Cmdlet    ActiveDirectory           Remove-ADUser...                                                                                                      
Get-ADUserResultantPasswordPolicy Cmdlet    ActiveDirectory           Get-ADUserResultantPasswordPolicy...                                                                                  
Set-ADUser                        Cmdlet    ActiveDirectory           Set-ADUser...                                                                                                         
Set-WinUserLanguageList           Cmdlet    International             Set-WinUserLanguageList...     
[ . . . ] 
```

Let's understand how thee `Get-ADUser` command works. 

```powershell
-------------------------- EXAMPLE 1 --------------------------                                                                                                                         
                                                                                                                                                                                            
    C:\PS>Get-ADUser -Filter * -SearchBase "OU=Finance,OU=UserAccounts,DC=FABRIKAM,DC=COM"                                                                                                  
                                                                                                                                                                                            
    Description                                                                                                                                                                             
                                                                                                                                                                                            
    -----------                                                                                                                                                                             
                                                                                                                                                                                            
    Get all users under the container 'OU=Finance,OU=UserAccounts,DC=FABRIKAM,DC=COM'.                                                                                                      
    -------------------------- EXAMPLE 2 --------------------------                                                                                                                         
                                                                                                                                                                                            
    C:\PS>Get-ADUser -Filter 'Name -like "*SvcAccount"' | FT Name,SamAccountName -A                                                                                                         
                                                                                                                                                                                            
                                                                                                                                                                                            
    Name             SamAccountName                                                                                                                                                         
    ----             --------------                                                                                                                                                         
    SQL01 SvcAccount SQL01                                                                                                                                                                  
    SQL02 SvcAccount SQL02                                                                                                                                                                  
    IIS01 SvcAccount IIS01                                                                                                                                                                  
                                                                                                                                                                                            
    Description                                                                                                                                                                             
                                                                                                                                                                                            
    -----------                                                                                                                                                                             
                                                                                                                                                                                            
    Get all users that have a name that ends with 'SvcAccount'.                                                                                                                             
    -------------------------- EXAMPLE 3 --------------------------                                                                                                                         
                                                                                                                                                                                            
    C:\PS>Get-ADUser GlenJohn -Properties *                                                                                                                                                 
                                                                                                                                                                                            
                                                                                                                                                                                            
    Surname           : John                                                                                                                                                                
    Name              : Glen John                                                                                                                                                           
    UserPrincipalName :                                                                                                                                                                     
    GivenName         : Glen                                                                                                                                                                
    Enabled           : False                                                                                                                                                               
    SamAccountName    : GlenJohn                                                                                                                                                            
    ObjectClass       : user                                                                                                                                                                
    SID               : S-1-5-21-2889043008-4136710315-2444824263-3544                                                                                                                      
    ObjectGUID        : e1418d64-096c-4cb0-b903-ebb66562d99d                                                                                                                                
    DistinguishedName : CN=Glen John,OU=NorthAmerica,OU=Sales,OU=UserAccounts,DC=FABRIKAM,DC=COM                                                                                            
                                                                                                                                                                                            
    Description                                                                                                                                                                             
                                                                                                                                                                                            
    -----------                                                                                                                                                                             
                                                                                                                                                                                            
    Get all properties of the user with samAccountName 'GlenJohn'. 
```

Using these 3 examples as reference we can run the following commands : 

```powershell
PS C:\users\cyborg1\desktop> get-aduser                                                                                                                                                     
                                                                                                                                                                                            
cmdlet Get-ADUser at command pipeline position 1                                                                                                                                            
Supply values for the following parameters:                                                                                                                                                 
(Type !? for Help.)                                                                                                                                                                         
Filter: Name -like "*Chris*"                                                                                                                                                                
                                                                                                                                                                                            
                                                                                                                                                                                            
DistinguishedName : CN=Rogers\, Chris\ ,OU=T-65,OU=X-Wing,DC=underthewire,DC=tech                                                                                                           
Enabled           : False                                                                                                                                                                   
GivenName         : Chris                                                                                                                                                                   
Name              : Rogers, Chris                                                                                                                                                           
ObjectClass       : user                                                                                                                                                                    
ObjectGUID        : ee6450f8-cf70-4b1d-b902-a837839632bd                                                                                                                                    
SamAccountName    : chris.rogers                                                                                                                                                            
SID               : S-1-5-21-758131494-606461608-3556270690-2177                                                                                                                            
Surname           : Rogers                                                                                                                                                                  
UserPrincipalName : chris.rogers                                                                                                                                                            
                                                                                                                                                                                            
PS C:\users\cyborg1\desktop> get-aduser chris.rogers -Properties *                                                                                                                          
                                                                                                                                                                                            
                                                                                                                                                                                            
AccountExpirationDate                :                                                                                                                                                      
accountExpires                       : 9223372036854775807                                                                                                                                  
AccountLockoutTime                   :                                                                                                                                                      
AccountNotDelegated                  : False                                                                                                                                                
AllowReversiblePasswordEncryption    : False                      
[ . . . ]
SmartcardLogonRequired               : False                                                                                                                                                
sn                                   : Rogers                                                                                                                                               
st                                   : kansas                                                                                                                                               
State                                : kansas                                                                                                                                               
StreetAddress                        :                                                                                                                                                      
Surname                              : Rogers                                                                                                                                               
Title                                :                                                                                                                                                      
TrustedForDelegation                 : False
[ . . .]
```

There we go. We have the answer as `kansas`. :) 
