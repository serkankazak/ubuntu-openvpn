### ubuntu/debian/linux openvpn setup (redirect all traffic through tun0)

#### tested on raspberry with ethernet and wifi

```
sudo apt-get install -y openvpn
sudo openvpn --config client.ovpn

sudo nano /etc/dhcpcd.conf # static domain_name_servers=8.8.8.8

sudo service dhcpcd restart
cat /etc/resolv.conf

sudo sysctl -w net.ipv4.ip_forward=1

sudo iptables -F
sudo iptables -t nat -F
sudo iptables -t mangle -F
sudo iptables -X

sudo iptables -t nat -A POSTROUTING -o tun0 -j MASQUERADE

sudo iptables -A FORWARD -i tun0 -o tun0 -j ACCEPT

sudo iptables -A FORWARD -i eth0 -o tun0 -j ACCEPT
sudo iptables -A FORWARD -i tun0 -o eth0 -m state --state ESTABLISHED,RELATED -j ACCEPT

sudo iptables -A FORWARD -i wlan0 -o tun0 -j ACCEPT
sudo iptables -A FORWARD -i tun0 -o wlan0 -m state --state ESTABLISHED,RELATED -j ACCEPT

sudo ip route add 8.8.8.8 via 192.168.0.1 dev eth0
sudo ip route add <public_ip_of_vpn_server> via 192.168.0.1 dev eth0
sudo ip route add 192.168.1.0/24 via 192.168.1.1 dev wlan0
sudo route del -net 0.0.0.0 gw 192.168.0.1 netmask 0.0.0.0 dev eth0
sudo route del -net 0.0.0.0 gw 192.168.1.1 netmask 0.0.0.0 dev wlan0
sudo ip route add default via <vpn_gateway> dev tun0

sudo route -n

curl ifconfig.me
```

#### to revert:

```
sudo killall openvpn
sudo sysctl -w net.ipv4.ip_forward=0
sudo route del -net 0.0.0.0 gw 192.168.0.1 netmask 0.0.0.0 dev eth0
sudo route del -net 0.0.0.0 gw 192.168.1.1 netmask 0.0.0.0 dev wlan0
sudo ip route add default via 192.168.0.1 dev eth0
sudo ip route add default via 192.168.1.1 dev wlan0
sudo ip route del 8.8.8.8 via 192.168.0.1 dev eth0
sudo ip route del <public_ip_of_vpn_server> via 192.168.0.1 dev eth0
ip route show
sudo iptables -F
sudo iptables -t nat -F
sudo iptables -t mangle -F
sudo iptables -X
sudo service dhcpcd restart
cat /etc/resolv.conf
sudo route -n
curl ifconfig.me
```
