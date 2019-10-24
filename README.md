# Linux Server Management

Linux notes and instructions for a course about Linux server management in Haaga-Helia University of Applied Sciences


# Author

Pekka Hämäläinen


# Initializing Rovius CP

Open VDI from address https://vdi.haaga-helia.fi if you are working outside of Haaga-Helia lab environment, and navigate to Rovius CP in address https://vdi-lab.cp.haaga-helia.fi/client/

Type your ```Username```, ```Password```, and ```Domain```, after which press ```Login``` - ```Username``` and ```Password``` are the same as in your student ID, and ```Domain``` is ```ITLAB```

Change ```Project``` to ```ICT4TN 022-3006``` from upper left corner

Navigate to ```Instances``` and press ```Add Instance```

Configure the following settings for the first instance that will be a graphical RDP (Remote Desktop Protocol) Linux workstation that is running Debian 10 Buster operating system

1. Choose ```HH01```, ```Template```, and press ```Next```
2. Choose ```BUSTERxrpAdminws1AH``` and press ```Next```
3. Choose ```Linux Normal``` and press ```Next```
4. Choose ```No thanks``` and press ```Next```
5. Press ```Next```
6. Change ```Name``` to ```pekanverkko```, keep ```Network Offering``` as ```DefaultIslatedNetworkOfferingWithSourceNatService```, and press ```Next```
7. Press ```Next```
8. Change ```Name``` to ```bustergraafinen```, choose ```Standard (US) keyboard```, and press ```Launch VM```

Configure the following settings for the second instance that will be an Ubuntu server used via SSH (Secure Shell) that is running Ubuntu 18.04.3 LTS (Long Term Support) operating system

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

The instances ```bustergraafinen``` and ```ubuntussh``` are be now working correctly, as well as the network ```pekanverkko```


# Remote connections to the instances

First, we will establish a remote desktop connection to the RDP  instance ```bustergraafinen``` using the following steps

- Open ```Remote Desktop Connection``` application and press ```Show Options```
- Type ```10.207.5.193:1003``` to ```Computer:``` and press ```Connect```
- A warning prompt will pop up for which you need to press ```Yes```
- In the login window, keep ```Session``` as ```Xorg```, enter ```oppilas``` for ```username```, enter ```salainen``` for ```password```, and press ```OK```

Next, we will establish an SSH connection to the Ubuntu server instance ```ubuntussh``` using the terminal in ```bustergraafinen```

Open the terminal with ```Ctrl + Alt + T```

Establish an SSH connection to ```ubuntussh```

```
ssh -I oppilas -p 1001 10.207.5.193
```

After giving the correct password, you should be in ```ubuntussh``` terminal

Exit back to local ```bustergraafinen``` terminal

```
exit
```

Alternatively, you can use ```PuTTY``` to establish an SSH connection to ```ubuntussh``` using the following steps

- Open ```PuTTY``` application and select ```Session```
- Type ```10.207.5.193``` to ```Host Name (or IP address)``` and ```1001``` to ```Port```, after which press ```Open```
- Type ```oppilas``` in ```login as:``` prompt and ```salainen``` to ```oppilas@10.207.5.193's password```, after which the terminal to ```ubuntussh``` should unlock

Remote connections to ```bustergraafinen``` and ```ubuntussh``` are now working correctly


# Changing the Ubuntu server hostname permanently

Navigate to address http://10.207.5.78/ first and then open the link from there to https://www.cyberciti.biz/faq/ubuntu-18-04-lts-change-hostname-permanently/ to view the instructions for changing the ```ubuntussh``` Ubuntu server hostname permanently

Open the terminal in ```bustergraafinen``` and establish an SSH connection to ```ubuntussh```

```
ssh -I oppilas -p 1001 10.207.5.193
```

Replace the old hostname ```saltminionweb1``` with ```salt_hamalainen``` - the new hostname is derived from your surname, but remember to replace all Scandinavian letters with English letters

```
sudo hostnamectl set-hostname salt_hamalainen
```

Check the new hostname

```
hostname
```

Navigate inside ```/etc/hosts``` file for editing

```
sudo nano /etc/hosts
```

Replace the old hostname ```saltminionweb1``` after the line ```127.0.1.1``` with the new hostname ```salt_hamalainen```

