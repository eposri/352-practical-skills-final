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

## Configuration Environment
__OS:__ Ubuntu   
__Tools:__ openssl, Apache2, node.js, Microsoft IIS   
__Files:__ caconf.cnf, server.crt, server.key, server.pem, cacert.pem, cakey.pem, cacert.crt, iis-server.crt, iis-server.pfx 

## Authors
__Contributor:__ Kevin Do  
__Contributions:__ Certificate Authority Setup on an Apache Web Server, Node.js Web Server, and Microsoft IIS Web Server; worked alongside with Alexis Rabago.

__Contributor:__ Alexis  
__Contributions:__  Certificate Authority Setup on an Apache Web Server, Node.js Web Server, and Microsoft IIS Web Server; worked alongside with Kevin Do.