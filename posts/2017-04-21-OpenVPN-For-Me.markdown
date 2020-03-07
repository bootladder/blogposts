---
layout: post
title:  "OpenVPN: What I need to know"
categories: guide
---

Remember:  
    only .key files should be kept confidential.
    .crt and .csr files can be sent over insecure channels such as plaintext email.
    do not need to copy a .key file between computers.
    each computer will have its own certificate/key pair. 

Step 4 — Generating Client Certificates
Each client will also each need a certificate and key in order to authenticate and connect to the VPN. Make sure you're in the /usr/local/etc/openvpn/easy-rsa/ directory.


=========================================================================
Server generates DH parameters  
openssl dhparam -out /etc/openvpn/dh2048.pem 2048

Before keys can be generated: 
Now, we can begin setting up the CA itself. First, initialize the Public Key Infrastructure (PKI).  
./build-ca    


Now we can build keys  
./build-key-server server  
Sign the certificate? [y/n]
1 out of 1 certificate requests certified, commit? [y/n]

**Step 7 — Move the Server Certificates and Keys**
*THIS IS IMPORTANT:  The server uses server.crt, server.key, ca.crt in some directory.  But they do not have to be created on the server; they can be moved to the server*
*In my case, I put the files somewhere else and referenced them in the server.ovpn file*

We will now copy the certificate and key to /etc/openvpn, as OpenVPN will search in that directory for the server's CA, certificate, and key.

    cp /etc/openvpn/easy-rsa/keys/{server.crt,server.key,ca.crt} /etc/openvpn



**Step 8 — Generate Certificates and Keys for Clients**

So far we've installed and configured the OpenVPN server, created a Certificate Authority, and created the server's own certificate and key. In this step, we use the server's CA to generate certificates and keys for each client device which will be connecting to the VPN.


*IMPORTANT:  the CA creates the keys for server and client.  The CA does not have to be on the server*

./build-key client1

Sign the certificate? [y/n]
1 out of 1 certificate requests certified, commit? [y/n]

Then, we'll copy the generated key to the Easy-RSA keys directory that we created earlier. Note that we change the extension from .conf to .ovpn. This is to match convention.

    cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf /etc/openvpn/easy-rsa/keys/client.ovpn

*IMPORTANT:  the SERVER needs copies of the client keys, and a client.ovpn*

We need to modify each client file to include the IP address of the OpenVPN server so it knows what to connect to.

Transferring Certificates and Keys to Client Devices

Recall from the steps above that we created the client certificates and keys, and that they are stored on the OpenVPN server in the /etc/openvpn/easy-rsa/keys directory.

For each client we need to transfer the client certificate, key, and profile template files to a folder on our local computer or another client device.

In this example, our client1 device requires its certificate and key, located on the server in:

    /etc/openvpn/easy-rsa/keys/client1.crt
    /etc/openvpn/easy-rsa/keys/client1.key

The ca.crt and client.ovpn files are the same for all clients. Download these two files as well; note that the ca.crt file is in a different directory than the others.

    /etc/openvpn/easy-rsa/keys/client.ovpn
    /etc/openvpn/ca.crt

================================================

crt and key files represent both parts of a certificate, key being the private key to the certificate and crt being the signed certificate.
