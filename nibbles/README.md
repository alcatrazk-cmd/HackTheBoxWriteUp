# Nibbles

### Mahchine Info
![](screenshots/nibbles.png)




#### Nmap
![](screenshots/nmap.png)


##### HTTP (Port 80)

Index page:
![](screenshots/index.png)

Index page source code:
![](screenshots/index-source.png)

Access to `10.10.10.75/nibbleblog/`:
![](screenshots/nibbleblog.png)

###### Gobuster
`Gobuster` to find out hidden page/directory:
![](screenshots/gobuster.png)

`nibbleblog/admin.php`:
![](screenshots/nibbleblog_admin.png)
I tried common password for admin and root but not worked.

`nibbleblog/README`:</br>
From this page, I can check the version of `nibbleblog`.
![](screenshots/nibbleblog_ver.png)

Check for public exploit:
![](screenshots/nibbleblog_vuln.png)

There is a one meatsploit version exploit for `Nibbleblog 4.0.3` and it is vulnerable to `Arbitrary File Upload`. I checked the exploit but it requires authenticated user. We have to find the `username` and `password`.

I looked up every pages I found from `gobuster` but I couldn't find credentials. Maybe we need to brute-force or just guessing:

![](screenshots/nibbleblog_brute.png)
After a few try manually I got this error message which means we cannot use tools such as `hydra` for brute-forcing.

I tried many credentials and finally found one => `[admin, nibbles]`:
![](screenshots/nibbleblog_login.png)

#### Exploit
Now we have credential and we can try the exploit what we found from `exploit-db`. Instead of using `metasploit`, I just followed the step based on exploit as it looks like very simple

First just check if `my_imag` plugin is installed:</br>
`http://10.10.10.75/nibbleblog/admin.php?controller=plugins&action=config&plugin=my_image`

![](screenshots/nibbleblog_myimage.png)

Create php reverse shell or you can just copy from share directory:
```
❯ cp /usr/share/webshells/php/php-reverse-shell.php ./
```
And just change `ip` and `port` from script.


Now we need to just upload and execute:
![](screenshots/nibbleblog_upload.png)

Access to `image.php`
![](screenshots/nibbleblog_exploit.png)


`nc` listener on attacking side:
![](screenshots/user_shell.png)



#### Privilege Escalation

Spawn tty and check for `sudo` privilge:
![](screenshots/tty.png)
![](screenshots/user_sudo.png)

In the user `nibbler` home directory, there is a zip file `personal.zip`. Let's just unzip and enumerate:
![](screenshots/user_personal.png)

So user can execute `/home/nibbler/personal/stuff/monitor.sh` with root privilege.

##### Exploit



Create and execute malicious `monitor.sh`:
![](screenshots/sudo_shell.png)

![](screenshots/sudo_shell2.png)

And you can get `root.txt` :)
