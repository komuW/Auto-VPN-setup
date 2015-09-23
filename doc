#https://www.zeitgeist.se/2013/11/22/strongswan-howto-create-your-own-vpn/
#https://www.gypthecat.com/ipsec-vpn-host-to-host-on-ubuntu-14-04-with-strongswan

$ sudo apt-get -y install strongswan
$ ipsec version
	Linux strongSwan U5.1.2/K3.13.0-55-generic
	Institute for Internet Technologies and Applications
	University of Applied Sciences Rapperswil, Switzerland
	See 'ipsec --copyright' for copyright information.

# Let’s start by creating a self-signed root CA certificate.
 #do this as root user
$ cd /etc/ipsec.d/
$ ipsec pki --gen --type rsa --size 4096 --outform pem > private/KomuVPNrootKey.pem
$ chmod 600 private/KomuVPNrootKey.pem
$ ipsec pki --self --ca --lifetime 3650 --in private/KomuVPNrootKey.pem --type rsa --dn "C=US, O=KomuVPN, CN=KomuVPN-Root-CA" --outform pem > cacerts/KomuVPNRootCert.pem
  #country (C), organization (O), and common name (CN)
  #KomuVPNRootCert.pem expires in 3650days(10yrs)
  # KomuVPNRootCert.pem and KomuVPNrootKey.pem should be stored securely; away from the internet

#create vpn host certificate
$ cd /etc/ipsec.d/
$ ipsec pki --gen --type rsa --size 2048 --outform pem > private/vpnHostKey.pem
$ chmod 600 private/vpnHostKey.pem
$ ipsec pki --pub --in private/vpnHostKey.pem --type rsa | ipsec pki --issue --lifetime 2920 --cacert cacerts/KomuVPNRootCert.pem --cakey private/KomuVPNrootKey.pem --dn "C=US, O=KomuVPN, CN=10.0.0.9" --san 10.0.0.9 --flag serverAuth --flag ikeIntermediate --outform pem > certs/vpnHostCert.pem
  #note: --dn "C=US, O=KomuVPN, CN=vpn.zeitgeist.se" --san vpn.zeitgeist.se
  #u must include (eg vpn.zeitgeist.se) ie the domain name or IP address of your VPN server in both thoe places 
  #the cert(vpnHostCert.pem) generated has a validity of 2920days(8yr)

  # to check: $ ipsec pki --print --in certs/vpnHostCert.pem

#create client cert
#Any client will require a personal certificate in order to use the VPN. 
$ cd /etc/ipsec.d/
$ ipsec pki --gen --type rsa --size 2048 --outform pem > private/KomuClientKey.pem
$ chmod 600 private/KomuClientKey.pem
$ ipsec pki --pub --in private/KomuClientKey.pem --type rsa | ipsec pki --issue --lifetime 2920 --cacert cacerts/KomuVPNRootCert.pem --cakey private/KomuVPNrootKey.pem --dn "C=US, O=KomuClient, CN=komuw05@gmail.com" --san komuw05@gmail.com --outform pem > certs/KomuClientCert.pem


#export cliet cert
#A VPN client needs a client cert, its private key, and the signing CA certificate. 
#best way is to package all in a PKCS#12 file and export it with a paraphrase.
$ cd /etc/ipsec.d/
$ openssl pkcs12 -export -inkey private/KomuClientKey.pem -in certs/KomuClientCert.pem -name "komu cert" -certfile cacerts/KomuVPNRootCert.pem -caname "Root CA" -out KomuClient.p12
  #u'll be asked for pasword(passphrase): komu
  #now u can send KomuClient.p12 and its export paraphrase to the person who’s going to install it onto the client.


#Revoking a client cert
# If a certificate is lost/stolen, it must be revoked so nobody can use it to connect to your VPN server. 
$ cd /etc/ipsec.d/
$ ipsec pki --signcrl --reason key-compromise --cacert cacerts/KomuVPNRootCert.pem --cakey private/KomuVPNrootKey.pem --cert certs/KomuClientCert.pem --outform pem > crls/crl.pem
  #This generates the new certificate revocation list (CRL) crls/crl.pem.

#To add another revoked certificate to the same list, we need to copy the existing list into a temporary file:
$ cd /etc/ipsec.d/
$ cp crls/crl.pem crl.pem.tmp
$ ipsec pki --signcrl --reason key-compromise --cacert cacerts/KomuVPNRootCert.pem --cakey private/KomuVPNrootKey.pem --cert certs/AnotherStolenCert.pem --lastcrl crl.pem.tmp --outform pem > crls/crl.pem
$ rm crl.pem.tmp

#certificates recap
/etc/ipsec.d/private/KomuVPNrootKey.pem  # CA private key
/etc/ipsec.d/cacerts/KomuVPNRootCert.pem # CA certificate
/etc/ipsec.d/private/vpnHostKey.pem     # VPN host private key
/etc/ipsec.d/certs/vpnHostCert.pem      # VPN host certificate
/etc/ipsec.d/private/KomuClientKey.pem   # Client "Alexander" private key
/etc/ipsec.d/certs/KomuClientCert.pem   # Client "Alexander" certificate
/etc/ipsec.d/KomuClient.p12              # Client "Alexander" PKCS#12 file

