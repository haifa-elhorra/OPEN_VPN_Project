# OpenVPN Server Setup on Fedora

This guide provides step-by-step instructions to set up an OpenVPN server on Fedora Linux.

## Prerequisites
- A Fedora Linux server with root/sudo access
- Basic familiarity with command line operations

## Installation

1. Update system and install required packages:

```bash
sudo dnf update -y
sudo dnf install -y openvpn easy-rsa
 ```

##  Certificate Authority Setup
2. Initialize Easy-RSA and create certificates:
   
```bash
sudo cp -r /usr/share/easy-rsa /etc/openvpn/
cd /etc/openvpn/easy-rsa
sudo ./easyrsa init-pki
sudo ./easyrsa build-ca nopass
sudo ./easyrsa gen-req server nopass
sudo ./easyrsa sign-req server server
sudo ./easyrsa gen-dh
sudo openvpn --genkey --secret ta.key
 ```

## Server Configuration
3. Copy certificates and keys to OpenVPN directory:
       
```bash
sudo cp pki/ca.crt pki/private/server.key pki/issued/server.crt ta.key pki/dh.pem /etc/openvpn/server/
 ```
4. Enable IP forwarding:

```bash
sudo sed -i 's/#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/' /etc/sysctl.conf
sudo sysctl -p
 ```
## Configure OpenVPN server configuration file /etc/openvpn/server/server.conf
There is a sample in /usr/share/doc/openvpn/sample/sample-config-files/server.conf
and in config/server.conf

# Enable and start OpenVPN server
sudo systemctl enable openvpn-server@server
sudo systemctl start openvpn-server@server
sudo systemctl status openvpn-server@server
"""
### Troubleshooting
Check logs: journalctl -u openvpn-server@server

Verify certificates are in correct locations

Ensure IP forwarding is enabled

   
