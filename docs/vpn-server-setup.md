# VPN Server Setup Script (Fedora)

# Update and install packages
sudo dnf update -y
sudo dnf install -y openvpn easy-rsa

# Initialize Easy-RSA
sudo cp -r /usr/share/easy-rsa /etc/openvpn/
cd /etc/openvpn/easy-rsa
sudo ./easyrsa init-pki
sudo ./easyrsa build-ca nopass
sudo ./easyrsa gen-req server nopass
sudo ./easyrsa sign-req server server
sudo ./easyrsa gen-dh
sudo openvpn --genkey --secret ta.key

# Copy certificates and keys to OpenVPN directory
sudo cp pki/ca.crt pki/private/server.key pki/issued/server.crt ta.key pki/dh.pem /etc/openvpn/server/

# Enable IP forwarding
sudo sed -i 's/#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/' /etc/sysctl.conf
sudo sysctl -p

# Enable and start OpenVPN service
sudo systemctl enable openvpn-server@server
sudo systemctl start openvpn-server@server
sudo systemctl status openvpn-server@server
