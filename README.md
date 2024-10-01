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

2.5   - [Use SCP FTP and SFTP](#use-scp-ftp-and-sftp)

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

Here's how to create your pair of key: use the following command
```
ssh-keygen -t ed25519
```
***ssh-keygen** is the command, **-t** specify the type of key to create and **ed25519** is the encryption algorithm (*Ed25519 is preferred over RSA because it is faster, provides better security with shorter key sizes, and is considered to be more resistant to certain types of cryptographic attacks.*)

Now that your key is set locally you just need to copy it on the server:
```
ssh-copy-id username@serveradress
```
You will have a message (type in "yes"), and then you will be asked to enter your password. You will receive a confirmation, now try to connect and you will see the server asking you for your passphrase if you put one or it will connect using the key directly.
- - -
## Disable password authentification
- - -
## Use SCP FTP and SFTP
- - -
## Create a SSH tunnel
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


