---
title: "Cyborg4 Writeup"
author: ekansh
date: 2026-10-09 05:10:15 +0530
categories: [Wargames, Cyborg]
tags: [underthewire, cyborg]
published : true
---

## Objective :

The password for cyborg5 is the PowerShell module name with a version number of 8.9.8.9 PLUS the name of the file on the desktop.

## Solution : 

The file name on the desktop is : 

```powershell
PS C:\users\cyborg4\desktop> gci                                                                                                                                                            
                                                                                                                                                                                            
                                                                                                                                                                                            
    Directory: C:\users\cyborg4\desktop                                                                                                                                                     
                                                                                                                                                                                            
                                                                                                                                                                                            
Mode                LastWriteTime         Length Name                                                                                                                                       
----                -------------         ------ ----                                                                                                                                       
-a----        8/30/2018  10:45 AM              0 _eggs                                                                                                                                      
                                                                                                                                                                                            
                                                                                                                                                                                            
PS C:\users\cyborg4\desktop>     
```

Searching for methods to find the name of module in powershell we find about the following : 

```powershell
PS C:\users\cyborg4\desktop> get-help "module"                                                                                                                                              
                                                                                                                                                                                            
Name                              Category  Module                    Synopsis                                                                                                              
----                              --------  ------                    --------                                                                                                              
ImportSystemModules               Function                            ...                                                                                                                   
Export-ModuleMember               Cmdlet    Microsoft.PowerShell.Core Specifies the module members that are exported.                                                                       
Get-Module                        Cmdlet    Microsoft.PowerShell.Core Gets the modules that have been imported or that can be imported into the current session.                            
Import-Module                     Cmdlet    Microsoft.PowerShell.Core Adds modules to the current session.                                                                                  
New-Module                        Cmdlet    Microsoft.PowerShell.Core Creates a new dynamic module that exists only in memory.                                                              
New-ModuleManifest                Cmdlet    Microsoft.PowerShell.Core Creates a new module manifest.                                                                                        
Remove-Module                     Cmdlet    Microsoft.PowerShell.Core Removes modules from the current session.                                                                             
Test-ModuleManifest               Cmdlet    Microsoft.PowerShell.Core Verifies that a module manifest file accurately describes the contents of a module.                                   
InModuleScope                     Function  Pester                    ...                                                                                                                   
Uninstall-Module                  Function  PowerShellGet             ...                                                                                                                   
Install-Module                    Function  PowerShellGet             ...                                                                                                                   
Publish-Module                    Function  PowerShellGet             ...                                                                                                                   
Update-ModuleManifest             Function  PowerShellGet             ...                                                                                                                   
Save-Module                       Function  PowerShellGet             ...                                                                                                                   
Update-Module                     Function  PowerShellGet             ...                                                                                                                   
Get-InstalledModule               Function  PowerShellGet             ...                                                                                                                   
Find-Module                       Function  PowerShellGet             ...     
[ . . .]
```

Using the examples found from the manual using the `Get-Help Find-Module -full` command we perform the following : 

```powershell
PS C:\users\cyborg4\desktop> Find-Module -RequiredVersion 8.9.8.9                                                                                                                           
Find-Module : The RequiredVersion, MinimumVersion, MaximumVersion or AllVersions parameters are allowed only when you specify a single name as the value of the Name parameter, without     
any wildcard characters.                                                                                                                                                                    
At line:1 char:1                                                                                                                                                                            
+ Find-Module -RequiredVersion 8.9.8.9                                                                                                                                                      
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~                                                                                                                                                      
    + CategoryInfo          : InvalidArgument: (:) [Find-Module], ArgumentException                                                                                                         
    + FullyQualifiedErrorId : VersionParametersAreAllowedOnlyWithSingleName,Find-Module                                                                                                     
                                                                                                                                                                                            
PS C:\users\cyborg4\desktop> Find-Module *                                                                                                                                                  
PS C:\users\cyborg4\desktop> Find-Module                                                                                                                                                    
PS C:\users\cyborg4\desktop> Find-Module -Name "Powershell" -RequiredVersion 8.9.8.9                                                                                                        
PackageManagement\Find-Package : No match was found for the specified search criteria and module name 'Powershell'. Try Get-PSRepository to see all available registered module             
repositories.                                                                                                                                                                               
At C:\Program Files\WindowsPowerShell\Modules\PowerShellGet\1.0.0.1\PSModule.psm1:1360 char:3                                                                                               
+         PackageManagement\Find-Package @PSBoundParameters | Microsoft ...                                                                                                                 
+         ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~                                                                                                                                 
    + CategoryInfo          : ObjectNotFound: (Microsoft.Power...ets.FindPackage:FindPackage) [Find-Package], Exception                                                                     
    + FullyQualifiedErrorId : NoMatchFoundForCriteria,Microsoft.PowerShell.PackageManagement.Cmdlets.FindPackage  
```

This gives an interesting error , Powershell suggests me to use the `Get-PSRepository` function. Which upon running gets me the following : 

```powershell
                                                                                                                                                                                            
PS C:\users\cyborg4\desktop> Get-PSRepository                                                                                                                                               
WARNING: Unable to find module repositories.                                                                                                                                                
PS C:\users\cyborg4\desktop>  
```

After trying some more commands to no avail. I referred to the Powershell docs from Microsoft Learn website at : https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/get-module?view=powershell-7.6

```powershell
PS C:\users\cyborg4\desktop> Get-Module -ListAvailable  
[ . . . ]
Manifest   8.9.8.9    bacon                               Get-bacon     
[ . . . ]
```

This is one way where we could manually read the list and find our required version. 
A smarter version would be : 

```powershell
PS C:\users\cyborg4\desktop> Get-Module -ListAvailable  | Where-Object -Property Version -eq "8.9.8.9"                                                                                      
                                                                                                                                                                                            
                                                                                                                                                                                            
    Directory: C:\Windows\system32\WindowsPowerShell\v1.0\Modules                                                                                                                           
                                                                                                                                                                                            
                                                                                                                                                                                            
ModuleType Version    Name                                ExportedCommands                                                                                                                  
---------- -------    ----                                ----------------                                                                                                                  
Manifest   8.9.8.9    bacon                               Get-bacon                                                                                                                         
                                                                                                                                                                                            
                                                                                                                                                                                            
PS C:\users\cyborg4\desktop>  
```

Hence the password for our next level becomes : `bacon_eggs`
