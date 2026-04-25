# CPSC 352-01 Final Project
__Course:__ Cryptography  
__Semester:__ Spring 2026    
__Date:__ May 17, 2026  

California State University, Fullerton

## Practical Skills Cryptography

### Certificate Authority (CA) System

### VPN Server
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

### SSH Server

### IPSec Channel

### Pretty Good Privacy (PGP) E-mail

### DNSSEC-based DNS Server

### Kerberos

## Configuration Environment
1. CA System  
__OS:__   
__Tools:__  
__Files:__

2. VPN Server  
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
__OS:__   
__Tools:__ 
__Files:__

7. DNSSEC Server  
__OS:__   
__Tools:__  
__Files:__

8. Kerberos  
__OS:__   
__Tools:__  
__Files:__


## Authors
__Contributor:__ Kevin Do  
__Contributions:__ Certificate Authority Setup on an Apache Web Server, Node.js Web Server, and Microsoft IIS Web Server; worked alongside with Alexis (?).

__Contributor:__ Alexis  
__Contributions:__   

__Contributor:__  Kayla Ngo  
__Contributions:__ Bridged VPN server, SSH server, IPSec channel betweeen two systems; worked alongside with Gianna D.  
  
__Contributor:__ Gianna Davila  
__Contributions:__ Bridged VPN server, SSH server, IPSec channel betweeen two systems; worked alongside with Kayla N.  

__Contributor:__ Tom  
__Contributions:__     

__Contributor:__  Jesse  
__Contributions:__   

__Contributor:__  Maritza Esparza  
__Contributions:__   

_Submission for Final Project_


