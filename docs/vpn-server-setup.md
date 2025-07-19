#!/bin/bash

# OpenVPN Server Auto-Installer for Fedora
# Version 1.0
# Requires root/sudo privileges

# Color codes for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

# Check if running as root
if [ "$(id -u)" -ne 0 ]; then
  echo -e "${RED}This script must be run as root${NC}" >&2
  exit 1
fi

# Header
echo -e "${GREEN}"
echo "##################################################"
echo "#          OpenVPN Server Auto-Installer         #"
echo "#                for Fedora Linux                #"
echo "##################################################"
echo -e "${NC}"

# Step 1: System Update
echo -e "${YELLOW}[1/7] Updating system packages...${NC}"
dnf update -y
if [ $? -ne 0 ]; then
  echo -e "${RED}System update failed${NC}" >&2
  exit 1
fi

# Step 2: Install dependencies
echo -e "${YELLOW}[2/7] Installing OpenVPN and Easy-RSA...${NC}"
dnf install -y openvpn easy-rsa
if [ $? -ne 0 ]; then
  echo -e "${RED}Package installation failed${NC}" >&2
  exit 1
fi

# Step 3: Setup PKI
echo -e "${YELLOW}[3/7] Setting up PKI infrastructure...${NC}"
mkdir -p /etc/openvpn/easy-rsa
cp -r /usr/share/easy-rsa/3/* /etc/openvpn/easy-rsa/
cd /etc/openvpn/easy-rsa || exit

./easyrsa init-pki
echo -e "${YELLOW}Creating Certificate Authority (leave fields blank for defaults)${NC}"
./easyrsa build-ca nopass

# Step 4: Generate server certificates
echo -e "${YELLOW}[4/7] Generating server certificates...${NC}"
./easyrsa gen-req server nopass
./easyrsa sign-req server server
./easyrsa gen-dh
openvpn --genkey --secret ta.key

# Step 5: Copy certificates
echo -e "${YELLOW}[5/7] Copying certificates to OpenVPN directory...${NC}"
mkdir -p /etc/openvpn/server
cp pki/ca.crt pki/private/server.key pki/issued/server.crt ta.key pki/dh.pem /etc/openvpn/server/

# Step 6: Enable IP forwarding
echo -e "${YELLOW}[6/7] Configuring network settings...${NC}"
echo 'net.ipv4.ip_forward=1' >> /etc/sysctl.conf
sysctl -p

# Step 7: Start service
echo -e "${YELLOW}[7/7] Starting OpenVPN service...${NC}"
systemctl enable --now openvpn-server@server
systemctl status openvpn-server@server

# Completion message
echo -e "${GREEN}"
echo "##################################################"
echo "#          Installation Completed!               #"
echo "#                                                #"
echo "# Next steps:                                    #"
echo "# 1. Configure /etc/openvpn/server/server.conf   #"
echo "# 2. Generate client certificates                #"
echo "# 3. Configure firewall rules                    #"
echo "##################################################"
echo -e "${NC}"
