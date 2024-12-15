---
description: Just a page with useful Linux configuration commands
---

# Linux Commands





## User control

```bash
## creates the user
adduser testuser

## adding the user to sudo group
usermod -aG sudo testuser

## Switching user
su - testuser

## Change password
passwd username
```

## Services

```bash
# Starting service
sudo systemctl start docker

# Start a service on boot
systemctl enable docker

# Show enabled services
systemctl list-unit-files --type=service --state=enabled

# Starting service
sudo service docker start

# List running services
sudo service --status-all
```



## Networking

### IP addressing

```bash
## Only show eth0 interface 
ip -br a
ip a show eth0
ip a list eth0
ip a show dev eth0
ifconfig
ifconfig eth0
 
## Only show running interfaces - mac address
ip -br link
ip link show up

## Assign IP Address
sudo ifconfig eth0 192.168.1.2 netmask 255.255.255.0
ip a add 192.168.1.200/255.255.255.0 dev eth0
ip addr add broadcast 172.20.10.255 dev eth0

## Deleting an IP address from an interface
ip a del 192.168.1.200/24 dev eth0

## Flushing all IPs from a range
ip -s -s a f to 192.168.2.0/24

## Changing state
ip link set dev eth1 down/up
ifconfig eth0 down

## Route table
ip r list

## Add route for traffic matching range to gateway
ip route add 192.168.1.0/24 via 192.168.1.254

## Add route for traffic matching via interace
ip route add 192.168.1.0/24 dev eth0

## Deleting default route
ip route del default

## Deleteing specific route
ip route del 192.168.1.0/24 dev eth0

## MTU
ip link set mtu 9000 dev eth0

## txqueuelen
ip link set txqueuelen 10000 dev eth0

## ARP cache
ip n show
arp -a

## Setting default gateway
sudo route add default gw 192.168.1.1 eth0

## DNS resolver at
/etc/resolv.conf

## quick add a dns
echo "nameserver 8.8.8.8" | sudo tee -a /etc/resolv.conf > /dev/null

## Persisting DNS changes on reboot - below file with example config
/etc/network/interfaces

auto eth0
iface eth0 inet static
  address 192.168.1.2
  netmask 255.255.255.0
  gateway 192.168.1.1
  dns-nameservers 8.8.8.8 8.8.4.4
  
### IPtables
## List stuff
iptables -L

## list specific chain
iptables -L DOCKER
iptables -L DOCKER --line-numbers

## Inserting a rule
iptables -I DOCKER-USER 1 -p tcp --dport 81 -s 210.246.24.77 -j ACCEPT

## Appending a rule
iptables -A DOCKER-USER -p tcp --dport 81 -s 210.246.24.77 -j ACCEPT

```

## File space and storage

```bash
## Looking at disk usage
df -h 

## Looking for space hogging - slow
sudo du -sh /*
sudo du -sh /* | sort -hr

## Then start working way in from there

## Finding top largest files in a folder
find /mnt/home/flobeey -type f -exec du -ah {} + | sort -rh | head -n 10

# Deleting files when too many arguements
find . -name "*.pdf" -print0 | xargs -0 rm
find . -maxdepth 1 -name "*.pdf" -print0 | xargs -0 rm
find . -name "*.pdf" -delete

## Clearing old kernal files
uname -r
dpkg-query --show 'linux-modules-*'
dpkg-query --show 'linux-modules-*' | cut -f1
sudo apt remove $(dpkg-query --show 'linux-modules-*' | cut -f1 | grep -v "$(uname -r)")

## Journal file
journalctl --disk-usage

## Set threshold of journal to 500M
sudo journalctl --vacuum-size=500M

## Finding package associated with file
dpkg -S /filepath
```





