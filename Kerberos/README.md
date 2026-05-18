# CPSC 352-01 Final Project
__Course:__ Cryptography  
__Semester:__ Spring 2026    
__Date:__ May 17, 2026  

California State University, Fullerton  

## Practical Skills: Kerberos Setup (Ubuntu)
This project sets up a simple Kerberos server and client on one Ubuntu machine. The same system is used as both the server and client for simplicity. Kerberos provides secure authentication using tickets instead of repeatedly sending passwords.  

## System Info
Hostname: kerberos-server.local  
Realm: MYEXAMPLE.COM  
User: kuser  

## Setup

Install Kerberos:
sudo apt update
sudo apt install krb5-kdc krb5-admin-server krb5-user -y

Set hostname:
sudo hostnamectl set-hostname kerberos-server.local

Configure hosts file:
sudo nano /etc/hosts

Add:
127.0.0.1 localhost
127.0.1.1 kerberos-server.local kerberos-server

Configure Kerberos:
sudo dpkg-reconfigure krb5-config

Use:
Realm: MYEXAMPLE.COM  
Server: kerberos-server.local  

Create Kerberos database:
sudo krb5_newrealm

Allow admin access:
sudo nano /etc/krb5kdc/kadm5.acl

Add:
*/admin *

Start services:
sudo systemctl start krb5-kdc
sudo systemctl start krb5-admin-server

Create user:
sudo kadmin.local
addprinc kuser
quit

Login (client):
kinit kuser

Verify ticket:
klist

## How It Works
The user logs in using kinit, the Kerberos server verifies the password, and then issues a ticket. The client uses this ticket instead of sending the password again, making authentication more secure.  

## Configuration Environment
__OS:__ Ubuntu Linux  
__Tools:__ krb5-kdc, krb5-admin-server, krb5-user, kadmin.local, kinit, klist, systemctl  
__Files:__ /etc/krb5.conf, /etc/hosts, /etc/krb5kdc/kadm5.acl, /var/lib/krb5kdc/principal  

## Author
__Contributor:__  Jesse  
__Contributions:__ Set up and configured the Kerberos server and client, installed required packages, configured hostname and system files, created and managed user principals, tested authentication using kinit and klist, and documented the setup process.