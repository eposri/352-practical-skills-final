# CPSC 352-01 Final Project
__Course:__ Cryptography  
__Semester:__ Spring 2026    
__Date:__ May 17, 2026  

California State University, Fullerton

## Practical Skills: IPSec Channel
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
## Configuration Environment
__OS:__   
__Tools:__  
__Files:__

## Author
__Contributor:__ Tom  
__Contributions:__ Configured the IPSec site to site VPN tunnel between the CSUF and CALPOLY routers in Cisco Packet Tracer; built the three-router topology with HWIC-2T serial modules and Serial DCE cabling; activated the securityk9 license; set up 
                   ISAKMP Phase 1 (AES-256, SHA, pre-shared key, DH Group 5) and IPSec Phase 2 transform set (ESP-AES, ESP-SHA-HMAC); applied the crypto map to both WAN interfaces; verified the tunnel with show crypto isakmp sa, show crypto ipsec sa, 
                   and ESP packet inspection in Simulation Mode.  