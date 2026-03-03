# 🕸️ Networking

> Frequently used networking commands for diagnostics, configuration, and security.

[← Back to Home](../README.md)

---

## 📋 Table of Contents

- [Network Interfaces](#network-interfaces)
- [IP & Routing](#ip--routing)
- [DNS](#dns)
- [Connectivity Testing](#connectivity-testing)
- [Port & Socket Inspection](#port--socket-inspection)
- [Firewall (UFW & iptables)](#firewall-ufw--iptables)
- [Packet Capture](#packet-capture)
- [Network Performance](#network-performance)
- [HTTP Debugging](#http-debugging)
- [VPN & Tunnels](#vpn--tunnels)

---

## Network Interfaces

```bash
# Show all interfaces
ip link show
ip addr show
ip -br addr show             # brief format

# Show specific interface
ip addr show eth0
ifconfig eth0                # legacy

# Enable / disable interface
sudo ip link set eth0 up
sudo ip link set eth0 down

# Set IP address
sudo ip addr add 192.168.1.100/24 dev eth0
sudo ip addr del 192.168.1.100/24 dev eth0

# Show interface statistics
ip -s link show eth0
cat /proc/net/dev

# Rename interface
sudo ip link set eth0 name myinterface

# MAC address
ip link show eth0 | grep ether
# Change MAC (temporarily)
sudo ip link set eth0 address aa:bb:cc:dd:ee:ff
```

---

## IP & Routing

```bash
# Show routing table
ip route show
ip route show table all
route -n                     # legacy

# Add / remove routes
sudo ip route add 10.0.0.0/8 via 192.168.1.1
sudo ip route add 10.0.0.0/8 dev eth0
sudo ip route del 10.0.0.0/8

# Default gateway
sudo ip route add default via 192.168.1.1
sudo ip route del default

# Show ARP table
arp -n
ip neigh show

# Add ARP entry
arp -s 192.168.1.50 aa:bb:cc:dd:ee:ff

# IP forwarding
cat /proc/sys/net/ipv4/ip_forward
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
# Persist:
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

---

## DNS

```bash
# Basic DNS lookup
nslookup google.com
nslookup google.com 8.8.8.8  # use specific DNS server

# dig — detailed DNS queries
dig google.com
dig google.com A             # A record
dig google.com MX            # Mail exchange
dig google.com NS            # Name servers
dig google.com TXT           # TXT records
dig google.com AAAA          # IPv6
dig google.com ANY           # all records
dig @8.8.8.8 google.com      # use specific resolver
dig +short google.com        # short output
dig +trace google.com        # trace full resolution path

# Reverse DNS lookup
dig -x 8.8.8.8
host 8.8.8.8

# host command
host google.com
host google.com 8.8.8.8

# Check local DNS config
cat /etc/resolv.conf
cat /etc/hosts

# Flush DNS cache
sudo systemd-resolve --flush-caches           # systemd
sudo resolvectl flush-caches                  # newer systemd
sudo service nscd restart                     # nscd

# Test DNS resolution time
time dig google.com +stats
```

---

## Connectivity Testing

```bash
# Ping
ping google.com
ping -c 4 google.com            # 4 packets
ping -i 0.2 google.com          # 200ms interval
ping -s 1400 google.com         # 1400 byte packets
ping6 ipv6.google.com           # IPv6

# Traceroute
traceroute google.com
traceroute -n google.com         # no DNS resolution
traceroute -I google.com         # use ICMP
traceroute -T -p 80 google.com   # use TCP port 80

# MTR (combined ping + traceroute)
mtr google.com
mtr --report google.com          # report mode
mtr -n --report google.com       # numeric

# Test TCP connection
telnet google.com 80
nc -zv google.com 80             # netcat
nc -zv google.com 80 443         # multiple ports

# Test if port is open
timeout 3 bash -c "</dev/tcp/google.com/80" && echo "Open" || echo "Closed"

# Test UDP
nc -zuv google.com 53
```

---

## Port & Socket Inspection

```bash
# List listening ports
ss -tulnp
ss -tulnp | grep :80
netstat -tulnp               # legacy (deprecated)

# All established connections
ss -tnp
ss -anp

# Specific port
ss -tulnp sport = :8080
lsof -i :8080
lsof -i tcp:8080

# Process using a port
fuser 8080/tcp               # prints PID
fuser -k 8080/tcp            # kill process on port

# Socket statistics
ss -s                        # summary

# All sockets
ss -a                        # all
ss -t                        # TCP
ss -u                        # UDP
ss -x                        # Unix sockets
```

---

## Firewall (UFW & iptables)

### UFW (Uncomplicated Firewall)

```bash
# Status
sudo ufw status
sudo ufw status verbose
sudo ufw status numbered

# Enable / disable
sudo ufw enable
sudo ufw disable

# Allow / deny rules
sudo ufw allow 22
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow from 192.168.1.0/24 to any port 22
sudo ufw deny 3306
sudo ufw deny from 10.0.0.5

# Delete rule
sudo ufw delete allow 22
sudo ufw delete 3             # delete by rule number

# Reset all rules
sudo ufw reset

# Application profiles
sudo ufw app list
sudo ufw allow 'Nginx Full'   # 80 + 443
sudo ufw allow 'OpenSSH'

# Reload
sudo ufw reload
```

### iptables

```bash
# List rules
sudo iptables -L -n -v
sudo iptables -L -n -v --line-numbers
sudo iptables -t nat -L -n -v

# Allow / drop
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -P INPUT DROP       # default drop for INPUT

# Delete a rule
sudo iptables -D INPUT -p tcp --dport 80 -j ACCEPT

# NAT — Port Forwarding
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 192.168.1.10:8080
sudo iptables -t nat -A POSTROUTING -j MASQUERADE

# Save / restore
sudo iptables-save > /etc/iptables/rules.v4
sudo iptables-restore < /etc/iptables/rules.v4

# nftables (modern replacement)
sudo nft list ruleset
sudo nft add rule ip filter input tcp dport 22 accept
```

---

## Packet Capture

```bash
# tcpdump
sudo tcpdump -i eth0                       # capture on eth0
sudo tcpdump -i any                        # all interfaces
sudo tcpdump -i eth0 port 80               # filter by port
sudo tcpdump -i eth0 host 192.168.1.100    # filter by host
sudo tcpdump -i eth0 -n                    # no DNS resolution
sudo tcpdump -i eth0 -v                    # verbose
sudo tcpdump -i eth0 -c 100               # capture 100 packets
sudo tcpdump -i eth0 -w capture.pcap       # save to file
sudo tcpdump -r capture.pcap               # read from file

# Filters
sudo tcpdump -i eth0 'tcp port 80'
sudo tcpdump -i eth0 'src 192.168.1.100 and dst port 443'
sudo tcpdump -i eth0 'tcp[13] & 0x02 != 0'    # TCP SYN packets

# Wireshark (GUI)
sudo apt install -y wireshark
wireshark capture.pcap
```

---

## Network Performance

```bash
# iperf3 — bandwidth testing
# Server:
iperf3 -s
iperf3 -s -p 5201

# Client:
iperf3 -c server-ip
iperf3 -c server-ip -t 30          # 30 seconds
iperf3 -c server-ip -u             # UDP test
iperf3 -c server-ip -P 4           # 4 parallel streams

# nload — real-time bandwidth
sudo apt install nload
nload eth0

# nethogs — bandwidth by process
sudo apt install nethogs
sudo nethogs eth0

# iftop — connections by host
sudo apt install iftop
sudo iftop -i eth0

# vnstat — historical network stats
sudo apt install vnstat
vnstat
vnstat -d                           # daily
vnstat -m                           # monthly

# Check network latency
ping -c 100 google.com | tail -1    # summary line
```

---

## HTTP Debugging

```bash
# curl
curl -v https://example.com                  # verbose (headers)
curl -I https://example.com                  # HEAD request
curl -o /dev/null -w "%{http_code}\n" https://example.com
curl -X POST -H "Content-Type: application/json" \
     -d '{"key":"val"}' https://api.example.com
curl -u user:pass https://example.com         # basic auth
curl -H "Authorization: Bearer TOKEN" https://api.example.com
curl --max-time 10 https://example.com        # timeout
curl -L https://example.com                   # follow redirects
curl -k https://self-signed.example.com       # skip SSL verify
curl --resolve example.com:443:1.2.3.4 https://example.com  # custom host

# wget
wget https://example.com/file.tar.gz
wget -q --spider https://example.com          # check if URL exists
wget -r -l 2 https://example.com              # recursive

# httpie (friendlier curl)
pip install httpie
http GET https://example.com
http POST https://api.example.com name=John token:=123

# wrk — HTTP benchmarking
wrk -t4 -c100 -d30s https://example.com       # 4 threads, 100 connections, 30s
```

---

## VPN & Tunnels

```bash
# WireGuard
sudo apt install wireguard

# Generate keys
wg genkey | tee privatekey | wg pubkey > publickey

# Status
sudo wg show
sudo wg show wg0

# Start / stop
sudo wg-quick up wg0
sudo wg-quick down wg0

# SSH Tunnels (already covered in Linux section)
# Local: forward local port to remote service
ssh -L 5432:localhost:5432 user@remote-server   # access remote postgres locally

# Remote: expose local service to remote
ssh -R 8080:localhost:3000 user@remote-server

# OpenVPN
sudo openvpn --config client.ovpn
sudo systemctl start openvpn@client
```

---

[← Back to Home](../README.md)
