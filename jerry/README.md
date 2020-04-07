# Jerry

### Machine Info
![](screenshots/jerry.png)

#### Nmap
![](screenshots/nmap.png)


##### HTTP (Port 8080)

Index Page:</br>
![](screenshots/index.png)
From the index page we can check the the version of `Apache Tomcat 7.0.88` and there are 3 links `Server Status`, `Manager App` and `Host Manager` which is for managing tomcat server.

`Server Status`:

This page requires the authentication and credential is `[admin, admin]`. From this page we can check the OS, Architecture, Server Status and Installed applications.
![](screenshots/server_status.png)


`Manager App`:

This page also requires the authentication and I tried with credential [admin, admin] but failed and get error message. In the error message, it shows how to config for manager user with an example credential, ``[tomcat, s3cret]``. I tried with that credential to login and was successful.

![](screenshots/admin_denied.png)

![](screenshots/manager.png)

From this page, we can deploy file with `war` extension:
![](screenshots/manager_deploy.png)


`Host Manager`:

Tried to login this page but failed. I couldn't find any credential for this page.



#### Exploit

The attacker can deploy some malicious file from the `web application manager` and it is vulnerable to `RCE(Remote Code Execution)`.

![](screenshots/manager_deploy2.png)

First create malicious shell file with `war` extension:
![](screenshots/msfvenom.png)

And deploy `shell.war` and you can see new path `/shell`:
![](screenshots/shell_dep.png)

Now just access to `/shell` page and `nc` listener on attacking side:
![](screenshots/root_shell.png)

And you can get `root.txt` :)
