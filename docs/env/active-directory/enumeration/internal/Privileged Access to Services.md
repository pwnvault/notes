
#### Using [[../../../../tools/PowerView]]

###### [[../../../../services/Remote Desktop]] Users Group

```powershell-session
Get-NetLocalGroupMember -ComputerName MS01 -GroupName "Remote Desktop Users"
```
```
Output
ComputerName : ACADEMY-EA-MS01
GroupName    : Remote Management Users
MemberName   : INLANEFREIGHT\forend
SID          : S-1-5-21-3842939050-3880317879-2865463114-5614
IsGroup      : False
IsDomain     : UNKNOWN
```

###### [[../../../../services/WinRM]]

```powershell-session
Get-NetLocalGroupMember -ComputerName MS01 -GroupName "Remote Management Users"
```

```powershell-session
ComputerName : ACADEMY-EA-MS01
GroupName    : Remote Management Users
MemberName   : INLANEFREIGHT\forend
SID          : S-1-5-21-3842939050-3880317879-2865463114-5614
IsGroup      : False
IsDomain     : UNKNOWN
```

###### Enumerating [[../../../../services/MSSQL]] Instances with [[../../../../tools/PowerUpSQL]]
```powrshell-session
Import-Module .\PowerUpSQL.ps1
```

```powershell-session
Get-SQLInstanceDomain
```

```powershell-session

ComputerName     : ACADEMY-EA-DB01.INLANEFREIGHT.LOCAL
Instance         : ACADEMY-EA-DB01.INLANEFREIGHT.LOCAL,1433
DomainAccountSid : 1500000521000170152142291832437223174127203170152400
DomainAccount    : damundsen
DomainAccountCn  : Dana Amundsen
Service          : MSSQLSvc
Spn              : MSSQLSvc/ACADEMY-EA-DB01.INLANEFREIGHT.LOCAL:1433
LastLogon        : 4/6/2022 11:59 AM
```

###### Running commands against SQL Server
```powershell-session
Get-SQLQuery -Verbose -Instance "172.16.5.150,1433" -username "inlanefreight\damundsen" -password "SQL1234!" -query 'Select @@version'
```

```powershell-session
VERBOSE: 172.16.5.150,1433 : Connection Success.

Column1
-------
Microsoft SQL Server 2017 (RTM) - 14.0.1000.169 (X64) ...
```
#### Using BloodHound

###### Custom Cypher Query WinRM
```cypher
MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) MATCH p2=(u1)-[:CanPSRemote*1..]->(c:Computer) RETURN p2
```

###### SQL Server Admin
```cypher
MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) MATCH p2=(u1)-[:SQLAdmin*1..]->(c:Computer) RETURN p2
```
###### Analysis/Prebuilt
`Find Workstations where Domain Users can RDP` 
`Find Servers where Domain Users can RDP`
`SQL Admin Rights` in the `Node Info` tab