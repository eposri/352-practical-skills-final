# CPSC 352-01 Final Project
__Course:__ Cryptography  
__Semester:__ Spring 2026    
__Date:__ May 17, 2026  

California State University, Fullerton

## Practical Skills: SSH Server
Setting up an SSH server so a user can securely connect without having to enter a password each time. SSH key-based auth uses two cryptographic keys:
- a private key that stays on the client
- a public key that gets placed on the server.

This is asymmetric cryptography applied through the Ed25519 algorithm (Edwards-curve Digital Signature Algorithm over Curve25519), chosen for its optimal balance of security and performance compared to RSA and ECDSA.
 
Both machines are set to **Bridged** network mode so they each receive their own IP on the local network and can communicate directly. NAT (the default) only gives VMs a private address visible to the hypervisor, not to other devices.
 
### Setup
 
**1. Install OpenSSH**
On both machines:
```
sudo apt update
sudo apt install openssh-server openssh-client
```
`openssh-client` to initiate connections and `openssh-server` to receive them.
 
**2. Generate the SSH Key Pair (Client)**
 
```
ssh-keygen -t ed25519 -C "label"
```
 
This generates:
- `~/.ssh/id_ed25519` — private key
- `~/.ssh/id_ed25519.pub` — public key
The key fingerprint displayed after generation (`SHA256:...`) is a SHA-256 hash of the public key, used to verify a key's identity without comparing the entire contents.
 
**3. Verify Password Access is Enabled (Server)**
 
```
sudo sshd -T | grep passwordauthentication
```
Should return `passwordauthentication yes` — we need this to copy the key over initially.
 
**4. Start the SSH Service (Server)**
 
```
sudo systemctl enable ssh
sudo systemctl start ssh
sudo systemctl status ssh
```
Should show `active (running)`.
 
**5. Verify Connectivity**
 
From the client, ping the server to confirm the machines can reach each other:
```
ping <SERVER-IP>
```
 
**6. Copy the Public Key to the Server**
 
From the client:
```
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@<SERVER-IP>
```
This places the public key into `~/.ssh/authorized_keys` on the server. It will ask for the password one final time.
 
**7. Verify the Key Was Added (Server)**
 
```
cat ~/.ssh/authorized_keys
```
 
**8. Connect**
 
From the client:
```
ssh user@<SERVER-IP>
``` 

## Configuration Environment
__OS:__   
__Tools:__  
__Files:__  

## Author
__Contributor:__ Gianna Davila  
__Contributions:__ SSH server  