# CPSC 352-01 Final Project
__Course:__ Cryptography  
__Semester:__ Spring 2026    
__Date:__ May 17, 2026  

California State University, Fullerton

## Practical Skills: DNSSEC

## Configuration Environment
__OS:__ Ubuntu 24.04.2 LTS (WSL2)
__Tools:__ BIND9 9.18.39, dnssec-keygen, dnssec-signzone, dig
__Files:__ named.conf.local, cs352.local.zone, cs352.local.zone.signed,
           Kcs352.local.+008+21740.key, Kcs352.local.+008+25623.key

## Author
__Contributor:__ Mohamed Alqubaisi
__Contributions:__ DNSSEC-based DNS Server setup using BIND9 on Ubuntu 24.04 
                   WSL2; generated ZSK and KSK key pairs using RSASHA256 algorithm; configured 
                   authoritative DNS zone cs352.local; signed zone using dnssec-signzone; 
                   verified DNSSEC signatures using dig queries confirming RRSIG and DNSKEY 
                   records are present and valid.