#### External Enumeration

#### Internal Enumeration
[WinRM](../../../services/WinRM.md)
##### [[../enumeration/internal/Privileged Access to Services|Privileged Access to Services]] to services
[[../../../services/Remote Desktop]]
```powershell-session
Get-NetLocalGroupMember -ComputerName MS01 -GroupName "Remote Desktop Users"
```
[[../../../services/WinRM]]

```powershell-session
Get-NetLocalGroupMember -ComputerName MS01 -GroupName "Remote Management Users"
```
[[../../../services/MSSQL]] with [[../../../tools/PowerUpSQL]]
```powershell-session
Get-SQLInstanceDomain
```
##### [[../enumeration/internal/Domain Trusts]]
```powershell-session
Get-ADDomain | select name,parentdomain
```

```powershell-session
Get-ADTrust -Filter *
```

```powershell-session
Get-DomainTrust 
```

With [[../../../tools/PowerView]]
```powershell-session
Get-DomainTrustMapping
```