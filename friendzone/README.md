# Friendzone

### Machine Info
![](screenshots/friendzone.png)

#### Nmap
![](screenshots/nmap.png)


##### FTP (Port 21)

I tried anonymous login but it is not allowed:
![](screenshots/ftp_anonymous.png)

##### SMB (Port 139,445)

SMB OS discovery:
![](screenshots/smb_os.png)

SMB Vulnerability:
![](screenshots/smb_vuln.png)


`smbclient`:
![](screenshots/smbclient.png)
Interesting point is `Files` share comment shows it's `/etc/Files`. `general` and `Development` could be `/etc/genral` and `/etc/Development`.

`smbmap`:
![](screenshots/smbmap_recur.png)


From the `general` share, I got the credential for admin:
![](screenshots/creds.png)


##### HTTP (Port 80)

Index Page:
![](screenshots/index.png)


###### Gobuster
![](screenshots/gobuster.png)


`/robots.txt`:
![](screenshots/http_robots.png)


`/wordpress`: Nothing from this page.


##### HTTPS (Port 443)

Index Page:
![](screenshots/https_index.png)

I tried to access `/index.html` and `/index.php` but also not found. So I just checked certificate:

![](screenshots/https_cert.png)

It seems like there are two different domain names `friendzone.red` and `friendzoneportal.red` (from http page). Just add domains to `/etc/hosts` file and try again:

`https://friendzoneportal_red`:
![](screenshots/friendzoneportal_red.png)

`https://friendzone.red`:
![](screenshots/friendzone_red.png)

Source code:
![](screenshots/friendzone_red_source.png)


`https://friendzone.red/js/js`:
![](screenshots/friendzone_red_jsjs.png)

Source Code:
```
<p>Testing some functions !</p><p>I'am trying not to break things !</p>VXRHMWR2UkIxTzE1ODI2ODEwMDNNSHZhWFV1ZmhZ
<!-- dont stare too much , you will be smashed ! , it's all about times and zones ! -->
```
This could be something or just rabbit hole.

###### Gobuster
![](screenshots/friendzone_red_gobuster.png)

I checked the `/admin` directory but it's empty.



##### DNS (Port 53)

###### Zone transfer

`friendzone.red`:
![](screenshots/friendzone_red_dig.png)

`friendzoneportal.red`:
![](screenshots/friendzoneportal_red_dig.png)

From both results, just grep only domains and add to `/etc/hosts`:

```
❯ dig axfr @10.10.10.123 friendzoneportal.red > zone
❯ dig axfr @10.10.10.123 friendzone.red >> zone
❯ cat zone | grep friendzone | grep IN | awk '{print $1}' | sed 's/\.$//g' | sort -u | tr '\n' ' ' > zonetransfer
```


`https://admin.friendzoneportal.red/`:
![](screenshots/admin_friendzoneportal_red.png)

Tried to login with `[admin, admin]`:
![](screenshots/admin_not_yet.png)

</br></br>

`https://uploads.friendzone.red/`:
![](screenshots/ports_friendzoneportal_red.png)

Just uploads `txt` files:
![](screenshots/upload_success.png)

It seems like file is uploaded successfully but I couldn't find where it stores.

</br></br>



`https://administrator1.friendzone.red/`:
![](screenshots/administrator1_friendzone_red.png)

Tried to login with `[admin, admin]`:
![](screenshots/wrong.png)

And with `[admin, WORKWORKHhallelujah@#]` which we found from SMB:
![](screenshots/login_success.png)


`https://administrator1.friendzone.red/dashboard.php`:
![](screenshots/dashboard.png)


Tried with default parameters:
`https://administrator1.friendzone.red/dashboard.php?image_id=a.jpg&pagename=timestamp`:
![](screenshots/default_param.png)

Tried with different parameters (with different image_id and timestamp value) but nothing worked.

###### Gobuster
![](screenshots/administrator_gobuster.png)


I checked `/images` directory and `image_id` comes from this directory. Also even `gobuster` didn't return about `timestamp.php`, I just tried to access and there is one:

![](screenshots/timestamp.png)

So `dashboard.php` timestamp value is from `timestamp.php`. I think this could be somewhat related to `LFI` Vulnerability.

Change parameter `pagename=timestamp` to `pagename=login`:
![](screenshots/param_login.png)
I got different result.


I tried with `pagename=/etc/passwd` but this didn't work and this is because maybe it appends `.php` to the end of the parameter. Therefore the file should be `.php` extension.

I also tried with `pagename=../uploads/upload`:
![](screenshots/param_uploads.png)


###### Checking Source Code

From google search I found that it is possible to read source code with `php filter`.

`pagenmae=php://filter/convert.base64-encode/resource=login` and this returns bas64 encoded strings:

`PD9waHAKCgokdXNlcm5hbWUgPSAkX1BPU1RbInVzZXJuYW1lIl07CiRwYXNzd29yZCA9ICRfUE9TVFsicGFzc3dvcmQiXTsKCi8vZWNobyAkdXNlcm5hbWUgPT09ICJhZG1pbiI7Ci8vZWNobyBzdHJjbXAoJHVzZXJuYW1lLCJhZG1pbiIpOwoKaWYgKCR1c2VybmFtZT09PSJhZG1pbiIgYW5kICRwYXNzd29yZD09PSJXT1JLV09SS0hoYWxsZWx1amFoQCMiKXsKCnNldGNvb2tpZSgiRnJpZW5kWm9uZUF1dGgiLCAiZTc3NDlkMGY0YjRkYTVkMDNlNmU5MTk2ZmQxZDE4ZjEiLCB0aW1lKCkgKyAoODY0MDAgKiAzMCkpOyAvLyA4NjQwMCA9IDEgZGF5CgplY2hvICJMb2dpbiBEb25lICEgdmlzaXQgL2Rhc2hib2FyZC5waHAiOwp9ZWxzZXsKZWNobyAiV3JvbmcgISI7Cn0KCgoKPz4K`


