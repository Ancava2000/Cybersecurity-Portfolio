# $\color{Magenta}{Pickle\ Rick\ Walkthrough\ TryHackMe}$
## $\color{Magenta}{This\ Rick\ and\ Morty-themed\ challenge\ requires\ you\ to\ exploit\ a\ web.}$
## $\color{Magenta}{server\ and\ find\ three\ ingredients\ to\ help\ Rick\ make\ his\ potion.}$
## $\color{Magenta}{and\ transform\ himself\ back\ into\ a\ human\ from\ a\ pickle.}$

![1_Jxrp0bFDLsm_VF4-FGBdww](https://github.com/user-attachments/assets/d79ca450-dc75-4ea2-a6a9-edc2991823e1)

### $\color{Cyan}{Scanning}$
Let’s start with a basic step, scanning the machine with nmap. The results show two ports:

- **Port 22/tcp,** Service SSH and Version OpenSSH.
- **Port 80/tcp,** Service http and Version Apache.

![1_rmcYv2bP3TKtF7O56YKrxQ](https://github.com/user-attachments/assets/f993b331-00ba-464f-aec6-6076d1576c72)

Focusing in the port 80 (we don’t have a username and password for the SSH service), we find the following web page:

![1_a7vUjcFJ1fFj5RHXYJi9fA](https://github.com/user-attachments/assets/518a1f78-9756-45ef-a722-1c863c60b8d5)

So the next logical step is to explore this web site!

### $\color{Cyan}{Enumeration}$
**1. Website Enumeration**

I check the source code to find any hidden messages and there it is! In a comment there’s a username. Maybe for the SSH?

![1_ub8Y6qTnGHH03-CO3ryRZQ](https://github.com/user-attachments/assets/479c353a-50d5-4622-b643-7abf22e29fcb)

I couldn’t find anything else so I proceed to do the directory enumeration with the gobuster tool (you can use other tools like dirbuster or ffuf), using the command:

`gobuster dir -u htttp://<IP>/ -w <path to wordlist> -x "php,txt"`

![1_yjpPIBJKh3Kex_ctHQEKtw](https://github.com/user-attachments/assets/8b7fe2f1-0ce6-4fc1-8739-f9655283c019)

The directories shown are:

- **assets:** all the resources used in the web site.
- **robots.txt:** shows a possible password.

![1_GBFoRTH0E1kdLL5tBg5JEg](https://github.com/user-attachments/assets/52c7b157-dae2-4214-a0f8-15179a69365d)

- **login.php:** a login page.
- **portal.php:** for now, it redirects us to the login.php.

So we have:

- A **username:** R1ckRul3s
- A possible **password:** Wubbalubbadubdub

Let’s try them in the login page… and success! We now can access the portal.php directory.

### $\color{Cyan}{Command Panel}$

Now it’s time to exploit this command panel vulnerabilities and find all the ingredients.

![1_EwFehYsbWre6a0mCP2I6Lg](https://github.com/user-attachments/assets/e4264b23-7c3f-497d-8394-dc5c40e18008)

We can use some commands, for example ls -la, which shows:

![1_XqYiJRfjEgFK39fr1CmdSg](https://github.com/user-attachments/assets/beb3596c-0484-45b1-b8e9-cb445b422f29)

**BUT** we can’t use cat. So I looked up other ways to see a text file. Some of the cat command alternatives are:

- **less** Sup3rS3cretPickl3Ingred.txt
- **tac** Sup3rS3cretPickl3Ingred.txt
- **grep .** Sup3rS3cretPickl3Ingred.txt (the “.” search for any character).

By using, for example, the command less we can see the first ingredient:

![1_aRHz8Jhi_reO2v2e8L1kiQ](https://github.com/user-attachments/assets/775edb2d-7eee-4cf8-92cc-bd36f7591a44)

The clue text file catches my attention so I read it too:

![1_jVpBTKDRYNoIXcbuTFi_vQ](https://github.com/user-attachments/assets/5f132c3b-0509-403f-bcd7-ed8d2aa19884)

Seeing that this command panel is working like a Linux terminal, it would be best to start looking in the home directory and in some of the profiles, since important information is usually stored there.

If we run “ls /home” it shows us two profiles, being the more interesting the rick one and not the default ubuntu profile.

![1__Nl5_POR8Gb5LX8Xv__7_A](https://github.com/user-attachments/assets/362cf66b-ecf8-4752-811a-7d815a25b969)

With “ls /home/rick” I find the second ingredient:

![1_2cVV4XHM5fwTdaQV_HeAWg](https://github.com/user-attachments/assets/59f002af-4c3b-43fb-bdbe-82f3bcc7fee7)

At this point, all that remains is to discover the last ingredient. As we no longer have any clues, I assume that the text will be found in the root folder, since in these challenges there’s usually a user flag and a root flag.

To enter the root folder we would need root privileges, and to find out if we have these privileges we must know our user and the type of privileges it has. If it isn’t root, we would need to elevate privileges.

With “whoami” I learn the user.

![1_wc7W2obOrk0VV4NTlm61cg](https://github.com/user-attachments/assets/9318ae46-d79d-406c-b17f-b0b242c8d554)

An with “sudo -l” I learn the commands that user can execute.

![1_NN09fHjRp_eLmkIra1BFdA](https://github.com/user-attachments/assets/25196a1c-b0e1-4b77-bb40-23230358cb1f)

And look at that! This indicates that the user can execute any type of command, including the sudo one.

Now that we know we have these privileges, we simply run “sudo ls /root” to show the files in the folder, and finally I can read the third ingredient.

![1_TCYiKlMKKRNu0IJC45mlbw](https://github.com/user-attachments/assets/a7a9557c-0662-4bed-956c-9f2a2fb25242)

![1_IBEVTXgTmrvUg05xdhXOZw](https://github.com/user-attachments/assets/9ee06b9a-30d7-4eb7-b6d6-345c7ff3222e)

**What is the first ingredient that Rick needs?**

`mr. meeseek hair`

**What is the second ingredient in Rick’s potion?**

`1 jerry tear`

**What is the last and final ingredient?**

`fleeb juice`
