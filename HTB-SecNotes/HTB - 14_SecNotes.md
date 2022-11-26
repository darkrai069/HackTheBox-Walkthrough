# HTB/THM : Name


```toc
```

---
---
---
---

## Machine Information

| Contents      | Description                 |
| ------------- | --------------------------- |
| Name          | HTB : SecNotes              |
| Difficulty    | Medium                      |
| OS            | Windows       |
| Shell_Exploit | Enter here                  |
| Priv_Esc      | WSL and smbexec             |
| Miscellaneous | Windows Subsystem for Linux |
| IP            | 10.10.10.97                            |


---
---
---


## Scanning
### Nmap

![[Pasted image 20221103090807.png]]
- Here in the nmap scan we can see that 2 ports are open.
- port 80 and port 445.
- SMB iis an attack verctor thats for sure but maybe not the initial loop hole. 
- Lets enumerate both *HTTP-80* and *SMB-445*

![[Pasted image 20221103102527.png]]
- OS discovery shows a possibility of **Wiindows 10 Enterprise 6.3**


---
---

## Enumeration
### Web Browser 

#### a. robots.txt

#### b. source code

#### c. basic website enumerations/Spidering

> **WebPage**
> > ![[Pasted image 20221103091543.png]]
> > - Well this is a login Page so lets try
> > > 1. SQLi
> > > 2. Default passwords
> > > 3. Brute Forcing
> > 
> > ![[Pasted image 20221103091807.png]]
> > - Theree is a Sign Up page as well, you wouldnt know where the vul'n would be so lets enumerate both the pages.
> > - sign up for a random account and you will be taken to the next page. Sometimes webpages wont take us anywhere even if we login, so this might be a problem.
> > 
> > ![[Pasted image 20221103122124.png]]
> > - You will be taken to this page when you Sign Up and then login with that credentials.
> > - See the thinking should be in this way. If I login with my credentials and its taken me to this page, what if I logiin with other credentials. And thats where **SQLi** comes into play. ==Lets try SQLi.==.
> > - you can also see the name of a user **tyler** and the domain **secnotes.htb**
> 
> > ![[Pasted image 20221103124226.png]]
> > - This is after you exploit the SQLi in the /signup.php page.
> > - you will get a password and another SBM page. lets try that one out.
> 

#### d. searchsploit/online Databases

#### e. other enumerations

> **SMB**
> > ![[Pasted image 20221103091200.png]]
> > - There is no anonymous login enabled in SMB and we can login without any password.
> > - THe idea would be to find a way to exploit SMB through which we can gain a foothold or then go to HTTP.
> 
> > ![[Pasted image 20221103133142.png]]
> > - psexec wont work, it might be because of some anti-virus in place.
> > - Keep thiis information in you mind when you start exoliting the machine for a shell.
> 
> > ![[Pasted image 20221103125923.png]]
> > - See we are able to login to smb usiing the credentials that we gfet from exploiting the HTTP sqli vul'n.
> > - Now its ablot leveraging this attack vector to get a user shell in the Vctim mahcine.
> > - Here we can do a thing, just like we have done in the **HTB - Devel** machine, we can create a reverse shell and upload it to the macine and run it. Cuz whatever we upload here in the smb directory is gonna be available in the main IIS page.
> > - But there is a chtch, we cant just upload the exe/aspx we have to use a php script that runs the payload otherwise the anti virus is gonna ditctect it.
> > - Now its all about Exploiting!!!


---
---

## Exploit

### a. Attack Vector

- Payload Upload and executing 
- We have to design a simple php code to run this payload as well.
- What we are doing here is that, since we cant run any Payloads since the machine blocks it, we are trying to run netcat on the victim machiine and get a reverse shell via netcat.
- So we need to first upload a netcat executable to the victim machine and then run that executable using the IP and PORT of out Attacker machine, thus getting a shell.


### b. Mode of Attack

> nc.exe + php basic reverse shell
> > ![[Pasted image 20221103143728.png]]
> > - we first create the PHP basiic reverse shell and the nc.exe.
> > - then add these to the smb share.
> > - and fiinally go to the IIIS site and run the php_basic_revese-shell **run_shell.php**
> 
> > ![[Pasted image 20221103143816.png]]
> > - And thats it we get the Basic shell to the Victim machine.


---
---

## Priv_Esc

### a. Attack Vector
> **Basic Enumerations.**
> > ![[Pasted image 20221103143912.png]]
> > - No token attacks possible
> 
> > ![[Pasted image 20221103143945.png]]
> > - we are up against some kinda defender as we have deductd before.
> 
> > ![[Pasted image 20221103144012.png]]
> > - locating the **wsl.exe** and **bash.exe** executable. as part of the Basic Enumeration
> 
> > ![[Pasted image 20221103144053.png]]
> > - We found the files **wsl.exe** and **bash.exe**. Here in this perticular machine lets use the **bash.exe** and get a root linux shell thats inside the windows shell.


### b. Mode of Attack

> **Exploitatiioin**
> > ![[Pasted image 20221103143554.png]]
> > - THats it .... the enumeration for the priv_Esc was easy, we straight away get the Admin password.
> > - Lets use either of the following impackets modules or the basic *smbcliient* 
> > > 1. **psexec** or 
> > > 2. **smbexec** or 
> > > 3. **wmiexec** or 
> > > 4. **smbclient**
> 
> > ![[Pasted image 20221103144337.png]]
> > - We are *NT AUTHORITY\SYSTEM*
> 
> > ![[Pasted image 20221103144402.png]]
> > - Machine Pwned!!!