From kail:

```
❯ echo -n PD9waHAKCgokdXNlcm5hbWUgPSAkX1BPU1RbInVzZXJuYW1lIl07CiRwYXNzd29yZCA9ICRfUE9TVFsicGFzc3dvcmQiXTsKCi8vZWNobyAkdXNlcm5hbWUgPT09ICJhZG1pbiI7Ci8vZWNobyBzdHJjbXAoJHVzZXJuYW1lLCJhZG1pbiIpOwoKaWYgKCR1c2VybmFtZT09PSJhZG1pbiIgYW5kICRwYXNzd29yZD09PSJXT1JLV09SS0hoYWxsZWx1amFoQCMiKXsKCnNldGNvb2tpZSgiRnJpZW5kWm9uZUF1dGgiLCAiZTc3NDlkMGY0YjRkYTVkMDNlNmU5MTk2ZmQxZDE4ZjEiLCB0aW1lKCkgKyAoODY0MDAgKiAzMCkpOyAvLyA4NjQwMCA9IDEgZGF5CgplY2hvICJMb2dpbiBEb25lICEgdmlzaXQgL2Rhc2hib2FyZC5waHAiOwp9ZWxzZXsKZWNobyAiV3JvbmcgISI7Cn0KCgoKPz4K | base64 -d > login.php
```

```
# login.php
❯ cat login.php
<?php


$username = $_POST["username"];
$password = $_POST["password"];

//echo $username === "admin";
//echo strcmp($username,"admin");

if ($username==="admin" and $password==="WORKWORKHhallelujah@#"){

setcookie("FriendZoneAuth", "e7749d0f4b4da5d03e6e9196fd1d18f1", time() + (86400 * 30)); // 86400 = 1 day

echo "Login Done ! visit /dashboard.php";
}else{
echo "Wrong !";
}

?>
```

By using same method, we can check that `upload.php` just returns timestamp but not uploading file:

```
❯ cat upload.php
<?php

// not finished yet -- friendzone admin !

if(isset($_POST["image"])){

echo "Uploaded successfully !<br>";
echo time()+3600;
}else{

echo "WHAT ARE YOU TRYING TO DO HOOOOOOMAN !";

}

?>
```

`dashboard.php`:
```
<?php

//echo "<center><h2>Smart photo script for friendzone corp !</h2></center>";
//echo "<center><h3>* Note : we are dealing with a beginner php developer and the application is not tested yet !</h3></center>";
echo "<title>FriendZone Admin !</title>";
$auth = $_COOKIE["FriendZoneAuth"];

if ($auth === "e7749d0f4b4da5d03e6e9196fd1d18f1"){
 echo "<br><br><br>";

echo "<center><h2>Smart photo script for friendzone corp !</h2></center>";
echo "<center><h3>* Note : we are dealing with a beginner php developer and the application is not tested yet !</h3></center>";

if(!isset($_GET["image_id"])){
  echo "<br><br>";
  echo "<center><p>image_name param is missed !</p></center>";
  echo "<center><p>please enter it to show the image</p></center>";
  echo "<center><p>default is image_id=a.jpg&pagename=timestamp</p></center>";
 }else{
 $image = $_GET["image_id"];
 echo "<center><img src='images/$image'></center>";

 echo "<center><h1>Something went worng ! , the script include wrong param !</h1></center>";
 include($_GET["pagename"].".php");
 //echo $_GET["pagename"];
 }
}else{
echo "<center><p>You can't see the content ! , please login !</center></p>";
}
?>
```

#### Exploit

The web application is vulnerable to `LFI` and if we can upload our `.php` malicious file we can get a shell.

The only method that we can upload is using `SMB Development share` as that is the only share we can write and there is no point that we can upload.

First just copy Kali php version of webshell from `/usr/share/webshells/php/php-reverse-shell.php` and change `ip` and `port`. Then upload to the smb share `Development`:

![](screenshots/upload_shell.png)

Change parameter `pagename=pagename=../../../../../etc/Development/shell` and `nc` from listener:

![](screenshots/low_shell.png)



#### Privilege Escalation

I checked the `/var/www/` directory and found credential about user `friend`:
![](screenshots/user_cred.png)

By using that credential, I can `ssh` to user `friend`:
![](screenshots/user_shell.png)


After some enumeration, I couldn't find something interesting so I just upload and run [pspy](https://github.com/DominicBreuker/pspy) which is for detecting cronjob and found one cronjob:
![](screenshots/cronjob.png)

The `/opt/server_admin/reporter.py` is running every 2 minutes with root privilege:
![](screenshots/reporter.png)
```
#!/usr/bin/python

import os

to_address = "admin1@friendzone.com"
from_address = "admin2@friendzone.com"

print "[+] Trying to send email to %s"%to_address

#command = ''' mailsend -to admin2@friendzone.com -from admin1@friendzone.com -ssl -port 465 -auth -smtp smtp.gmail.co-sub scheduled results email +cc +bc -v -user you -pass "PAPAP"'''                                                                

#os.system(command)

# I need to edit the script later
# Sam ~ python developer
```

But as we can see, we cannot fix python script due to lack of permission which means we need to find another way. During the enumeration I found that `/usr/lib/python2.7/os.py`, which is python library, is writable by anyone.
![](screenshots/writable.png)

The `reporter.py` script is importing `os` library so we can spawn a shell by just adding shell script to end of the `os.py`:
![](screenshots/python_shell.png)

`nc` listener on attacking side:
![](screenshots/root_shell.png)

And you can get `root.txt` :)
