# CPSC 352-01 Final Project
__Course:__ Cryptography  
__Semester:__ Spring 2026    
__Date:__ May 17, 2026  

California State University, Fullerton

## Practical Skills Cryptography

### Certificate Authority (CA) System
## **Overview**.


* **Objective:** To establish a Root CA and deploy signed certificates across three distinct web server environments.  
* **The Trust Model:** We have 1 Root CA and 3 leaves (apache, [node.js](http://node.js), microsoft iis) 

## **Technical Environment**.

* **Operating Systems:** Ubuntu 22.04/24.04 (CA/Apache/Node.js) and Windows 11 (IIS).  
* **Software:** OpenSSL v3.0+, Node.js v20+, Apache2, IIS.  
* **Network:** the CA, Apache, and [Node.js](http://Node.js) were on localhosts with different port numbers (insert port number here)

## **1\. Setting up the CA**

1. Create the environment where our CA will be in

```
cd && mkdir \-p myCA/signedcerts && mkdir myCA/private && cd myCA
```

\~/myCA : contains CA certificate, certificates database, generated certificates, keys, and requests

\~/myCA/signedcerts : contains copies of each signed certificate

\~/myCA/private : contains the private key

To generate a CA and key:
```
openssl req \-x509 \-newkey rsa:2048 \-out cacert.pem \-outform PEM \-days 1825
```

Creating a Self-Signed Server Certificate : 

Create a config file for the server

```

export OPENSSL\_CONF=\~/myCA/exampleserver.cnf
```

To use this as default for signing.

Generate CA and key : 
```
openssl req \-newkey rsa:1024 \-keyout tempkey.pem \-keyform PEM \-out tempreq.pem \-outform PEM
```

Turn temp key into server key
```

openssl rsa \< tempkey.pem \> server\_key.pem
```
Sign server CA with out own
```
export OPENSSL\_CONF=\~/myCA/caconfig.cnf
openssl ca \-in tempreq.pem \-out server\_crt.pem
```
Remove temp credentials 
```
rm \-f tempkey.pem && rm \-f tempreq.pem
```
We now have the self-signed server application certificate, and key pair:

server\_crt.pem : Server application certificate file

server\_key.pem : Server application key file


## **2\. Setting up IIS server**

To begin, we made sure our windows device had rootca.crt installed.  
When installing the crt, we need to store it in local host, and have it placed in the trusted root certification authorities  
![][image10] 

On the ubuntu VM, we make the pfx file using the following commands below, the iis server crt and pfx were imported to the windows machine.

```
// make the csr and key
openssl genrsa -out iis-server.key 2048
openssl req -new -key iis-server.key -out iis-server.csr

// sign with root ca
openssl x509 -req -in iis-server.csr -CA thenameofwhateverrootca.pem -CAkey thenameofwhateverrootca.key -CAcreateserial -out iis-server.crt -days 365 -sha256

// makes the pfx file
openssl pkcs12 -export -out iis-server.pfx -inkey iis-server.key -in iis-server.crt 
```

Once done, we open IIS and click on "server certificates"

We then import the iisserver.pfx here.  

After that, we need to ensure that windows will accept it so we go to the default web site, click bindings, and add a https binding that uses the iis server certificate.  

Side issues I ran into:  
Because of the way the CA was signed, I had to make up fullerton and [mydomain.com](http://mydomain.com) in my hosts file. It kept on displaying connection unsecure otherwise.  
![][image16]

Another important note, you must set the host name during “binding” to the actual domain name, because i named mine test earlier and not [mydomain.com](http://mydomain.com), it couldn’t find it.   



## **3\. Setting up the Apache Server**
```
Sudo apt install apache2   
Sudo systemctl start apache2  
```
On web URL, insert [http://127.0.0.1](http://127.0.0.1) to be send onto the apache frontpage 

If we tried to uses “https” instead of the base “http” we will be met with this error  
![][image3]
```
// Activate ssl module for apache  
a2enmod ssl
```
Go to /etc/apache2/sites-available/default-ssl.conf  
Change the SSLCertificateFIle and KeyFile location to wherever your server cert and key is located.
```
Sudo systemctl restart apache2
```
Go to https:localhost , and you should now see that the site cannot be reached.

This means that the Page now has a proper SSL CA \!  
Except your computer or browser does not recognize it as a safe certificate.  
Click on proceed and you'll be met with the same page, but with a slightly different URL

```
sudo a2enmod headers (both these commands r to move it to another port)
```
## **4\. Setting up [Node.js](http://Node.js) server**

This is the set of commands we used to download the [Node.js](http://Node.js) files:  
```
\# Download and install nvm:  
curl \-o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.4/install.sh | bash  
\# in lieu of restarting the shell  
\\. "$HOME/.nvm/nvm.sh"  
\# Download and install Node.js:  
nvm install 24  
\# Verify the Node.js version:  
node \-v \# Should print "v24.15.0".  
\# Verify npm version:  
npm \-v \# Should print "11.12.1".  
```
This is a simple file that creates a web page that just displays that it function correctly   
const http \= require('node:http');
```
const hostname \= '127.0.0.1';  
const port \= 6767;

const server \= http.createServer((req, res) \=\> {  
 res.statusCode \= 200;  
 res.setHeader('Content-Type', 'text/plain');  
 res.end('This is a test to see if the server is working.\\n');  
});

server.listen(port, hostname, () \=\> {  
 console.log(\`Server running at http://${hostname}:${port}/\`);  
});
```
Now the problem here is that that the webpage runs on simple http and not https, meaning its insecure.

HTTPS is the HTTP protocol over TLS/SSL. In Node.js this is implemented as a separate module.

We want to confirm that the https node support is included, and running the code before we can confirm that it does.  
let https;  
try {  
 https \= await import('node:https');  
} catch (err) {  
 console.error('https support is disabled\!');  
}

We then modify our code to include our certificates  
```
const https \= require('node:https');  
const fs \= require('node:fs');

const hostname \= '127.0.0.1';  
const port \= 6767;

// Reads the certificate and private key  
const ca\_authority \= {  
 key: fs.readFileSync('./certs/server.key'),  
 cert: fs.readFileSync('./certs/server.crt'),  
};

const server \= https.createServer(ca\_authority, (req, res) \=\> {  
 res.statusCode \= 200;  
 res.setHeader('Content-Type', 'text/plain');  
 res.end('This is a test to see if the HTTPS server is working.\\n');  
});

server.listen(port, hostname, () \=\> {  
 console.log(\`HTTPS Server running at https://${hostname}:${port}/\`);  
});
```
Now the page is now HTTPS, however the cert that we used isn't recognized by our webpage, so it gives us a warning. since we dont have the cert saved as a valid cert in our browser. We can justy proceed to the webpage since it is secure, its just that its not recognized by our browser.  

## **5\. VPN Server**
Key Generation Process:
```
# The following commands are from EasyRSA3 documentation:
./easyrsa init-pki
./easyrsa build-ca
./easyrsa gen-req <name_of_request>
./easyrsa sign-req <type> name_of_request

# The following command is from OpenVPN 2.x documentation:
openvpn --genkey tls-auth ta.key
```
For VPN server:
```
# Run the following in your CLI:
sudo apt update
sudo apt install bridge-utils
```
Bridge Settings:
| SETTING       | BRIDGE-START PARAMETER | VALUE |
| ------------- |:-------------:|:-------------:|
| Ethernet Interface     | eth     | eth0     |
| Local IP Address      | eth_ip     | `<SERVER_IP>`  |
| Local Netmask      | eth_netmask   | `<NETMASK_IP>`  |
| Local Broadcast Address      | eth_broadcast   | `<BROADCAST_IP>`  |
| VPN Client Address Pool      | -     | `<START_IP>` to `<END_IP>`     |
| Virtual Bridge Interface      | br     | br0     |
| Virtual TAP Interface      | tap     | tap0     |
```
# VPN configuration
server-bridge <eth_ip, eth_netmask, vpn_client_address_pool_start, vpn_client_address_pool_end>
tls-auth path/to/vpn_server/pki/private/ta.key
```

For Linux Firewall:
```
# The following commands are from OpenVPN 2.x documentation:
sudo iptables -A INPUT -i tap0 -j ACCEPT
sudp iptables -A INPUT -i br0 -j ACCEPT
sudo iptables -A FORWARD -i br0 -j ACCEPT
```
How to Run:
```
sudo ./bridge_start.sh
sudo openvpn server.conf    # terminal 0
sudo openvpn client.conf    # terminal 1 or different machine
# Ctrl + C to terminate openvpn
sudo ./bridge_stop.sh
```

## **6\. Setting up the SSH Server**

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
---



## **7\. IPSec Channel**
A site-to-site IPSec VPN was built between two Cisco 2911 routers ("CSUF" and "CALPOLY") in Cisco Packet Tracer, connected through a third router that simulates the public Internet. Traffic between the two private LANs (192.168.1.0/24 and 192.168.2.0/24) is protected by ESP in tunnel mode using AES-256 encryption and SHA-HMAC integrity; Phase 1 (ISAKMP / IKEv1) authenticates the peers with a pre-shared key and Diffie-Hellman Group 5.

### **1\. Topology**

Three 2911 routers (CSUF, CALPOLY, Internet), two 2960 switches, two PC-PT end devices. Each 2911 needs an HWIC-2T serial module installed (power off → drag module in → power on). Cable Serial DCE from Internet to each edge router, Copper Straight-Through from each edge router to its switch and from the switch to its PC. PC0 = 192.168.1.2/24 (gw .1), PC1 = 192.168.2.2/24 (gw .1).

### **2\. Activate the security license**

The default 2911 image has no `securityk9` feature set, so the crypto commands fail until it's enabled. On both edge routers:

```
license boot module c2900 technology-package securityk9
end
write memory
reload
```

Accept the EULA, then reload — the router boots with the security feature set active.

### **3\. Internet router**

No IPSec — just two serial interfaces forwarding between the WAN subnets:

```
hostname Internet
interface Serial0/0/0
 ip address 101.1.1.2 255.255.255.252
 clock rate 64000
 no shutdown
interface Serial0/0/1
 ip address 102.1.1.2 255.255.255.252
 clock rate 64000
 no shutdown
```

### **4\. CSUF router (CALPOLY is the mirror image)**

```
hostname CSUF
interface GigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown
interface Serial0/1/0
 ip address 101.1.1.1 255.255.255.252
 no shutdown
ip route 0.0.0.0 0.0.0.0 101.1.1.2

! Phase 1
crypto isakmp policy 10
 encryption aes 256
 hash sha
 authentication pre-share
 group 5
 lifetime 86400
crypto isakmp key CS352SECRET address 102.1.1.1

! Phase 2
crypto ipsec transform-set TSET esp-aes esp-sha-hmac

! Interesting traffic + crypto map (the ACL must be a mirror image on CALPOLY)
access-list 101 permit ip 192.168.1.0 0.0.0.255 192.168.2.0 0.0.0.255
crypto map CMAP 10 ipsec-isakmp
 set peer 102.1.1.1
 set transform-set TSET
 set pfs group5
 match address 101
interface Serial0/1/0
 crypto map CMAP
```

CALPOLY is the same with the addresses swapped: LAN `192.168.2.1/24`, WAN `102.1.1.1/30` on Serial0/0/0, default route via `102.1.1.2`, ISAKMP key tied to `101.1.1.1`, ACL with `192.168.2.0` as source and `192.168.1.0` as destination, crypto map peer = `101.1.1.1`.

Note: the standard reference config has a `mode tunnel` line inside the transform set, but Packet Tracer doesn't accept it (its transform-set doesn't enter a submode). Tunnel mode is the default anyway, so it's safe to omit.

### **5\. Bring up and verify**

The tunnel activates when interesting traffic hits the crypto map. From PC0:

```
ping 192.168.2.2
```

First one or two pings time out during IKE negotiation, then replies come back. Verify on either router:

```
show crypto isakmp sa     # state QM_IDLE, status ACTIVE  -> Phase 1 up
show crypto ipsec sa      # non-zero #pkts encaps/decaps  -> Phase 2 carrying traffic
```

## **8\. Pretty Good Privacy (PGP)**
### Overview
The Pretty Good Privacy (PGP) secure email system was implemented using Mozilla Thunderbird with built in OpenPGP support to give encrypted and digitally signed email communication between two separate accounts. The purpose of the system was to ensure confidentiality, authentication, and integrity of email messages through the use of public key cryptography.

The implementation used two configured email accounts where each account generated its own public and private key pair. Public keys were exchanged and imported through the OpenPGP Key Manager, allowing encrypted communication between both endpoints. Emails were encrypted using the receivers public key and digitally signed using the sender’s private key.

---

## Security Protocols

### SECURITY PROTOCOL: OpenPGP and Public Key Cryptography
The PGP email system uses asymmetric encryption through OpenPGP public/private key cryptography. Each user creates:
- A public key used for encryption and signature verification
- A private key used for decryption and digital signing

The public key is shared between users, while the private key remains securely stored locally within Thunderbird.

---

### SECURITY PROTOCOL: Digital Signatures and Integrity
Digital signatures were used to verify sender authenticity and message integrity. The sender signs the message using their private key, and the receiver verifies the signature using the sender’s public key. This confirms that the email was not modified during transmission.

---

### SECURITY PROTOCOL: End-to-End Encryption
The configured system provides end to end encryption , meaning only the sender and receiver can access the message data. Emails were encrypted using the recipients public key and decrypted only with the matching private key.

---

## Testing and Field Evaluation

### Test 1: Public Key Exchange

**Functionality Evaluated:**  
To verify that public keys could be successfully exported, shared, imported, and trusted between both accounts.

**Result:**  
Both accounts successfully imported each others public keys using the OpenPGP Key Manager.

**Corrective Action:**  
Initially, encryption was unavailable because the imported key was not properly trusted. Re-importing and accepting the key fixed the issue.

**Screenshot:**  
- OpenPGP Key Manager showing imported keys and trusted status

---

### Test 2: Encrypted Email Transmission

**Functionality Evaluated:**  
To verify that encrypted emails could only be decrypted by the intended recipient using their private key.

**Result:**  
Encrypted emails were successfully sent and decrypted by the receiving account.

**Corrective Action:**  
No issues faced after successful key exchange.

**Screenshot:**  
- Thunderbird encrypted email showing lock/encryption symbol

---

### Test 3: Digital Signature Verification

**Functionality Evaluated:**  
To verify sender identity and message integrity using digital signatures.

**Result:**  
The receiver successfully verified that the digital signature was valid, confirming the message authenticity and integrity.

**Corrective Action:**  
No issues encountered.

**Screenshot:**  
- Thunderbird “Signature Valid” verification message

---

### Test 4: End-to-End Communication

**Functionality Evaluated:**  
To verify complete encrypted and digitally signed communication between both configured email accounts.

**Result:**  
Both accounts successfully sent, received, decrypted, and verified encrypted messages, demonstrating fully functional secure communication.

**Corrective Action:**  
No issues encountered after setup completion.

**Screenshot:**  
- Account A → Account B encrypted/signed email  
- Account B → Account A encrypted/signed reply

---

## Configuration Environment

### PGP E-mail
**OS:** Linux (Ubuntu)  
**Tools:** Mozilla Thunderbird, Built-in OpenPGP Encryption  
**Files:** Public key files (.asc), private keys securely stored in Thunderbird, encrypted and digitally signed email messages used for secure communication testing between both accounts.

## **9\. DNSSEC-based DNS Server**

## **10\. Kerberos**

## Configuration Environment
1. CA System  
__OS:__ Ubuntu   
__Tools:__ openssl, Apache2, node.js, Microsoft IIS   
__Files:__ caconf.cnf, server.crt, server.key, server.pem, cacert.pem, cakey.pem, cacert.crt, iis-server.crt, iis-server.pfx  

3. VPN Server  
__OS:__ Kali Linux  
__Tools:__ OpenVPN (2.7), EasyRSA3, bridge-utils  
__Files:__ bridge_start.sh, bridge_stop.sh, client.conf, server.conf, ca.crt, ca.key, server.crt, server.key, client_0.crt, client_0.key

4. SSH Server  
__OS:__   
__Tools:__  
__Files:__  

5. IPSec Channel  
__OS:__   
__Tools:__  
__Files:__

6. PGP E-mail  
__OS:__ Linux (Ubuntu)  
__Tools:__ Mozilla Thunderbird (used for sending & recieving emails), Built-in OpenPGP in Thunderbird (for encryption & digital signatures)
__Files:__ Public key files(.asc), Private keys(stored safely from Thunderbird), Encrypted & digitally signed email messages used for testing communication between two accounts

DNSSEC Server
__OS:__ Ubuntu 24.04.2 LTS (WSL2)
__Tools:__ BIND9 9.18.39, dnssec-keygen, dnssec-signzone, dig
__Files:__ named.conf.local, cs352.local.zone, cs352.local.zone.signed,
           Kcs352.local.+008+21740.key, Kcs352.local.+008+25623.key

8. Kerberos  
__OS:__   
__Tools:__  
__Files:__


## Authors
__Contributor:__ Kevin Do  
__Contributions:__ Certificate Authority Setup on an Apache Web Server, Node.js Web Server, and Microsoft IIS Web Server; worked alongside with Alexis Rabago.

__Contributor:__ Alexis  
__Contributions:__  Certificate Authority Setup on an Apache Web Server, Node.js Web Server, and Microsoft IIS Web Server; worked alongside with Kevin Do.

__Contributor:__  Kayla Ngo  
__Contributions:__ Bridged VPN server, SSH server, IPSec channel betweeen two systems; worked alongside with Gianna D.  
  
__Contributor:__ Gianna Davila  
__Contributions:__ Bridged VPN server, SSH server, IPSec channel betweeen two systems; worked alongside with Kayla N.  

__Contributor:__ Tom  
__Contributions:__ Configured the IPSec site to site VPN tunnel between the CSUF and CALPOLY routers in Cisco Packet Tracer; built the three-router topology with HWIC-2T serial modules and Serial DCE cabling; activated the securityk9 license; set up 
                   ISAKMP Phase 1 (AES-256, SHA, pre-shared key, DH Group 5) and IPSec Phase 2 transform set (ESP-AES, ESP-SHA-HMAC); applied the crypto map to both WAN interfaces; verified the tunnel with show crypto isakmp sa, show crypto ipsec sa, 
                   and ESP packet inspection in Simulation Mode.    

__Contributor:__  Jesse  
__Contributions:__   

__Contributor:__ Mohamed Alqubaisi
__Contributions:__ DNSSEC-based DNS Server setup using BIND9 on Ubuntu 24.04 
                   WSL2; generated ZSK and KSK key pairs using RSASHA256 algorithm; configured 
                   authoritative DNS zone cs352.local; signed zone using dnssec-signzone; 
                   verified DNSSEC signatures using dig queries confirming RRSIG and DNSKEY 
                   records are present and valid.

__Contributor:__  Maritza Esparza  
__Contributions:__ Configured PGP secure email communication using Mozilla Thunderbird with OpenPGP; generated public and private key pairs for two accounts; exported and exchanged public keys; enabled encryption and digital signatures; sent encrypted and signed emails and verified decryption and signature authenticity on the recipients end.  

_Submission for Final Project_
