# Proving Grounds Play: DriftingBlues6 Walkthrough
## _Learn step by step how to find the Offsec DriftingBlues6 lab flag._

### **Access to the machine**

First you must download the Offsec vpn, which is downloaded under the name universal.ovpn. Once downloaded, type _“sudo openvpn universal.ovpn”_ to access the vpn and so be able to connect to the machine we are going to hack (any errors and questions with this process can be resolved on the Offsec page itself).
![image 1](https://miro.medium.com/v2/resize:fit:1100/format:webp/0*HUZI6RL9GDIbxDc1)

You can check that you are connected when the VPN logo is green.
![image 2](https://miro.medium.com/v2/resize:fit:720/format:webp/0*xeQCPLy6oO_8n3CI)

### **IP Enumeration**

In another terminal (it is important to leave open the one that maintains the vpn connection) the basic process of any pentesting is started with its first step: **enumeration**. For this purpose **nmap** is used, which scans the services and open ports of a device.
![image 3](https://miro.medium.com/v2/resize:fit:720/format:webp/0*masb_OkNB1pC0MrH)

This is the first step because it shows you the services, and even their versions, associated to an ip, so you can see if there are any vulnerable service and, later, the exploits that can be performed on these vulnerabilities.

As can be seen in the screenshot, port **80** appears as open, which indicates an application on the Internet **(HTTP)**. Therefore, we are going to search on the internet for the ip itself, showing this web page.
![image 4](https://miro.medium.com/v2/resize:fit:720/format:webp/0*pudqVmxzY6jp1j9N)

### **Website Enumeration**
Once the ip address is scanned, the second step would be to scan and enumerate the web page itself. In a web page you can find directories and files with vulnerabilities such as configuration errors, causing these supposedly hidden files and directories to be shown.

In order to find all the directories and files that a website has, you can use tools such as **dirbuster, dirsearch, gobuster...**

In this case we’ve used **dirsearch and gobuster**, which can apply multiple wordlists: lists containing the most used words to name directories and files in web pages. These, automatically and using brute force, add to the indicated web these directories and show those that exist with the code **200 OK.**

With **dirsearch** you simply use _“-u <ip>”_.

![image 5](https://miro.medium.com/v2/resize:fit:720/format:webp/0*LgU0rCRNw-TCVk_5)
![image 6](https://miro.medium.com/v2/resize:fit:720/format:webp/0*gPw5rodccUp2k6Jc)

As you can see, it first indicates where the search output is going to be saved, and then you can see the directory attempts with all the words in the list. In this case **db, robots.txt and textpattern** appear. **db** is where the picture of the website is located, **robots** indicates what the crawlers can and cannot see and textpattern indicates the existence of a website using **Textpattern CMS** (content management system).

The **robots.txt** file specifically gives us two clues:

- Another directory that is hidden from crawlers called **/textpattern/textpatern/.**
- The existence of a .zip file.

![image 7](https://miro.medium.com/v2/resize:fit:640/format:webp/0*c5hRnEkM-ZVBQuA0)

If you search the directory **/textpattern/textpattern**, a login page is shown.
![image 8](https://miro.medium.com/v2/resize:fit:720/format:webp/0*Oe56p1UHlLGH5TH8)

To see if there might be any other directories on the web page, another tool, **gobuster**, is used.

You must indicate the type of attack with **“dir ”**, the target with **“-u <ip>”** and the wordlist to use with **“-w <wordlist>”**. In this case the wordlist is the **directory-list-2.3-medium**, as it is one of the most complete. With it, we find a new directory called spammer.
![image 9](https://miro.medium.com/v2/resize:fit:720/format:webp/0*TEVXU9qx4qytJVei)

If you search the **spammer** directory, a .zip file is downloaded directly.

### **Cracking .zip passwords**

In this .zip file there’s a text file called creds which is encrypted (a password is needed to extract it).

In order to decrypt the password of a .zip archive there is a tool called **zip2john**. This tool processes zip files in a format understandable by **johntheripper**, so it can then use brute force to discover the password.

Simply indicate the zip file and the file where the output is saved, which is in a format understood by **johntheripper.**
![image 10](https://miro.medium.com/v2/resize:fit:720/format:webp/0*3DV96ib6Aw2_eEfU)

Once done, with **johntheripper** you can crack the zip password by passing that output, resulting in myspace4.

It should be noted that since I had cracked it previously, I get the following message: **No password hashes left to crack.** This indicates that the password has already been discovered, and that to see it there are two options:

- Go to the address **/etc/john/john/john.pot**, where already discovered passwords are stored.
- Type john _“ — show spammer.txt ”_, where the password appears.

![image 11](https://miro.medium.com/v2/resize:fit:720/format:webp/0*njkANFDbQKKWfd2p)

With this password you can now extract the text file, which contains the credentials for the login page seen above.

### **Web Site Vulnerabilities**

Once inside the **textpattern** page (which is like a wordpress), you must find its vulnerabilities. In this case, the application used and its version are indicated at the bottom of the page.

When you search for the application along with its version you can see what exploits are related to it. **ExploitDB**, is a database that shows all the vulnerabilities associated with technologies and their versions, along with exploits to take advantage of these.
![image 12](https://miro.medium.com/v2/resize:fit:720/format:webp/0*d2wphDefuklhEgOB)

Another way to search for exploits is in the terminal with searchsploit. You must indicate the technology and, if possible for more specific results, its version.

If we do this with our case, we get the same exploit shown above, since **searchsploit** uses the same database as **exploitDB.**
![image 13](https://miro.medium.com/v2/resize:fit:720/format:webp/0*PahdT9vqdBJ5mTvX)

As a tip, if you know the code used to write the exploit (either in C, python…), it is good to read it to understand what the exploit is doing.

Another tip is that you can directly download the exploit by typing _“searchsploit -m <exploit_code>”._

### **Reverse Shell PHP**

In this first exploit, **Remote code execution (Authenticated) (2)**, with id **49620**, it is explained that an authenticated user can execute code remotely after uploading a .php file to the textpattern page. As this exploit only consists in the fact that a php file can be uploaded, we can upload any file we want from the textpattern page itself.

Therefore, you can upload a **reverse shell in php**. Basically a reverse shell is a technique where the victim’s machine starts a connection with the attacker’s machine, allowing remote access to the victim’s system through a shell.

In this case, a php reverse shell would be uploaded, the port where the connection would be initialized would be listened with netcat and, once the victim interacts with that php file and starts the connection, the attacked system could be controlled from the shell.

To get this reverse shell in php, visit the page **pentestmonkey** (page with resources for pentesters).
![image 14](https://miro.medium.com/v2/resize:fit:720/format:webp/0*SKNaVKyjMH7ZcWzn)

Once you have downloaded the exploit you must modify the php with **gedit**, a program that allows you to graphically view the code. Specifically, change line 49 _“ip”_ with your local ip, while the port can be left as it is.
![image 15](https://miro.medium.com/v2/resize:fit:720/format:webp/0*edH24Str_CfjAgwR)

Once this is configured, we must listen to the port being used (1234 in this case), so when the victim interact with the file we can enter the terminal. We do this with **netcat**, typing _“nc -nlvp <port_to_listen>”_:

- **n** = ip numbers only.
- **l** = listen mode for incoming connection requests (on the specified port).
- **v** = detailed output.
- **p** = port to listen.

![image 16](https://miro.medium.com/v2/resize:fit:720/format:webp/0*rD57hAyciZ0uLx44)

The next step is to upload the reverse Shell php file to the page. In the content folder, in files, you can easily upload it. After that, you must access the files folder where the files are saved, so that netcat can listen to it. This can be done by searching the link _“textpattern/files/<name_of_the_uploaded_file>”_.
![image 17](https://miro.medium.com/v2/resize:fit:720/format:webp/0*Ll7xjrlST2ZzaaS8)

In the Shell you can see that you have connected to the target machine, you can also check by typing _“hostname”_, which indicates the name of the machine, driftingblues.

![image 18](https://miro.medium.com/v2/resize:fit:720/format:webp/0*XrNmyRcl-OvW5mTv)

Inside this shell you can find the flag. We start searching in the home directory to see which users there are, but finding none, it is deduced that the flag it’s in the root user. So now we must elevate privileges to be able to enter the root directory.

### **Shell Enumeration and Privilege Escalation**
In this step we make our **third enumeration, of the target machine itself.**

First you must know its version in order to find the appropriate exploit. This is done with _“uname -a”._

By searching for the whole version plus the word exploit, **linux 3.2.0 debian 3.2.78 exploit**, the **dirty Cow exploit** appears as the first result. As a note, you can also search with **searchsploit.**

This exploit written in C generates a new root user, overwriting the original root. You can change the new user name in the file (being the generic username firefart) and the new password can be typed once it is compiled. To compile it you must write: _“gcc -pthread 40839.c -o dirty -lcrypt”_, to run it: _“./dirty”_ or _“./dirty my-new-password”_, and once everything is changed you enter the new root user with _“su firefart”_ (if you have not changed the generic username).
![image 19](https://miro.medium.com/v2/resize:fit:720/format:webp/0*mGgtBUOjZzufOgdO)

Since the privilege escalation is to be done on the target machine, the exploit must be executed from its shell. To do this, the .c file can be uploaded to the textpattern page, so that it can be compiled and executed from the shell itself.

To compile it and execute it, you must go to the directory **/var/www/textpattern/file**, where the uploaded files are stored.
![image 20](https://miro.medium.com/v2/resize:fit:720/format:webp/0*9S7eK4fbO9VmKb05)

Once compiled as indicated, it’s necessary to know that the executable doesn’t have the necessary permissions, to change that you should type _“chmod 777 <executable_name>”_.

![image 21](https://miro.medium.com/v2/resize:fit:720/format:webp/0*XgxBIn1O-ZmvEYX1)

Once it’s executed, it shows that the password and the new username have been changed. However, when we type _“su firefart”_ as indicated in the exploit, an error appears: _“su”_ must be run in a terminal, not a shell.
![image 22](https://miro.medium.com/v2/resize:fit:720/format:webp/0*AMIv_Bs5PKA2Fzkv)

One of the options to fix this is to make an **interactive shell**, which behaves like a terminal.

One of the most commonly used ways is the _python -c ‘import pty; pty.spawn(“/bin/bash”)’_ command, which imports an interactive shell, generating a pseudo-terminal that tricks commands like _“su”_ into thinking they are being executed in a proper terminal.

With this, we can now enter _“su firefart”_ and, with the new password, we are root.
![image 23](https://miro.medium.com/v2/resize:fit:720/format:webp/0*qfEmj0GGNZMuc6vs)

Finally, we access root and here at last is the flag.
