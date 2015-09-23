## About

Automates the setting up of a VPN server.     
Uses StrongSwan 5.*       


## pre-requisites
[Ansible](http://www.ansible.com/)      
`sudo add-apt-repository -y ppa:rquillo/ansible && sudo apt-get update && sudo apt-get -y install ansible`

## How To

* open up vars/base.yml and specify this values:      

VPN_SERVER_IP: The public IP of the server      
ANSIBLE_SSH_USER: the user to ssh into server as       
PSK_PASSWORD: the paswword to be using to connect to the VPN server from your VPN client       
ETHERNET_INTERFACE: ethernet interface, leave it as is unless you know what you are doing.

* run `make setup`

* you can also run `vagrant up` locally to play around with the setup