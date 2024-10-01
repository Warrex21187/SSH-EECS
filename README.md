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

5. [Analysis](#analysis)

6. [Ressources](#ressources)

# Introduction
For my first network lesson, I had to create and setup a whole server using SSH. The goal was to learn the usage of many tools, like SSH, SCP, ufw, and use them to setup our own working server. In this depository you will find my complete experience, I will list all the steps that were required to finish the assignment, listing: the commands used, what I learned. Then I will list the issues I faced, and the solutions I found. I will finish with an analysis where I will mostly talk about the matter of SSH for data security and the pros of the security measures. 
- - -
# List of Manipulations
## VPS Creation
To start the work, I needed a VPS to (Virtual Private Server). It's a distant server that is fractionned into smaller dedicated servers. I paid one small VPS on [Ionos](https://www.ionos.fr/). 

Now I needed to create a user so I will not have to connect into r00t because it can cause many issues (all details in (#issue-with-r00t-user)). To do so, I had to connect into root user using the following command: 
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
Now that user is set, let's talk quickly about Key Authentification. We have a password, so why do we would use a Key over a password ? Well for many reasons: firstly for security reasons, **SSH Keys Are Cryptographically Stronger**, the SSH key pair (local and server) is much stronger than a typical password, making it nearly impossible for an attacker to brute-force. It's also more easier to use, and easy to configure: just generate it on your local machine, and then copy it onto the server. It will not ask you to enter a password everytime you want to connect (*except if you put a Key Phrase*) so you don't risk to forgot your password. 

Here's how I created a pair of key:
```
ssh-keygen -t ed25519
```
**ssh-keygen** is the command, **-t** specify the type of key to create and **ed25519** is the encryption algorithm (*Ed25519 is preferred over RSA because it is faster, provides better security with shorter key sizes, and is considered to be more resistant to certain types of cryptographic attacks.*)

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
**nano** gave me access and the ability to edit the file.

Now that I was in, looked for the "Password Authentification yes" line, uncommented it and replaced it by "Password Authentification no".

Next saved it and closed the window by doing this combination on keys: *CRTL+O, Enter, CRTL+X*

Then restart the SSH service with the command:
```
sudo systemctl restart ssh
```
Then test the connexion, if you connect to the server with another computer, it will use the Key or ask you for the Passphrase **and not the password**
If you've done all these steps and it still asks you for your password, consult
- - -
## Use SCP FTP and SFTP
Now I learned  you how to transfer files from your local machine to the server using the SCP Protocol (Secure Copy Protocol), it encrypts the file and send them surely.

First I needed a file to transfer, here's the command: (*careful you need to create it in your local machine not in the server*)
```
echo "Ceci est un fichier test" > fichier.txt
```
**echo** is the command, what's in quotes is the file's content and **> file name**

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
- - -
## Create a SSH tunnel
Now that my server is up locally, I needed a way to route network traffic from my local machine through a remote server. For that I used the Secure Shell (SSH) protocol, to setup a SSH Tunnel.

First step is to redirect the local port **8888** to the port **80** of a distant server. I used the following commands:

```
ssh -L 8080:localhost:80 username@serveraddress
```
Now I tried to connect to the server by connecting to: http://localhost:8080 or 192.67.197.102

But it didn't worked. The reason is simple: I needed to install a web server of my VPS (because if I want to connect to a web server without the web server existing it's complicated). So I installed Nginx using the following commands:
```
sudo apt update #look for updates
sudo apt install nginx #install nginx
sudo systemctl start nginx #start nginx
sudo systemctl enable nginx #enable nginx to start on boot
```
Now that Nginx is succesfully installed I can connect to the server.
- - -
## Change the SSH Port
- - - 
## Setup Fail2Ban
- - -
# Issues and solutions
- - -
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


