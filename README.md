# Linux Server Management

Linux notes and instructions for a course about Linux server management in Haaga-Helia University of Applied Sciences


## Author

Pekka Hämäläinen


# Linux Basics

- Linux USB stick
- Xubuntu initialization
- Testing and installing example programs
- Causing log entries and reading them 
- Dream apt-get command
- 

# Rovious CP

Navigate to Rovius CP in address https://vdi-lab.cp.haaga-helia.fi/client/, or https://vdi.haaga-helia.fi” if you are outside school internet

Type your username, password, domain, and press login

Change project to ICT4TN 022-3006 Pekka Hämäläinen

Navigate to Instances and Add Instance

Configure the following setup to the first instance
- Choose HH01 and Template and press Next
- Choose BUSTERxrpAdminws1AH and press Next
- Choose Linux Normal and press Next
- Choose No thanks and press Next
- Press Next
- Change Name to pekanverkko and keep Network Offering as DefaultIslatedNetworkOfferingWithSourceNatService and press Next
- Press Next
- Change Name to bustergraafinen and choose Standard (US) keyboard and press Launch VM

Add Instance again and configure the following setup to the second instance
- Choose HH01 and Template and press Next
- Press Community tab and choose saltedLabExamUBU1804srv_HA2019s
- Choose Linux Normal and press Next
- Choose No thanks and press Next
- Press Next
- Choose pekanverkko and press Next
- Press Next
- Change Name to ubuntussh and choose Standard (US) keyboard and press Launch VM

Navigate to Network and choose pekanverkko
- View IP addresses
- Choose 10.207.5.193
- Choose Configuration tab
- Source CIDR 0.0.0.0/0, Protocol TCP, Start Port 1001, and End port 1009, and Add

Navigate to Network and choose pekanverkko
- View IP addresses
- Choose 10.207.5.193
- Choose Configuration Tab
- Choose Port Forwarding
- Add Port 22-22 1001-1001 TCP active add to ubuntussh
- Add Port 22-22 1002-1002 TCP active add to bustergraafinen
- Add Port 3389-3389 1003-1003 TCP Active add to bustergraafinen

Open Remote Desktop Connection to bustergraafinen
- Show Options 
- Computer 10.207.5.193:1003
- Connect and press Yes
- Username oppilas and enter password

Open PuTTY to ubuntussh
- Host Name 10.207.5.193 and Port 1001
- Press Yes
- Username oppilas and enter password
- exit

https://www.cyberciti.biz/faq/ubuntu-18-04-lts-change-hostname-permanently/

Open terminal in bustergraafinen and establish SSH connection to ubuntussh
- ssh -I oppilas -p 1001 10.207.5.193 command to open ubuntussh and login
- sudo hostnamectl set-hostname salt_hamalainen and give password
- hostname
- sudo nano /etc/hosts

Change the new name salt_hamalainen in place of saltminionweb1

```
27.0.0.1 localhost
127.0.1.1 salt_hamalainen
10.207.5.78 salt
10.207.5.78 saltgrandmaster1

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

Exit and save Ctrl + X + Y + Enter

sudo reboot

ssh -I oppilas -p 1001 10.207.5.193

http://10.207.5.78/

Install Salt minion

```
sudo apt-get -y install salt-minion
```

```
SALTID=$(getent passwd $USER| cut -d ':' -f 5 | cut -d ',' -f 1|tr -c '[a-zA-Z]' '_'; echo -n "_$(hostname)_"; date +'%H%M%S') echo -e "master: 10.207.5.78\nid: $SALTID"|sudo tee /etc/salt/minion
```

```
sudo systemctl restart salt-minion
```

Exit ubuntussh

```
exit
```

https://computingforgeeks.com/install-docker-and-docker-compose-on-debian-10-buster/

Continue using bustergraafinen terminal

```
sudo apt update
sudo apt -y install apt-transport-https ca-certificates curl gnupg2 software-properties-common
```

curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -

```
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"
```

```
/etc/apt/sources.list
```

Copy line ```deb [arch=amd64] https://download.docker.com/linux/debian buster stable``` inside the sources.list file

```
# 

# deb cdrom:[Debian GNU/Linux 10.0.0 _Buster_ - Official amd64 DVD Binary-1 20190706-10:24]/$

# deb cdrom:[Debian GNU/Linux 10.0.0 _Buster_ - Official amd64 DVD Binary-1 20190706-10:24]/$

deb http://www.nic.funet.fi/debian/ buster main
deb-src http://www.nic.funet.fi/debian/ buster main

deb http://security.debian.org/debian-security buster/updates main contrib non-free
deb-src http://security.debian.org/debian-security buster/updates main contrib non-free

# buster-updates, previously known as 'volatile'
deb http://www.nic.funet.fi/debian/ buster-updates main contrib non-free
deb-src http://www.nic.funet.fi/debian/ buster-updates main contrib non-free
deb [arch=amd64] https://download.docker.com/linux/debian buster stable

# This system was installed using small removable media
# (e.g. netinst, live or single CD). The matching "deb cdrom"
# entries were disabled at the end of the installation process.
# For information about how to configure apt package sources,
deb [arch=amd64] https://download.docker.com/linux/debian buster stable
# deb-src [arch=amd64] https://download.docker.com/linux/debian buster stable
# see the sources.list(5) manual.
```

# Docker


# Salt


# WordPress