```
127.0.0.1 localhost
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

Exit and save with ```Ctrl + X + Y + Enter```

Reboot the system

```
sudo reboot
```

The hostname is now changed permanently on ```ubuntussh```


# Installing Salt Minion

Navigate to address http://10.207.5.78/ to view the instructions for installing Salt Minion

Open the terminal in ```bustergraafinen``` and establish an SSH connection to ```ubuntussh```

```
ssh -I oppilas -p 1001 10.207.5.193
```

Install Salt Minion

```
sudo apt-get -y install salt-minion
```

Give the following command - note that it is a single command, even though it might be printed on two lines

```
SALTID=$(getent passwd $USER| cut -d ':' -f 5 | cut -d ',' -f 1|tr -c '[a-zA-Z]' '_'; echo -n "_$(hostname)_"; date +'%H%M%S') echo -e "master: 10.207.5.78\nid: $SALTID"|sudo tee /etc/salt/minion
```

Restart Salt Minion

```
sudo systemctl restart salt-minion
```

Exit ```ubuntussh``` back to ```bustergraafinen``` terminal

```
exit
```

Salt Minion is now installed on ```ubuntussh```


# Installing Docker

Navigate to address https://computingforgeeks.com/install-docker-and-docker-compose-on-debian-10-buster/ to view the instructions for installing Docker on ```bustergraafinen``` Debian 10 Buster

Open ```bustergraafinen``` terminal and update the package lists

```
sudo apt-get update
```

Start the installation by ensuring that all the packages used by Docker as dependencies are installed

```
sudo apt -y install apt-transport-https ca-certificates curl gnupg2 software-properties-common
```

Import Docker GPG key used for signing Docker packages

```
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
```

Add Docker repository which contain the latest stable releases of Docker CE (Community Edition) - note that this command should be copied from the instructions of the website using the browser with ```bustergraafinen``` because it won't format right if copied from Windows

```
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"
```

Check that the previous command added the line ```deb [arch=amd64] https://download.docker.com/linux/debian buster stable``` inside the ```/etc/apt/sources.list``` file

```
sudo nano /etc/apt/sources.list
```

In case the previously used command didn't work for adding the line, you can manually copy the line ```deb [arch=amd64] https://download.docker.com/linux/debian buster stable``` inside the ```/etc/apt/sources.list``` file as follows

```
# 

# deb cdrom:[Debian GNU/Linux 10.0.0 _Buster_ - Official amd64 DVD Binary-1 20190706-10:24]/ buster contrib main non-free

# deb cdrom:[Debian GNU/Linux 10.0.0 _Buster_ - Official amd64 DVD Binary-1 20190706-10:24]/ buster contrib main non-free

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
# deb-src [arch=amd64] https://download.docker.com/linux/debian buster stable
# see the sources.list(5) manual.
```

Update the package lists

```
sudo apt-get update
```

Install Docker CE on Debian 10 Buster

```
sudo apt -y install docker-ce docker-ce-cli containerd.io
```

Docker is now installed on ```bustergraafinen```


# Installing Docker Compose (Work in progress)

Navigate to address https://computingforgeeks.com/how-to-install-latest-docker-compose-on-linux/ to view the instructions for installing latest Docker Compose on ```bustergraafinen```

You need cURL installed before starting this installation operation

```
sudo apt install -y curl
```

Download the latest Compose - note that this command should be copied from the instructions of the website using the browser with ```bustergraafinen``` because it won't format right if copied from Windows

```
curl -s https://api.github.com/repos/docker/compose/releases/latest \
  | grep browser_download_url \
  | grep docker-compose-Linux-x86_64 \
  | cut -d '"' -f 4 \
  | wget -qi -
```

Make the binary file executable

```
chmod +x docker-compose-Linux-x86_64
```

Move the file to your path

```
sudo mv docker-compose-Linux-x86_64 /usr/local/bin/docker-compose
```

Confirm Docker Compose version

```
docker-compose version
```

Place the completion script inside the ```/etc/bash_completion.d/docker-compose``` file

```
sudo curl -L https://raw.githubusercontent.com/docker/compose/master/contrib/completion/bash/docker-compose -o /etc/bash_completion.d/docker-compose
```

Check that the previous command added the script inside the ```/etc/bash_completion.d/docker-compose``` file

```
sudo nano /etc/bash_completion.d/docker-compose
```

Source the file to enjoy completion feature

```
source /etc/bash_completion.d/docker-compose
```

Create a test Docker Compose file

```
sudo nano docker-compose.yml
```

Copy the following data to the file

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

Check that the previously copied data is inside the ```docker-compose.yml``` file

```
cat docker-compose.yml
```

Look for help

```
docker-compose --help
```

```
sudo docker-compose -f ./docker-compose.yml up -d
```

```
wget localhost:8080
```

```
cat index.html
```

```
sudo docker-compose -f ./docker-compose.yml down
```


# Installing Portainer


# Installing Geany


# Installing WordPress
