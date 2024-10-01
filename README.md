# **SSH Protocol**
## How to setup a server using SSH
### In this readme you will find all the steps required to create and secure a SSH server with basic knowledge.
- - - 
# Table of Contents
1. [Introduction](#introduction)

2. [List of manipulations](#list-of-manipulations)

2.1   - [VPS creation](#vps-creation)

2.2   - [Connect to the VPS server](#connect-to-the-vps-server)

2.3   - [Key Authentification](#key-authentification)

2.4   - [Disable password authentification](#disable-password-authentification)

2.5   - [Use SCP and SFTP](#use-scp-ftp-and-sftp)

2.6   - [Create a SSH tunnel](#create-a-ssh-tunnel)

2.7   - [Change the SSH port](#change-the-ssh-port)

2.8   - [Setup Fail2ban](#setup-fail2ban)

4. [Issues and solutions](#issues-and-solutions)

      - [Issues with r00t user](#issues-with-r00t-user)
      - [Password Authentification still activated](#password-authentification-still-activated)
      - [Issues with Port22](#issues-with-port22)
      - [Can't connect on Port 2222](#can't-connect-on-port-2222)
      - [How to unban an user Fail2Ban](#how-to-unban-an-user-fail2ban)

5. [Analysis](#analysis)

6. [Ressources](#ressources)

# Introduction
For my first network lesson, I had to create and setup a whole server using SSH. The goal was to learn the usage of many tools, like SSH, SCP, ufw, and use them to setup my own working server. In this depository you will find my complete experience, I will list all the steps that were required to finish the assignment, listing the commands used, and what I learned. Then I will list the issues I faced, and the solutions I found. I will finish with an analysis where I will mostly talk about the matter of SSH for data security and the pros of the security measures. 
- - -
# List of Manipulations
## VPS Creation
To start the work, I needed a VPS  (Virtual Private Server). It's a distant server that is fractionned into smaller dedicated servers. I paid one small VPS on [Ionos](https://www.ionos.fr/). 

Now I needed to create a user so I will not have to connect into r00t because it can cause many issues (all details in ([Issue with r00t user](#issue-with-r00t-user)). To do so, I had to connect into root user using the following command: 
```
ssh root@serveradress
```
Then I needed to enter the password that was hidden into the Ionos server panel. 
After being successfully connected I needed to create a new user using the following command:
```
adduser "name"
```
and then fill in all the required informations (password and optionnal fields).
- - -
## Connect to the VPS server
Now that I had created my user I needed to give him all permissions. I typed the following command:
```
sudo visudo
```
And in "# User privilege specification" I added the following line:
```
loris ALL=(ALL:ALL) ALL
```
(To put your user just replace "loris" by your own username)
By that I gave to my user, all the sudo permission to use locked commands.
Now I just needed to disconnect and reconnect using the following command:
```
ssh loris@serveradress
```
- - -
## Key Authentification
Now that user is set, let's talk quickly about Key Authentification. We have a password, so why do we would use a Key over a password ? Well for many reasons: firstly for security reasons, **SSH Keys Are Cryptographically Stronger**, the SSH key pair (*local and server*) is much stronger than a typical password, making it nearly impossible for an attacker to brute-force. It's also more easier to use, and easy to configure: just generate it on your local machine, and then copy it onto the server. It will not ask you to enter a password everytime you want to connect (*except if you put a Key Phrase*) so you don't risk to forgot your password. 

Here's how I created a pair of key:
```
ssh-keygen -t ed25519
```
**ssh-keygen** is the command to create a key, **-t** specify the type of key to create and **ed25519** is the encryption algorithm (*Ed25519 is preferred over RSA because it is faster, provides better security with shorter key sizes, and is considered to be more resistant to certain types of cryptographic attacks.*)

Now that my key is set locally I needed to copy it on the server:
```
ssh-copy-id username@serveradress
```
You will have a message (type in "yes"), and then you will be asked to enter your password. You will receive a confirmation, now try to connect and you will see the server asking you for your passphrase if you put one or it will connect using the key directly.
- - -
## Disable password authentification
Now that I've set a Key up, I wanted to disable Password authentification to add another layer of security.
To access and modify the file located in /etc/ssh/sshd_config I used the following command:
```
sudo nano /etc/ssh/sshd_config
```
**nano** gave me access to the file and the ability to edit this file.

Now that I was in, I looked for the "Password Authentification yes" line, uncommented it and replaced it by **"Password Authentification no"**.

Next saved it and closed the window by doing this combination of keys: *CRTL+O, Enter, CRTL+X*

Then restart the SSH service with the command:
```
sudo systemctl restart ssh
```
Then I tested the connexion, if you connect to the server with another computer, it will use the Key or ask you for the Passphrase **and not the password**
If you've done all these steps and it still asks you for your password, consult([Password Authentification still activated](#password-authentification-still-activated))
- - -
## Use SCP FTP and SFTP
Next I learned how to transfer files from my local machine to the server using the SCP Protocol (Secure Copy Protocol), it encrypts the file and send them surely.

First I needed a file to transfer, here's the command: (*careful you need to create it in your local machine not in the server*)
```
echo "Ceci est un fichier test" > fichier.txt
```
**echo** is the command that create a file, what's in quotes is the file's content and **> file name** is how the file will be named

Now to send it use the following command:
```
scp fichier.txt username@serveraddress:/home/username/
```
For example if I want to move it onto my server:
```
scp fichier.txt loris@serveraddress:/home/loris/
```
To check if the file has been succesfully sent use the following command:
```
ssh username@serveradress "ls -l /home/username/"
```
There's another protocol called SFTP that allow for an interactive transfer.
To connect with SFTP use command:
```
sftp user@serveradress
```
And to transfer the file simply use the command:
```
put "fichier.txt"
```
Remember to move into the folder where you want to place the file using the **cd** and **pwd** commands. 
- - -

## Create a SSH tunnel
Now that my server is up locally, I needed a way to route network traffic from my local machine through a remote server. For that I used the Secure Shell (SSH) protocol, to setup a SSH Tunnel.

First step is to redirect the local port **8888** to the port **80** of a distant server. I used the following commands:

```
ssh -L 8080:localhost:80 username@serveraddress
```
Now I tried to connect to the server by connecting to: http://localhost:8080 or 192.67.197.102

But it didn't worked. The reason is simple: I needed to install a web server of my VPS (because if I want to connect to a web server without the web server existing it's impossible). So I installed Nginx using the following commands:
```
sudo apt update #look for updates
sudo apt install nginx #install nginx
sudo systemctl start nginx #start nginx
sudo systemctl enable nginx #enable nginx to start on boot
```
Now that Nginx is succesfully installed I can connect to the server. The SSH tunnel is now ready, if I want to connect on the server without being in localhost (for example with my phone) it's possible.
- - -
## Change the SSH Port
Now I want to change the SSH Port, because by default it's set on the Port 22, with this port there are several issues with this port (check [Issues with Port22](#issues-with-port22))

To change the port I went in /etc/ssh/sshd_config, found the line Port and put **Port 2222**

WARNING: To avoid being fully rejected by the firewall and not being able to reconnect if connection with Port 2222 is impossible, add another line **Port 22** until you are sure you can connect on port 2222, otherwise you'll have to re install your VPS and start everything from scratches...

Then I am going to add rules on the firewall using **ufw**.

To allow a new port use the following command:
```
sudo ufw allow (port/rule)
```
Here is a list of the port you need to open:
```
sudo ufw allow OpenSSH #to allow SSH
sudo ufw allow 22 #allow port 22 to avoid full lock if port 2222 don't work
sudo ufw allow 2222 #allow port 2222
sudo ufw allow http #allow HTTP
sudo ufw allow https  #allow HTTPS
sudo ufw allow ‘Nginx Full’ #to allow Nginx
```
Now I want to connect using the port 2222. I use the following command:
```
ssh -p 2222 loris@serveradress
```
If connection on port 2222 fails check [Can't connect on Port 2222](#can't-connect-on-port-2222)
- - - 
## Setup Fail2Ban
To avoid brute-force attack on my server I added a simple but efficient solution called **Fail2Ban**
To install Fail2Ban I used the following command:
```
sudo apt install fail2ban
```
Then I configured it by accessing config file:
```
[sshd]
enabled = true
port = 2222
```
Add these lines, then restart Fail2Ban for it to be operationnal:
```
sudo systemctl restart fail2ban
```
Now if someone is trying to connect with SSH and fail password or key auth too many times, his IP will be banned.
If a user is banned check [How to unban an user Fail2Ban](#how-to-unban-an-user-fail2ban)
- - -
# Issues and solutions
## Issues with r00t user
Here are some issues by operating the server while in root:
- Security Risks

     Increased Attack Surface: The root account has unrestricted access to all commands and files on the system. If compromised, an attacker can gain full control of the system.
     Target for Attacks: Since the root account is well-known, it is often a primary target for brute-force attacks. Attackers may try to guess the root password to gain access.

-  Accidental Damage

    Unintended Changes: As the root user, any command executed can affect the entire system. Accidental commands (e.g., rm -rf /) can lead to catastrophic data loss or system corruption.
    Overwriting Critical Files: Being root allows you to overwrite important system files, which can lead to system instability or failure.


- Inconsistent Environments

    Different Environment Variables: The environment for the root user may differ from that of regular users. This can lead to inconsistencies when running scripts or applications that behave differently under root versus a standard user.

- Bypassing User-Level Security

    Ignoring Permissions: When using root, you bypass user-level permissions, which can expose the system to accidental or malicious actions that could compromise user data.

-  Misconfigurations

    Improper Configuration Changes: Changes made by the root user might inadvertently misconfigure services or security settings, leading to vulnerabilities or service outages.

You should use Sudo instead of root user to avoid mistakes.
- - -
## Password Authentification still activated
If you can still connect by using your password, it may come from a conflict between different authentification system.
First check if the PubkeyAuthentication yes line is present and uncommented in the config file in /etc/ssh/sshd_config (with the command **sudo nano /etc/ssh/sshd_config**) and restart SSH with **sudo systemctl restart ssh**.
Then check files permission with the following commands:
```
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```
To check if password authentification status use command:
```
sudo sshd -T | grep passwordauthentication
```
If it still doesn't work, go again in /etc/ssh/sshd_config, and look/add the following line: 
- **ChallengeResponseAuthentication no**
- **UsePAM no**
  Then restart SSH (sudo systemctl restart ssh)
  NOTE: These solutions didn't worked for me even tho they should've, something may be missing.
- - - 
## Issues with Port22
Here's a list of issues with the Port 22 and why you should **NOT** use it.
- Port 22 is the default Port for SSH, it's the most known Port, a lot of malicious bot are set up to scan IP adress and try to force-brut attack. Changing the port will not stop these attack tho, but will make it way harder to achieve.
- It will reduce the amount of malicious bot's scan because they are configured to scan only specific ports
- 



# Analysis
- - -
# Ressources
- - -
### Advanced Usage
Advanced usage details.

# Contributing
Guidelines for contributing.

# License
License details.


