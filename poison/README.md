# Poison

### Machine Info
![](screenshots/poison.png)

#### Nmap
![](screenshots/nmap.png)


##### HTTP (Port 80)

Index Page:
![](screenshots/index.png)

From the index page I can look up some script files:

`listfiles.php`:
![](screenshots/listfiles.png)

`pwdbackup.txt`:
![](screenshots/pwdbackup.png)

`/etc/passwd`:</br>
I can access to this file as this application is vulnerable to `LFI(Local File Inclusion)`. Just access to file `../../../../../../../etc/passwd`
![](screenshots/etc_passwd.png)


###### Exploit

Decrypt `pwdbackup.txt` password with `base64`:

As we have to decrypt 13 times, I just created simple bash script.
![](screenshots/decrypt_sh.png)

And this returns passwd:
![](screenshots/user_passwd.png)


From the `/etc/passwd` we already checked that there is user `charix` and password includes string `Charix`. Let's just ssh to `charix`:

![](screenshots/user_shell.png)

we are user `charix` now.



#### Privilege Escalation

There is a zip file `secret.zip` in home directory:
![](screenshots/zip.png)

Transfer to my local machine and unzip with password `Charix!2#4%6&8(0`:
![](screenshots/unzip.png)

This returns non-ascii character file. Just leave it because we don't know what it is.


Check for `netstat`:
![](screenshots/netstat.png)
There are 2 suspicious ports, `5901` and `5801`, as these ports are usually used for `vnc` service.

Check for process `ps`:
![](screenshots/ps.png)
There is a vnc process and it is running with `root`.


##### Exploit

First make dynamic port forwarding:
![](screenshots/portfwd.png)

`/etc/proxychains.conf`:
![](screenshots/proxychains.png)

And execute `vncviewer` with proxychains:</br>

At first try, I failed as it requires password for root. So I just try with password file that we got from user home directory and it worked.
![](screenshots/vncviewer.png)

`vncviewer` root shell:
![](screenshots/root_shell.png)

And you can get `root.txt` :)
