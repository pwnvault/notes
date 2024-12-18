#### Using netdom to query domain trust
```cmd-session
netdom query /domain:inlanefreight.local trust
```

```cmd-session
C:\htb> netdom query /domain:inlanefreight.local trust
Direction Trusted\Trusting domain                         Trust type
========= =======================                         ==========

<->       LOGISTICS.INLANEFREIGHT.LOCAL
Direct
 Not found

<->       FREIGHTLOGISTICS.LOCAL
Direct
 Not found

The command completed successfully.
```

#### Using netdom to query domain controllers

```cmd-session
netdom query /domain:inlanefreight.local dc
```

```cmd-session
C:\htb> netdom query /domain:inlanefreight.local dc
List of domain controllers with accounts in the domain:

ACADEMY-EA-DC01
The command completed successfully.
```

#### Using netdom to query workstations and servers

```cmd-session
netdom query /domain:inlanefreight.local workstation
```

```cmd-session
C:\htb> netdom query /domain:inlanefreight.local workstation
List of workstations with accounts in the domain:

ACADEMY-EA-MS01
ACADEMY-EA-MX01      ( Workstation or Server )

SQL01      ( Workstation or Server )
ILF-XRG      ( Workstation or Server )
MAINLON      ( Workstation or Server )
CISERVER      ( Workstation or Server )
INDEX-DEV-LON      ( Workstation or Server )
...SNIP...
```