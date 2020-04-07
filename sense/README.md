# Sense

### Machine Info
![](screenshots/machine.png)

#### Nmap
![](screenshots/nmap.png)

##### HTTP (Port 80)
When I access to port 80 it redirects to https server (port443).

##### HTTPS (Port 443)
Index Page:
![](screenshots/index.png)

I tried to login with default/common credentials but couldn't login.

###### Gobuster
![](screenshots/gobuster.png)


Changelog.txt:
![](screenshots/changelog.png)


system-users.txt:
![](screenshots/system-users.png)
This looks like the credential for the `pfsense` page.


Login to `pfsense`:
Credential => (rohit, pfsense)
![](screenshots/pfsense-login.png)


From the dashboard we can check some information:
![](screenshots/info.png)
`pfsense` version 2.1.3 is running.


#### Exploit

I searched for `pfsense version 2.1.3` and found one public exploit from exploit-db:

Exploit-code: []()https://www.exploit-db.com/exploits/43560
![](screenshots/pfsense-exploit.png)

We don't need to make some change to the exploit code and just need to execute:
![](screenshots/exploit.png)

`nc` listener on attacking side:
![](screenshots/root_shell.png)

And you can get `root.txt` :)
