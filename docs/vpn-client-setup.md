# VPN Client Setup (Windows)

##  Requirements
- OpenVPN Client for Windows (https://openvpn.net)
- Files to transfer:
  - `ca.crt`
  - `ta.key`
  - `client1.crt`
  - `client1.key`

##  File Transfer
Use WinSCP or `scp`:
```bash
scp ca.crt client1.crt client1.key ta.key user@windows_ip:/path
```
