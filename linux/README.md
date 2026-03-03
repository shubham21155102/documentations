# 🐧 Linux

> Frequently used Linux commands for system administration and DevOps tasks.

[← Back to Home](../README.md)

---

## 📋 Table of Contents

- [File & Directory Operations](#file--directory-operations)
- [File Permissions](#file-permissions)
- [Searching & Finding Files](#searching--finding-files)
- [Process Management](#process-management)
- [System Information](#system-information)
- [Disk & Storage](#disk--storage)
- [Network Commands](#network-commands)
- [User & Group Management](#user--group-management)
- [Package Management](#package-management)
- [Archiving & Compression](#archiving--compression)
- [Text Processing](#text-processing)
- [SSH & Remote Access](#ssh--remote-access)
- [Crontab](#crontab)
- [Systemd Services](#systemd-services)

---

## File & Directory Operations

```bash
# Navigation
pwd                          # print working directory
cd /var/log                  # change directory
cd ~                         # go to home
cd -                         # go to previous directory
ls -la                       # list with details & hidden files
ls -lh                       # human-readable sizes

# Create
mkdir mydir
mkdir -p /tmp/a/b/c          # create nested dirs
touch file.txt
echo "hello" > file.txt      # create with content
cat > file.txt << EOF
line 1
line 2
EOF

# Copy / move / delete
cp file.txt backup.txt
cp -r mydir/ backup/         # recursive copy
mv file.txt newname.txt
mv file.txt /tmp/
rm file.txt
rm -rf mydir/                # recursive force delete
rmdir emptydir               # remove empty dir

# View files
cat file.txt
less file.txt                # paginated view (q to quit)
head -20 file.txt            # first 20 lines
tail -20 file.txt            # last 20 lines
tail -f /var/log/app.log     # follow log in real time

# Links
ln -s /path/to/file symlink  # symbolic link
ln file.txt hardlink.txt     # hard link

# File info
file file.txt                # detect file type
stat file.txt                # detailed metadata
wc -l file.txt               # count lines
wc -c file.txt               # count bytes
```

---

## File Permissions

```bash
# View permissions
ls -la

# chmod: change permissions
chmod 755 script.sh          # rwxr-xr-x
chmod 644 file.txt           # rw-r--r--
chmod +x script.sh           # add execute for all
chmod -w file.txt            # remove write for all
chmod -R 755 mydir/          # recursive

# Symbolic permission notation:
# u=user, g=group, o=others, a=all
chmod u+x script.sh
chmod go-w file.txt
chmod a=rw file.txt

# chown: change owner
chown user:group file.txt
chown -R ubuntu:ubuntu /app/ # recursive
chown user file.txt          # change user only

# Special permissions
chmod +s script.sh           # setuid
chmod g+s mydir/             # setgid
chmod +t /tmp/               # sticky bit
chmod 4755 script.sh         # setuid (numeric)

# Default permissions mask
umask                        # view current umask
umask 022                    # set umask
```

| Number | Permission |
|--------|-----------|
| 7 | rwx |
| 6 | rw- |
| 5 | r-x |
| 4 | r-- |
| 0 | --- |

---

## Searching & Finding Files

```bash
# find
find /var/log -name "*.log"
find /home -user ubuntu
find / -type f -size +100M   # files larger than 100MB
find /tmp -mtime +7 -delete  # delete files older than 7 days
find . -name "*.py" -exec grep -l "import os" {} \;

# locate (fast, uses database)
sudo updatedb
locate myfile.txt

# which / whereis
which python3
whereis nginx

# grep — search in files
grep "error" /var/log/app.log
grep -r "TODO" ./src/             # recursive
grep -i "error" file.txt          # case-insensitive
grep -n "error" file.txt          # show line numbers
grep -v "debug" file.txt          # invert match (exclude)
grep -l "pattern" *.txt           # list matching files
grep -c "error" file.txt          # count matches
grep -A 3 -B 3 "error" file.txt   # context lines

# ripgrep (faster grep)
rg "error" /var/log/
rg -i "TODO" ./src/
```

---

## Process Management

```bash
# View processes
ps aux                       # all processes
ps aux | grep nginx
top                          # real-time process monitor
htop                         # improved top
pgrep nginx                  # get PID by name
pidof nginx

# Kill processes
kill 1234                    # send SIGTERM
kill -9 1234                 # send SIGKILL (force)
killall nginx                # kill by name
pkill nginx                  # kill by pattern
kill -HUP 1234               # reload (SIGHUP)

# Background / foreground
./long-script.sh &           # run in background
jobs                         # list background jobs
fg %1                        # bring job 1 to foreground
bg %1                        # send job 1 to background
nohup ./script.sh &          # run immune to hangup
disown %1                    # detach job from shell

# Process priority
nice -n 10 ./script.sh       # start with lower priority
renice -n -5 -p 1234         # change priority of running process

# strace / lsof
strace -p 1234               # trace system calls
lsof -p 1234                 # files opened by process
lsof -i :8080                # process using port 8080
```

---

## System Information

```bash
# OS info
uname -a                     # kernel info
cat /etc/os-release          # distribution info
hostnamectl                  # hostname & OS details

# CPU
lscpu
cat /proc/cpuinfo
nproc                        # number of processors

# Memory
free -h                      # memory usage
cat /proc/meminfo
vmstat 1                     # virtual memory stats

# Disk
df -h                        # disk usage
lsblk                        # block devices
fdisk -l                     # partition info (root)
mount                        # mounted filesystems

# Uptime & load
uptime
w                            # who is logged in + load
last                         # login history

# System logs
journalctl -xe               # recent journal entries
journalctl -u nginx          # logs for a service
journalctl --since "1 hour ago"
dmesg                        # kernel ring buffer
tail -f /var/log/syslog
```

---

## Disk & Storage

```bash
# Disk usage
df -h                        # filesystem usage
du -sh /var/log/             # directory size
du -sh * | sort -rh | head   # largest items in current dir
ncdu /var                    # interactive disk usage

# Mount / unmount
mount /dev/sdb1 /mnt/data
umount /mnt/data
mount | grep sdb             # show mount info

# Create filesystem
mkfs.ext4 /dev/sdb1
mkfs.xfs /dev/sdb1

# LVM
pvdisplay                    # physical volumes
vgdisplay                    # volume groups
lvdisplay                    # logical volumes
lvextend -L+10G /dev/mapper/vg-lv   # extend LV
resize2fs /dev/mapper/vg-lv         # resize filesystem

# /etc/fstab — persist mounts
# /dev/sdb1  /mnt/data  ext4  defaults  0  2

# Swap
swapon --show
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

---

## Network Commands

```bash
# Interface info
ip addr show
ip addr show eth0
ifconfig                     # deprecated but still common
ip link show

# Routing
ip route show
ip route add 10.0.0.0/8 via 192.168.1.1
route -n                     # legacy

# Connectivity
ping google.com
ping -c 4 google.com
traceroute google.com
mtr google.com               # ping + traceroute

# DNS
nslookup google.com
dig google.com
dig google.com MX
host google.com

# Ports & connections
ss -tulnp                    # listening ports
ss -tulnp | grep :80
netstat -tulnp               # legacy
lsof -i :8080                # process on port

# Firewall (UFW)
sudo ufw status
sudo ufw allow 22
sudo ufw allow 80/tcp
sudo ufw deny 3306
sudo ufw enable
sudo ufw reload

# Firewall (iptables)
iptables -L -n -v
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -j DROP
iptables-save > /etc/iptables/rules.v4

# Curl / wget
curl -I https://example.com                    # headers only
curl -X POST -H "Content-Type: application/json" -d '{"key":"val"}' https://api.example.com
curl -o output.txt https://example.com
wget https://example.com/file.tar.gz
wget -q -O- https://example.com/script.sh | bash
```

---

## User & Group Management

```bash
# Users
adduser username             # interactive
useradd -m -s /bin/bash username
userdel -r username          # remove user + home
passwd username              # change password
id username                  # show UID/GID
who                          # currently logged in users

# Groups
groupadd mygroup
groupdel mygroup
usermod -aG mygroup username  # add user to group
gpasswd -d username mygroup   # remove user from group
groups username               # list user's groups

# Sudo
visudo                       # edit sudoers safely
echo "username ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/username

# Switch user
su - username
sudo -i                      # become root
sudo -u username command
```

---

## Package Management

```bash
# APT (Debian/Ubuntu)
apt update                   # update package list
apt upgrade -y               # upgrade packages
apt install -y nginx git vim
apt remove nginx
apt purge nginx              # remove + config files
apt autoremove               # remove unused deps
apt search nginx
apt show nginx
apt list --installed

# YUM/DNF (RHEL/CentOS/Fedora)
yum update -y
yum install -y nginx
yum remove nginx
dnf install -y nginx
dnf update -y

# Snap
snap install code --classic
snap list
snap remove code

# pip (Python)
pip install requests
pip install -r requirements.txt
pip list
pip freeze > requirements.txt
pip uninstall requests
```

---

## Archiving & Compression

```bash
# tar
tar -czf archive.tar.gz directory/     # create gzip archive
tar -cjf archive.tar.bz2 directory/    # create bzip2 archive
tar -xzf archive.tar.gz                # extract gzip
tar -xzf archive.tar.gz -C /opt/       # extract to dir
tar -tzf archive.tar.gz                # list contents

# zip / unzip
zip -r archive.zip directory/
unzip archive.zip
unzip archive.zip -d /opt/
unzip -l archive.zip                   # list contents

# gzip / gunzip
gzip file.txt                          # creates file.txt.gz
gunzip file.txt.gz

# 7zip
7z a archive.7z directory/
7z x archive.7z
7z l archive.7z
```

---

## Text Processing

```bash
# sed — stream editor
sed 's/old/new/g' file.txt                     # replace all
sed -i 's/old/new/g' file.txt                  # in-place edit
sed -n '5,10p' file.txt                        # print lines 5-10
sed '/^$/d' file.txt                           # delete blank lines
sed 's/[[:space:]]*$//' file.txt               # remove trailing spaces

# awk — text processing
awk '{print $1}' file.txt                      # print first column
awk -F: '{print $1}' /etc/passwd               # use : as delimiter
awk 'NR==5' file.txt                           # print 5th line
awk '$3 > 100 {print $1, $3}' file.txt         # conditional
awk '{sum += $1} END {print sum}' file.txt     # sum first column

# sort / uniq / cut
sort file.txt
sort -r file.txt                               # reverse
sort -n file.txt                               # numeric
sort -k2 file.txt                              # sort by column 2
sort file.txt | uniq                           # remove duplicates
sort file.txt | uniq -c                        # count occurrences
cut -d: -f1 /etc/passwd                        # cut by delimiter
cut -c1-10 file.txt                            # cut by character

# tr
echo "hello" | tr 'a-z' 'A-Z'                 # uppercase
echo "hello world" | tr -d ' '                 # delete spaces

# xargs
cat files.txt | xargs rm
find . -name "*.log" | xargs gzip
echo "file1 file2 file3" | xargs -n1 echo     # one per line
```

---

## SSH & Remote Access

```bash
# Connect
ssh user@host
ssh -p 2222 user@host                          # custom port
ssh -i ~/.ssh/mykey.pem user@host              # identity file

# SSH keys
ssh-keygen -t ed25519 -C "user@example.com"
ssh-keygen -t rsa -b 4096
ssh-copy-id user@host                          # copy public key
cat ~/.ssh/id_ed25519.pub | ssh user@host "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

# SSH config (~/.ssh/config)
# Host myserver
#   HostName 192.168.1.100
#   User ubuntu
#   IdentityFile ~/.ssh/mykey.pem
#   Port 2222

# SCP — secure copy
scp file.txt user@host:/tmp/
scp user@host:/tmp/file.txt ./
scp -r mydir/ user@host:/opt/

# rsync
rsync -avz ./dir/ user@host:/opt/dir/
rsync -avz --delete ./dir/ user@host:/opt/dir/ # delete files not in source
rsync -avz -e "ssh -p 2222" ./dir/ user@host:/opt/

# SSH tunneling
ssh -L 8080:localhost:80 user@host             # local port forward
ssh -R 8080:localhost:80 user@host             # remote port forward
ssh -D 1080 user@host                          # SOCKS proxy

# SSH agent
eval $(ssh-agent)
ssh-add ~/.ssh/mykey.pem
ssh-add -l                                     # list loaded keys
```

---

## Crontab

```bash
# Edit crontab
crontab -e

# List crontab
crontab -l

# Remove crontab
crontab -r

# Crontab syntax:
# ┌───────────── minute (0-59)
# │ ┌───────────── hour (0-23)
# │ │ ┌───────────── day of month (1-31)
# │ │ │ ┌───────────── month (1-12)
# │ │ │ │ ┌───────────── day of week (0-6, 0=Sunday)
# │ │ │ │ │
# * * * * * command

# Examples
0 * * * * /opt/scripts/hourly.sh            # every hour
0 2 * * * /opt/scripts/backup.sh            # daily at 2am
0 2 * * 0 /opt/scripts/weekly.sh            # weekly (Sunday)
0 2 1 * * /opt/scripts/monthly.sh           # monthly (1st)
*/5 * * * * /opt/scripts/check.sh           # every 5 minutes
@reboot /opt/scripts/startup.sh             # at boot
```

---

## Systemd Services

```bash
# Service management
systemctl start nginx
systemctl stop nginx
systemctl restart nginx
systemctl reload nginx            # reload config without restart
systemctl enable nginx            # enable at boot
systemctl disable nginx
systemctl status nginx

# View logs
journalctl -u nginx
journalctl -u nginx -f            # follow
journalctl -u nginx --since today
journalctl -u nginx --since "2024-01-01 00:00:00"

# List services
systemctl list-units --type=service
systemctl list-units --type=service --state=active

# Create a systemd service (/etc/systemd/system/myapp.service):
```

```ini
[Unit]
Description=My Application
After=network.target

[Service]
Type=simple
User=ubuntu
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/python3 app.py
Restart=always
RestartSec=10
Environment=PORT=8080

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable myapp
sudo systemctl start myapp
```

---

[← Back to Home](../README.md)
