#### What is it?
Sometimes user passwords are stored using RC4 encryption. The key used to encrypt them is stored in the registry and can be accessed by a Domain Admin.

#### Prerequisites
 - Domain Admin
 - "Store passwords using reversible encryption" option is set


#### Enumeration
```powershell-session
Get-ADUser -Filter 'userAccountControl -band 128' -Properties userAccountControl
```

```powershell-session
PS C:\htb> Get-ADUser -Filter 'userAccountControl -band 128' -Properties userAccountControl

DistinguishedName  : CN=PROXYAGENT,OU=Service Accounts,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
Enabled            : True
GivenName          :
Name               : PROXYAGENT
ObjectClass        : user
ObjectGUID         : c72d37d9-e9ff-4e54-9afa-77775eaaf334
SamAccountName     : proxyagent
SID                : S-1-5-21-3842939050-3880317879-2865463114-5222
Surname            :
userAccountControl : 640
UserPrincipalName  :
```

With [[../../../tools/PowerView]]
```powershell-session
Get-DomainUser -Identity * | ? {$_.useraccountcontrol -like '*ENCRYPTED_TEXT_PWD_ALLOWED*'} |select samaccountname,useraccountcontrol
```

```powershell-session
PS C:\htb> Get-DomainUser -Identity * | ? {$_.useraccountcontrol -like '*ENCRYPTED_TEXT_PWD_ALLOWED*'} |select samaccountname,useraccountcontrol

samaccountname                         useraccountcontrol
--------------                         ------------------
proxyagent     ENCRYPTED_TEXT_PWD_ALLOWED, NORMAL_ACCOUNT
```

Can be decrypted with [[../../../tools/Impacket/secretsdump.py]] when performing a [[DCSync]] attack