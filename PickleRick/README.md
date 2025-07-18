
# Pickle Rick TryHackMe CTF – Write Up

Room link: https://tryhackme.com/room/picklerick

This is a simple web application that lets you simulate exciting anime fight scenes by triggering attack animations and effects with button clicks or keyboard shortcuts. Choose your fight, unleash powerful moves, and watch your enemy's health bar drop!



## Task 1: What is the first ingredient that Rick needs?

My initial step in approaching this challenge was to perform network discovery on the target machine. I used nmap to scan the IP address **10.10.105.232** for open ports and services:

![Alt text](/assets/0.png)

The nmap scan revealed two open ports:
*	**22/tcp**: Running OpenSSH 8.2p1 (Ubuntu Linux)
*	**80/tcp**: Running Apache httpd 2.4.41 (Ubuntu)
The presence of an HTTP service on port 80 indicated a web server. To facilitate access, I added the target IP and a custom hostname (**picklerick.thm**) to my **/etc/hosts** file:

![Alt text](/assets/1.png)

Navigating to **http://picklerick.thm/** in a web browser displayed a static page titled "Help Morty!" with a message from Rick. There were no immediate interactive elements.

![Alt text](/assets/2.png)

To uncover more information, I inspected the page's source code **(view-source:http://picklerick.thm/)**. This revealed a hidden comment:

![Alt text](/assets/3.png)

This username, **R1ckRul3s**, was a valuable piece of information, but its intended use was unclear at this stage. My next logical step was to perform a directory scan to uncover hidden files or directories on the web server. I used **gobuster**.

![Alt text](/assets/4.png)

The gobuster scan identified several files and directories. The **/login.php** page presented a simple login form. I attempted to use the discovered username R1ckRul3s and brute force the password but failed.

![Alt text](/assets/5.png)

I then examined the /robots.txt file, which contained the string **"Wubbalubbadubdub"**. Recognizing this as Rick's iconic catchphrase, I tried it as the password for the **/login.php** page, successfully gaining access. 

![Alt text](/assets/6.png)
![Alt text](/assets/7.png)

Upon successful login, I was redirected to **/portal.php**, which featured a "Command Panel" with an input box. This suggested a command injection vulnerability. I tested it with a basic Linux command, **ls**, to list files and directories in the current working directory 

![Alt text](/assets/8.png)

The output of ls revealed a file named **Sup3rS3cretPickl3Ingred.txt** that was not found during the initial gobuster scan. Navigating directly to **http://picklerick.thm/Sup3rS3cretPickl3Ingred.txt** in my browser revealed the first ingredient. 

![Alt text](/assets/9.png)
## Task 2: What is the second ingredient in Rick's potion?

Returning to the command panel, I continued exploring the file system. I used **ls /home** to list the contents of the **/home** directory, which revealed a user directory named **rick**

![Alt text](/assets/10.png)

To investigate further, I ran **ls -la /home/rick** to list all files and their permissions within Rick's home directory. This command revealed a file named second ingredients.

![Alt text](/assets/11.png)

Attempting to read the contents of second ingredients using **cat /home/rick/"second ingredients"** resulted in an error message: "Command disabled to make it hard for future PICKLEEEE RICCCKKKK." This indicated that direct cat commands were blocked.

![Alt text](/assets/12.png)

To bypass this restriction, I decided to establish a reverse shell. I set up a Netcat listener on my attacking machine:

![Alt text](/assets/13.png)

Next, I used the command panel to execute a BusyBox reverse shell, connecting back to my listener. (Note: The IP address changed due to a machine restart.)

![Alt text](/assets/14.png)
![Alt text](/assets/15.png)

Upon receiving the connection, I stabilized the reverse shell for better interactivity. From within the stabilized shell, I navigated to **/home/rick** and successfully read the second ingredients file using cat, revealing the second ingredient.

![Alt text](/assets/16.png)
![Alt text](/assets/17.png)

## Task 3: What is the last and final ingredient?

With a reverse shell established, the next objective was to escalate privileges and find the final ingredient. I began by checking the current user's sudo privileges using **sudo -l**

The output showed that the **www-data** user had **NOPASSWD: ALL** privileges, meaning it could execute any command as root without needing a password. This was a critical finding for privilege escalation.

![Alt text](/assets/18.png)

After doing some searching in the file folders. I attempted to access the **/root** directory, but was initially denied due to permissions:

![Alt text](/assets/19.png)
![Alt text](/assets/20.png)

However, leveraging the NOPASSWD sudo privilege, I could list the contents of **/root** using **sudo ls -la /root**. This revealed a file named **3rd.txt**.

Finally, I used **sudo cat /root/3rd.txt** to read the contents of the file, successfully obtaining the third and final ingredient.

![Alt text](/assets/21.png)

## Conclusion
This challenge provided a comprehensive experience in web exploitation, including network scanning, directory enumeration, source code analysis, command injection. All three ingredients were successfully retrieved, completing Rick's potion.