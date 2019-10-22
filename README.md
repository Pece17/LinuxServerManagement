# Linux Server Management

Linux notes and instructions for a course about Linux server management in Haaga-Helia University of Applied Sciences


## Author

Pekka Hämäläinen


# Rovius CP

Open VDI from address https://vdi.haaga-helia.fi if you are working outside of Haaga-Helia lab environment, and navigate to Rovius CP in address https://vdi-lab.cp.haaga-helia.fi/client/

Type your ```Username```, ```Password```, and ```Domain```, after which press ```Login``` - ```Username``` and ```Password``` are the same as in your student ID, and ```Domain``` is ```ITLAB```

Change ```Project``` to ```ICT4TN 022-3006``` from upper left corner

Navigate to ```Instances``` and press ```Add Instance```

Configure the following settings for the first instance that will be a graphical RDP (Remote Desktop Protocol) Linux workstation

1. Choose ```HH01```, ```Template```, and press ```Next```
2. Choose ```BUSTERxrpAdminws1AH``` and press ```Next```
3. Choose ```Linux Normal``` and press ```Next```
4. Choose ```No thanks``` and press ```Next```
5. Press ```Next```
6. Change ```Name``` to ```pekanverkko```, keep ```Network Offering``` as ```DefaultIslatedNetworkOfferingWithSourceNatService```, and press ```Next```
7. Press ```Next```
8. Change ```Name``` to ```bustergraafinen```, choose ```Standard (US) keyboard```, and press ```Launch VM```

Configure the following settings for the second instance that will be an Ubuntu server used via SSH (Secure Shell)

1. Choose ```HH01```,  ```Template```, and press ```Next```
2. Press ```Community``` tab, choose ```saltedLabExamUBU1804srv_HA2019s```, and press ```Next```
3. Choose ```Linux Normal``` and press ```Next```
4. Choose ```No thanks``` and press ```Next```
5. Press ```Next```
6. Choose previously made ```pekanverkko``` under ```Networks``` and press ```Next```
7. Press ```Next```
8. Change ```Name``` to ```ubuntussh```, choose ```Standard (US) keyboard```, and press ```Launch VM```

Navigate to ```Network```, choose ```pekanverkko```, press ```Egress rules```, and configure the following parameters

- Add address ```10.208.0.0/24``` to ```Source CIDR```
- Add address ```0.0.0.0/0``` to ```Destination CIDR```
- Change ```Protocol``` to ```All```
- Leave ```Start Port``` and ```End Port``` empty
- Press ```Add```

Navigate back to ```Network```, choose ```pekanverkko```, press ```View IP Addresses```, choose ```10.207.5.193 [Source NAT]```, press ```Configuration```, choose ```Firewall```, and configure the following parameters

- Add address ```0.0.0.0/0``` to ```Source CIDR```
- Change ```Protocol``` to ```TCP```
- Add ```1001``` to ```Start Port```
- Add ```1009``` to ```End Port```
- Press ```Add```

Lastly, navigate to ```Network```, choose ```pekanverkko```, press ```View IP Addresses```, choose ```10.207.5.193 [Source NAT]```, press ```Configuration```, choose ```Port Forwarding```, and configure the following parameters for three different ports


- For the first port, add ```22``` ```22``` to ```Private Port``` and ```1001``` ```1001``` to ```Public Port```, change ```Protocol``` to ```TCP```, press ```Add```, select ```ubuntussh```, and press ```Apply```
- For the second port, add ```22``` ```22``` to ```Private Port``` and ```1002``` ```1002``` to ```Public Port```, change ```Protocol``` to ```TCP```, press ```Add```, select ```bustergraafinen```, and press ```Apply```
- Fort the third port, add ```3389``` ```3389``` to ```Private Port``` and ```1003``` ```1003``` to ```Public Port```, change ```Protocol``` to ```TCP```, press ```Add```, select ```bustergraafinen```, and press ```Apply```

The instances ```bustergraafinen``` and ```ubuntussh``` should be working correctly now, as well as the network ```pekanverkko```


# Remote connection to the instances

First, we will establish a remote desktop connection to the RDP instance ```bustergraafinen``` using the following steps

- Open ```Remote Desktop Connection``` application and press ```Show Options```
- Type ```10.207.5.193:1003``` to ```Computer:``` and press ```Connect```
- A warning prompt will pop up for which you need to press ```Yes```
- In the login window, keep ```Session``` as ```Xorg```, type ```oppilas``` for ```username```, type ```salainen``` for ```password```, and press ```OK```

Next, we will establish an SSH connection to the Ubuntu server instance ```ubuntussh``` using the terminal in ```bustergraafinen```

- Open the terminal with ```Ctrl + Alt + T```
- Establish an SSH connection ```ssh -I oppilas -p 1001 10.207.5.193```



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

```
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
```

```
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"
```

```
sudo nano /etc/apt/sources.list
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

```
sudo apt-get update
```

```
sudo apt -y install docker-ce docker-ce-cli containerd.io
```

https://computingforgeeks.com/how-to-install-latest-docker-compose-on-linux/

sudo apt install -y curl

```
curl -s https://api.github.com/repos/docker/compose/releases/latest \
  | grep browser_download_url \
  | grep docker-compose-Linux-x86_64 \
  | cut -d '"' -f 4 \
  | wget -qi -
```

chmod +x docker-compose-Linux-x86_64

sudo mv docker-compose-Linux-x86_64 /usr/local/bin/docker-compose

docker-compose version

sudo curl -L https://raw.githubusercontent.com/docker/compose/master/contrib/completion/bash/docker-compose -o /etc/bash_completion.d/docker-compose

```
source /etc/bash_completion.d/docker-compose
```

```
sudo nano docker-compose.yml
```

Copy following text

```
version: '3'  
services:
  web:
    image: nginx:latest
    ports:
     - "8080:80"
    links:
     - php
  php:
    image: php:7-fpm
```

cat docker-compose.yml

docker-compose --help|less

sudo docker-compose -f ./docker-compose.yml up -d

wget localhost:8080

ct index.html

sudo docker-compose -f ./docker-compose.yml down

# Docker


# Salt


# WordPress
