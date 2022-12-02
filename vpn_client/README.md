** Client OpenVPN Installation **
```
sudo apt install openvpn
```
   
start the service
```
sudo systemctl start openvpn@client
```
   
check the client status
```
sudo systemctl status openvpn@client
```
   
Routing configuration   
   
Edit /etc/sysctl.conf and uncomment the following line to enable IP forwarding.
```
#net.ipv4.ip_forward=1
```
   
Then reload sysctl.
```
sudo sysctl -p /etc/sysctl.conf
```
   
IPTABLES rules   
add the next rules
```
sudo iptables -t nat -A POSTROUTING -o tun0 -j MASQUERADE
sudo iptables -A FORWARD -i eth0 -o tun0 -j ACCEPT
sudo iptables -A FORWARD -i tun0 -o etho -m state --state RELATED,ESTABLISHED -j ACCEPT
```
   


