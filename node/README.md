# Node

### Machine Info
![](screenshots/node.png)

#### Nmap
![](screenshots/nmap.png)

##### HTTP (Port 3000)

Index Page:
![](screenshots/index.png)

Login Page:
![](screenshots/login.png)

First I checked the source code and found some credentials from `js` sources.

`/api/users`:
![](screenshots/api_users.png)

`/api/users/latest`:
![](screenshots/api_latest.png)

I decoded hashes from online decoder, `crackstation`, and got password for admin user `myp14ceadm1nacc0unt`. I tried to login with credential and I was able to:

Password for admin user is `manchester`:
![](screenshots/admin_hash.png)

Successfully logged in:
![](screenshots/admin_login.png)

From the admin page, I could download backup file. The downloaded file is `base64` encoded file and it is huge. Let's decode and check again:
![](screenshots/backup_decode.png)

![](screenshots/zip.png)

The decoded file is `zip` archive data and I just tried to unzip the file to check data:
![](screenshots/zip2.png)

The file requires password to unzip. I wasn't able to get some password during enumeration and admin password that I used in login step was useless. Only option left is brute-forcing:

![](screenshots/zip_brute.png)

I could get password by brute-forcing and unzipped successfully.


The backup files look like just back up for whole directory of `/var/www/html`. There are many files and I got some interesting file which includes the password and username for `mongodb` database:

`app.js`:
![](screenshots/app_js.png)

And I just tried `ssh` to user mark with that credential and it worked:
![](screenshots/low_shell.png)

#### User Mark to User Tom

During enumeration I found one binary file `/usr/local/bin/backup` which looks like very interesting but I couldn't do change or read due to lack of permission:
![](screenshots/back_up.png)
I think this file can be used for root privilege escalation.


I found that there are 2 processes of `Tom`, `/var/scheduler/app.js` and `/var/www/myplace/app.js`.
![](screenshots/ps_tom.png)

First I checked the `/var/www/myplace/app.js` and it was just same as we got from the backup directory. Then I checked the other one and it was about the scheduling job in mongodb:
![](screenshots/scheduler.png)

This seems like we can add some job and the job will be executed.

##### Exploit

The user `tom` is in the group `admin` and we can add job and executes with user `tom` privilege.


###### First try:</br>
Let's just copy `/bin/bash` to `/tmp` with changing permission:
![](screenshots/first_add.png)

I was able to get a shell but I wasn't able to execute or run binary `backup` because I wasn't in the group `admin` even I was user `tom`.


###### Second Try:</br>
I successfully got the shell and I am user `tom` and in the group `admin`.
![](screenshots/second_add.png)
![](screenshots/last_add.png)



#### User To Root Privilege Escalation

From the `app.js` that I got from backup directory of admin page, I can check the usage of `/usr/local/bin/backup`:
![](screenshots/backup_usage.png)

It requires arguments for `backup_key` and `dirname`. The `backup_key` is constant and it is `45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d474`. We just need to change of directory name that we want to back-up.

##### Exploit

###### First Try

```
/usr/local/bin/backup -q 45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d474 /root
```

Decode and unzip like we did before and I got:
![](screenshots/fail.png)

I think if we do something wrong, we will get that face :(


###### Second Try

I just copied the `backup` binary to my machine and debugged:
![](screenshots/backup_badchar.png)

As we can see it checks for string `/root` from `encoded` backup hash. There are more bad characters, `..`, `.`, `/`, `;`, `&`, and if it includes these characters it will not encoded correctly and we will get that face.

Command:
```
/usr/local/bin/backup -q 45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d474 root

```

Decode and unzip:
![](screenshots/root_dir.png)

WE CAN GET `root.txt` but no shell.


###### Third Try

I can get `root.txt` but I want to get a root shell. I debugged again to get a shell:

![](screenshots/root_shell_debug.png)

The command `/usr/bin/zip` takes 3 arguments and it will be executed via `system()` function as we can see. The `system()` allows a new line so maybe we can do command injection to catch a shell:

I added 'b' after `/bin/bash` because if we do not add, the `/dev/null` will be on the `/bin/bash`.
![](screenshots/backup_root_shell.png)

And you can get `root.txt` :)
