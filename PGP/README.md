# CPSC 352-01 Final Project
__Course:__ Cryptography  
__Semester:__ Spring 2026    
__Date:__ May 17, 2026  

California State University, Fullerton

## Practical Skills: Pretty Good Privacy (PGP)
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
**OS:** Linux (Ubuntu)  
**Tools:** Mozilla Thunderbird, Built-in OpenPGP Encryption  
**Files:** Public key files (.asc), private keys securely stored in Thunderbird, encrypted and digitally signed email messages used for secure communication testing between both accounts.

## Author
__Contributor:__  Maritza Esparza  
__Contributions:__ Configured PGP secure email communication using Mozilla Thunderbird with OpenPGP; generated public and private key pairs for two accounts; exported and exchanged public keys; enabled encryption and digital signatures; sent encrypted and signed emails and verified decryption and signature authenticity on the recipients end.  
