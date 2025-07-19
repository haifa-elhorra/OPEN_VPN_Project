# OpenVPN Server Setup for Fedora

Complete guide to set up an OpenVPN server on Fedora Linux.

## Prerequisites
- Fedora server with sudo/root access
- Stable internet connection
- Basic terminal knowledge

## Installation Steps

### 1. Update System and Install Packages
bash
sudo dnf update -y
sudo dnf install -y openvpn easy-rsa

2. Initialize PKI Infrastructure
bash
sudo mkdir -p /etc/openvpn/easy-rsa
sudo cp -r /usr/share/easy-rsa/3/* /etc/openvpn/easy-rsa/
cd /etc/openvpn/easy-rsa
3. Create Certificate Authority
bash
sudo ./easyrsa init-pki
sudo ./easyrsa build-ca nopass
4. Generate Server Certificates
bash
sudo ./easyrsa gen-req server nopass
sudo ./easyrsa sign-req server server
sudo ./easyrsa gen-dh
sudo openvpn --genkey --secret ta.key
5. Copy Certificates
bash
sudo mkdir -p /etc/openvpn/server
sudo cp pki/ca.crt pki/private/server.key pki/issued/server.crt ta.key pki/dh.pem /etc/openvpn/server/
6. Enable IP Forwarding
bash
echo 'net.ipv4.ip_forward=1' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
7. Start OpenVPN Service
bash
sudo systemctl enable --now openvpn-server@server
sudo systemctl status openvpn-server@server
Post-Installation
Configure your server:

bash
sudo nano /etc/openvpn/server/server.conf
Generate client certificates:

bash
cd /etc/openvpn/easy-rsa
sudo ./easyrsa gen-req client1 nopass
sudo ./easyrsa sign-req client client1
Set up firewall rules if needed

Verification
Check service status:

bash
journalctl -u openvpn-server@server -f
Troubleshooting
Service won't start: Check /var/log/messages or journalctl -xe

Connection issues: Verify certificates and ports (default: 1194/udp)

Client can't connect: Ensure proper routing and firewall rules

Note: Remember to configure your server settings in /etc/openvpn/server/server.conf after installation
