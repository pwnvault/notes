
Forward data recieved on port 8080 to port 3389 on remote host.
```cmd-session
netsh.exe interface portproxy add v4tov4 listenport=8080 listenaddress=10.129.15.150 connectport=3389 connectaddress=172.16.5.25
```
Verify
```cmd-session
netsh.exe interface portproxy show v4tov4
```
Results
```cmd-session
Listen on ipv4:             Connect to ipv4:

Address         Port        Address         Port
--------------- ----------  --------------- ----------
10.129.42.198   8080        172.16.5.25     3389
```