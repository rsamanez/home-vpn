## VPN Server configuration 
   
Openvpn installation
```
sudo apt install openvpn easy-rsa
```
   
### Certificate Authority Setup
```
sudo make-cadir /etc/openvpn/easy-rsa
```
   
As root user change to the newly created directory **/etc/openvpn/easy-rsa** and run:
```
./easyrsa init-pki
./easyrsa build-ca
```
   
### Server Keys and Certificates
Next, we will generate a key pair for the server:
```
./easyrsa gen-req myservername nopass
```
Diffie Hellman parameters must be generated for the OpenVPN server. The following will place them in pki/dh.pem.
```
./easyrsa gen-dh
```
   
And finally a certificate for the server:
```
./easyrsa gen-req myservername nopass
./easyrsa sign-req server myservername
```
   
All certificates and keys have been generated in subdirectories. Common practice is to copy them to /etc/openvpn/:
```
cp pki/dh.pem pki/ca.crt pki/issued/myservername.crt pki/private/myservername.key /etc/openvpn/
```
Generate tls-auth like:
```
cd /etc/openvpn
sudo openvpn --genkey --secret ta.key
```
   
### Generate Clients Certificates
The VPN client will also need a certificate to authenticate itself to the server. Usually you create a different certificate for each client.
   
To create the certificate, enter the following in a terminal while being user root:
```
./easyrsa gen-req vpnClient1 nopass
./easyrsa sign-req client vpnClient1
```
copy the following files to the client using a secure method:
```
pki/ca.crt
pki/issued/vpnClient1.crt
pki/private/vpnClient1.key
```
As the client certificates and keys are only required on the client machine, you can remove them from the server.
   
### Simple Server Configuration
   
create a file **/etc/openvpn/server.conf**  
make sure the following lines are pointing to the certificates and keys you created in the section above.
```
ca ca.crt
cert myservername.crt
key myservername.key
dh dh.pem
```
Complete **server.conf** file
```
port 1194
proto udp
dev tun
ca ca.crt
cert myservername.crt
key myservername.key 
dh dh.pem
server 10.8.0.0 255.255.255.0
ifconfig-pool-persist /var/log/openvpn/ipp.txt

push "redirect-gateway def1 bypass-dhcp"
push "dhcp-option DNS 8.8.8.8"
push "dhcp-option DNS 208.67.220.220"

keepalive 10 120
tls-auth ta.key 0 # This file is secret
cipher AES-256-CBC
persist-key
persist-tun
status /var/log/openvpn/openvpn-status.log
verb 3
explicit-exit-notify 1
```
complete file with comments: **server.conf**
   
Edit /etc/sysctl.conf and uncomment the following line to enable IP forwarding.
```
#net.ipv4.ip_forward=1
```
Then reload sysctl.
```
sudo sysctl -p /etc/sysctl.conf
```
   
start the service
```
sudo systemctl start openvpn@myserver
```
   
   
IPDABLES rules
```
iptables -A FORWARD --in-interface tun0 -j ACCEPT
iptables --table nat -A POSTROUTING --out-interface eth0 -j MASQUERADE
```