#Server conf
  /etc/strongswan.conf
  /etc/ipsec.conf
  /etc/ipsec.secrets

#/etc/ipsec.conf file:
#full list of options: http://wiki.strongswan.org/projects/strongswan/wiki/StrongswanConf
# ipsec.conf - strongSwan IPsec configuration file

config setup
	# uniqueids=never
	charondebug="cfg 2, dmn 2, ike 2, net 2"

conn %default
	keyexchange=ikev2
	ike=aes128-sha256-ecp256,aes256-sha384-ecp384,aes128-sha256-modp2048,aes128-sha1-modp2048,aes256-sha384-modp4096,aes256-sha256-modp4096,aes256-sha1-modp4096,aes128-sha256-modp1536,aes128-sha1-modp1536,aes256-sha384-modp2048,aes256-sha256-modp2048,aes256-sha1-modp2048,aes128-sha256-modp1024,aes128-sha1-modp1024,aes256-sha384-modp1536,aes256-sha256-modp1536,aes256-sha1-modp1536,aes256-sha384-modp1024,aes256-sha256-modp1024,aes256-sha1-modp1024!
	esp=aes128gcm16-ecp256,aes256gcm16-ecp384,aes128-sha256-ecp256,aes256-sha384-ecp384,aes128-sha256-modp2048,aes128-sha1-modp2048,aes256-sha384-modp4096,aes256-sha256-modp4096,aes256-sha1-modp4096,aes128-sha256-modp1536,aes128-sha1-modp1536,aes256-sha384-modp2048,aes256-sha256-modp2048,aes256-sha1-modp2048,aes128-sha256-modp1024,aes128-sha1-modp1024,aes256-sha384-modp1536,aes256-sha256-modp1536,aes256-sha1-modp1536,aes256-sha384-modp1024,aes256-sha256-modp1024,aes256-sha1-modp1024,aes128gcm16,aes256gcm16,aes128-sha256,aes128-sha1,aes256-sha384,aes256-sha256,aes256-sha1!
	dpdaction=clear
	dpddelay=300s
	rekey=no
	left=%any
	leftsubnet=0.0.0.0/0
	leftcert=vpnHostCert.pem
	right=%any
	rightdns=8.8.8.8,8.8.4.4
	rightsourceip=172.16.16.0/24

conn IPSec-IKEv2
	keyexchange=ikev2
	auto=add

conn IPSec-IKEv2-EAP
	also="IPSec-IKEv2"
	rightauth=eap-mschapv2
	rightsendcert=never
	eap_identity=%any

conn CiscoIPSec
	keyexchange=ikev1
	# forceencaps=yes
	rightauth=pubkey
	rightauth2=xauth
	auto=add

#full options: http://wiki.strongswan.org/projects/strongswan/wiki/IpsecConf
  #This configuration has settings for three types of VPN services: IKEv2 + RSA certificate, IKEv2 + EAP, and IKEv1 + Xauth RSA,
  #thus providing compatibility for a wide range of IPsec clients.
  ## uniqueids=never: : (disabled here) by default only one client can connect at the same time with an identical certificate and/or password  combo; 
  #the newer connection will always replace the older. 
  #If you don’t like this, for instance because you want to use the same client certificates on multiple clients at the same time, enable this option
  #NB:: leftcert=vpnHostCert.pem change it to name your host VPN certificate instead.


#/etc/ipsec.secrets file:

# This file holds shared secrets or RSA private keys for authentication.
# RSA private key for this host, authenticating it to any other host
# which knows the public part.  Suitable public keys, for ipsec.conf, DNS,
# or configuration of other implementations, can be extracted conveniently
# with "ipsec showhostkey".

: RSA vpnHostKey.pem
user1 : EAP "topsecretpassword"
user2 : XAUTH "evenmoretopsecretpassword"

  # identifies the private key of the VPN host to allow your host to authenticate itself with its host certificate
  # defines an EAP credential (username / password) that can be used by clients to connect without client certificate
  # defines an XAUTH credential (username / password) that is required in addition to a client certificate for IKEv1 + Xauth RSA connections (eg for iOS clients)

$ ipsec rereadsecrets #whenever u edit /etc/ipsec.secrets 

#/etc/sysctl.conf file:
# VPN
net.ipv4.ip_forward = 1
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0

#update iptables rules
$ iptables -t nat -A POSTROUTING -o eth0 ! -p esp -j SNAT --to-source <your VPN host IP>
  #remember to add in ua vpnHostIP and adjust the interface if its not eth0
iptables -t nat -A POSTROUTING -o eth0 ! -p esp -j SNAT --to-source 10.0.0.9

#If you have a restrictive firewall for incoming traffic, allow IPsec communications. Three rules are required:
$ iptables -A INPUT -p udp --dport 500 --j ACCEPT
$ iptables -A INPUT -p udp --dport 4500 --j ACCEPT
$ iptables -A INPUT -p esp -j ACCEPT

# That’s it! Restart strongSwan and your VPN server is ready.
$ ipsec restart #service ipsec restart


##Client configuration
You cannot do anything with until you’ve configured your clients.
Google how to do that