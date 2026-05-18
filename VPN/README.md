# CPSC 352-01 Final Project
__Course:__ Cryptography  
__Semester:__ Spring 2026    
__Date:__ May 17, 2026  

California State University, Fullerton

## Practical Skills: VPN Server
 For this project, a VPN server was configured using OpenVPN with TAP tunneling, which bridges networks at the data link layer (Layer 2) by encapsulating and transmitting Ethernet frames between the server and a client. A Public Key Infrastructure (PKI) was implemented internally using EasyRSA3 to create a root CA to sign and issue requests for certificates (x.509) and keys for the TLS/SSL connectivity protocol. HMAC signature was also configured with tls-auth directive in this setup to provide further security features to help protect against buffer overflow vulnerabilities, DDoS attacks, and unauthorized TLS handshaking attempts from unknown users.  


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
sudo iptables -A INPUT -i br0 -j ACCEPT
sudo iptables -A FORWARD -i br0 -j ACCEPT
```
How to Run:
```
# make scripts executable
chmod a+x bridge_start.sh
chmod a+x bridge_stop.sh

sudo ./bridge_start.sh
sudo openvpn server.conf    # terminal 0
sudo openvpn client.conf    # terminal 1 or different machine
# Ctrl + C to terminate openvpn
sudo ./bridge_stop.sh
```

## Configuration Environment
__OS:__ Kali Linux  
__Tools:__ OpenVPN (2.7), EasyRSA3, bridge-utils, hping3 (for testing)  
__Files:__ bridge_start.sh, bridge_stop.sh, client.conf, server.conf, ca.crt, ca.key, server.crt, server.key, client_0.crt, client_0.key, eth0_vm_kali.pcapng, windows_os_capt.pcapng

## Author
Contributor:__  Kayla Ngo  
__Contributions:__ Bridged VPN server
