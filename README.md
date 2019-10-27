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


# Remote connections to Buster and Ubuntu

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


# Changing the hostname permanently on Ubuntu

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


# Installing Salt Minion on Ubuntu

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


# Installing LAMP on Ubuntu (Work in progress)

Navigate to addresses https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-ubuntu-18-04 and http://spotwise.com/2008/11/05/adding-roles-to-ubuntu-server/ to view the instructions for installing LAMP (Linux, Apache, MySQL, PHP) on ```ubuntussh``` - I also check my own instructions from a previous Linux course from address https://pekkahamalainen.wordpress.com/2017/10/01/linux-palvelimet-5-viikon-laksyt-messuraportti-lamp-php-ja-wordpressin-asennus/

Open the terminal in ```bustergraafinen``` and establish an SSH connection to ```ubuntussh```

```
ssh -I oppilas -p 1001 10.207.5.193
```

Install Tasksel tool that can install multiple related packages as a co-ordinated "task" onto your system

sudo apt-get install tasksel

Run Tasksel

sudo tasksel

Select ```LAMP server``` in ```Software selection``` by pressing ```Space``` and press ```Enter``` to start installation, already-installed tasks will have an asterisk beside their name

Install Net-tools package containing a collection of programs which form the base of Linux networking

sudo apt-get install net-tools

Check your IP address

ifconfig

Open the browser with ```bustergraafinen``` and enter the IP address http://10.208.0.238/ of ```ubuntussh``` in the address bar - Apache2 default page opens which indicates that LAMP installation was successful

sudo ufw app list

sudo ufw app info "Apache Full"

sudo ufw allow in "Apache Full"

sudo a2enmod apache2

sudo systemctl restart apache2

sudoedit /etc/apache2/mods-available/php7.2.conf

Put hashtags

```
# Deny access to files without filename (e.g. '.php')
<FilesMatch "^\.ph(ar|p|ps|tml)$">
    Require all denied
</FilesMatch>

# Running PHP scripts in user directories is disabled by default
# 
# To re-enable PHP in user directories comment the following lines
# (from <IfModule ...> to </IfModule>.) Do NOT set it to On as it
# prevents .htaccess files from disabling it.
#<IfModule mod_userdir.c>
#    <Directory /home/*/public_html>
#        php_admin_flag engine Off
#    </Directory>
#</IfModule>
```

sudo mkdir public_html

cd public_html

sudo nano public_html

```
<!DOCTYPE html>
<html>
<head>
<title>Page Title</title>
</head>
<body>

<h1>Palvelinten hallinta</h1>

<p>This is a paragraph.</p>

<?php

print (1+7);

?>

</body>
</html>
```

sudo systemctl restart apache2

Open the browser with ```bustergraafinen``` and navigate to address http://10.208.0.238/~oppilas/ - this page shows the contents of the previously created ```index.php``` file and prints out the PHP calculation correctly

sudo apt-get remove --purge mysql-client mysql-server mysql-common mysql-client-core-5.7 mysql-server-core-5.7

Press ```Yes``` on ```Configuring mysql-server-5.7``` screen

sudo apt-get install mysql-server

sudo mysql_secure_installation

Press ```n```

New password: ```salainen```

Remove anonymous users? (Press y|Y for Yes, any other key for No) : y

Disallow root login remotely? (Press y|Y for Yes, any other key for No) : y

Remove test database and access to it? (Press y|Y for Yes, any other key for No) : y

Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y

sudo mysql

SELECT user,authentication_string,plugin,host FROM mysql.user;

ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'salainen';

FLUSH PRIVILEGES;

SELECT user,authentication_string,plugin,host FROM mysql.user;

exit

sudo mysql -p

Enter the password ```salainen```

exit

# Installing WordPress on Ubuntu (Work in progress)

Navigate to address https://www.digitalocean.com/community/tutorials/how-to-install-wordpress-with-lamp-on-ubuntu-18-04 to view the instructions for installing WordPress on ```ubuntussh``` - I also check my own instructions from a previous Linux course from address https://pekkahamalainen.wordpress.com/2017/10/01/linux-palvelimet-5-viikon-laksyt-messuraportti-lamp-php-ja-wordpressin-asennus/

sudo mysql -u root -p

CREATE DATABASE oppilas DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;

GRANT ALL ON oppilas.* TO 'oppilas'@'localhost' IDENTIFIED BY 'salainen';

FLUSH PRIVILEGES;

EXIT;

sudo apt-get update

sudo apt install php-curl php-gd php-mbstring php-xml php-xmlrpc php-soap php-intl php-zip

sudo systemctl restart apache2

sudo nano /etc/apache2/sites-available/000-default.conf 

```
<VirtualHost *:80>
        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        #ServerName www.example.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html

        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
        # error, crit, alert, emerg.
        # It is also possible to configure the loglevel for particular
        # modules, e.g.
        #LogLevel info ssl:warn

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        # For most configuration files from conf-available/, which are
        # enabled or disabled at a global level, it is possible to
        # include a line for only one particular virtual host. For example the
        # following line enables the CGI configuration for this host only
        # after it has been globally disabled with "a2disconf".
        #Include conf-available/serve-cgi-bin.conf

        <Directory /var/www/wordpress/>
                AllowOverride All
        </Directory>
</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```

sudo a2enmod rewrite

sudo apache2ctl configtest

sudo systemctl restart apache2

cd /tmp

curl -O https://wordpress.org/latest.tar.gz

tar xzvf latest.tar.gz

touch /tmp/wordpress/.htaccess

cp /tmp/wordpress/wp-config-sample.php /tmp/wordpress/wp-config.php

mkdir /tmp/wordpress/wp-content/upgrade

sudo cp -a /tmp/wordpress/. /var/www/wordpress

sudo chown -R www-data:www-data /var/www/wordpress

sudo find /var/www/wordpress/ -type d -exec chmod 750 {} \;

sudo find /var/www/wordpress/ -type f -exec chmod 640 {} \;

curl -s https://api.wordpress.org/secret-key/1.1/salt/

```
define('AUTH_KEY',         '8)b6`snQr(c2B4p(yECs2Vy$,wo#>EL+t?=C)_%t0RS0=]T>|_couV]+@FkGrG]x');
define('SECURE_AUTH_KEY',  'Oaj(sLk+j&0g!O|K+`6+]@GZ_O^$a~oI|UvN-F>k+w%wWL+3+?^S}5+Tje?5$j(>');
define('LOGGED_IN_KEY',    '8jmy{jP7hJOt[1zJnQy].wQu{+tfD^$XT`E4}: @SeKVO+TNn| a+K=*+VL+?ktx');
define('NONCE_KEY',        'FF=/21X|||[Ow>zy3_d7`rT,.6zH[L&HTA=sGgT#vE]v(wBpS32 8h7e~/g+ean*');
define('AUTH_SALT',        '*Th=i9Re!|n0@z!L-fuY[`YK=K+[n?lZ8FfVM{Q!|-_;.P3`=/63X]|h}.}m:9am');
define('SECURE_AUTH_SALT', '!,Y9g:p%qmhA7pMbuTO*dA(~tx oblA(K#QXiC$8{C*[.;NUE/=)I+X%<[p6>akl');
define('LOGGED_IN_SALT',   'Tu}?TP-k8B4ohyA|Zo1z+o^6rXjMW-S.jwfWCk2-K|mrZb+J>2M!{IVS2{Y*P=-5');
define('NONCE_SALT',       '5)gUV|S}*jcs|+KPNu4^zNB~~FaE_o}7?[|8]I.IAqrSeAaY7kC0<!2C#hidVB0H');
```

sudo nano /var/www/wordpress/wp-config.php

```
/**#@+
 * Authentication Unique Keys and Salts.
 *
 * Change these to different unique phrases!
 * You can generate these using the {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org secret-key service}
 * You can change these at any point in time to invalidate all existing cookies. This will force all users to have to log in again.
 *
 * @since 2.6.0
 */
define('AUTH_KEY',         '8)b6`snQr(c2B4p(yECs2Vy$,wo#>EL+t?=C)_%t0RS0=]T>|_couV]+@FkGrG]x');
define('SECURE_AUTH_KEY',  'Oaj(sLk+j&0g!O|K+`6+]@GZ_O^$a~oI|UvN-F>k+w%wWL+3+?^S}5+Tje?5$j(>');
define('LOGGED_IN_KEY',    '8jmy{jP7hJOt[1zJnQy].wQu{+tfD^$XT`E4}: @SeKVO+TNn| a+K=*+VL+?ktx');
define('NONCE_KEY',        'FF=/21X|||[Ow>zy3_d7`rT,.6zH[L&HTA=sGgT#vE]v(wBpS32 8h7e~/g+ean*');
define('AUTH_SALT',        '*Th=i9Re!|n0@z!L-fuY[`YK=K+[n?lZ8FfVM{Q!|-_;.P3`=/63X]|h}.}m:9am');
define('SECURE_AUTH_SALT', '!,Y9g:p%qmhA7pMbuTO*dA(~tx oblA(K#QXiC$8{C*[.;NUE/=)I+X%<[p6>akl');
define('LOGGED_IN_SALT',   'Tu}?TP-k8B4ohyA|Zo1z+o^6rXjMW-S.jwfWCk2-K|mrZb+J>2M!{IVS2{Y*P=-5');
define('NONCE_SALT',       '5)gUV|S}*jcs|+KPNu4^zNB~~FaE_o}7?[|8]I.IAqrSeAaY7kC0<!2C#hidVB0H');
/**#@-*/
```

```
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'oppilas' );

/** MySQL database username */
define( 'DB_USER', 'oppilas' );

/** MySQL database password */
define( 'DB_PASSWORD', 'salainen' );

define('FS_METHOD', 'direct');
```

cd /wordpress/

cd ..

sudo mv wordpress /home/oppilas/public_html/

Open address http://10.208.0.238/~oppilas/wordpress/

English (United States)

Site Title: Pekanloki

Username: oppilas

Password: salainen

Your Email: pekka.hamalainen@myy.haaga-helia.fi

Search Engine Visibility: untick

Confirm and Log in

Creat a first post


# Configuring SSH key-based authentication on Ubuntu (Work in progress)

Navigate to address https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-freebsd-server to view the instructions for configuring SSH key-based authentication on ```ubuntussh```


# Installing Docker on Buster

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

Add Docker repository which contain the latest stable releases of Docker CE (Community Edition) - note that this command should be copied from the instructions of the website using the browser in ```bustergraafinen``` because it won't format right if copied from Windows

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

The previous installation will add ```docker``` group to the system without any users so we are adding your user account to the group to run Docker commands as non-privileged user

```
sudo usermod -aG docker $USER
```

Change the current group of your user to ```docker``` where we added it previously

```
newgrp docker
```

Check Docker and Compose version

```
docker version
```

Log out and log back in so that the group membership of your user is re-evaluated

Run a test Docker container

```
docker run --rm -it  --name test alpine:latest /bin/sh
```

Check the operating system of the container

```
cat /etc/os-release
```

Exit the container

```
exit
```

Docker is now installed and tested on ```bustergraafinen```


# Installing Docker Compose on Buster

Navigate to address https://computingforgeeks.com/how-to-install-latest-docker-compose-on-linux/ to view the instructions for installing latest Docker Compose on ```bustergraafinen```

You need cURL installed before starting this installation operation

```
sudo apt install -y curl
```

Download the latest Compose - note that this command should be copied from the instructions of the website using the browser in ```bustergraafinen``` because it won't format right if copied from Windows

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
nano docker-compose.yml
```

Copy the following data inside the ```docker-compose.yml``` file

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

Check help for Docker Compose command syntax

```
docker-compose --help
```

Create and start service containers

```
docker-compose up -d
```

Show running containers

```
docker-compose ps
```

Enter the following command to download a ```index.html``` file for testing purposes

```
wget localhost:8080
```

Check the contents inside the ```index.html``` file

```
cat index.html
```

Stop service containers

```
docker-compose stop
```

Remove stopped service containers

```
docker-compose rm -f
```

Remove the ```index.html``` file

```
rm -r index.html
```

Docker Compose is now installed and tested on ```bustergraafinen```


# Installing Portainer on Buster

Navigate to address https://computingforgeeks.com/install-docker-ui-manager-portainer/ to view the instructions for installing Portainer on ```bustergraafinen```

Open ```bustergraafinen``` terminal and update the package lists

```
sudo apt-get update
```

To persist Docker container data, create a directory that will hold all Portainer data

```
sudo mkdir ~/portainer
```

Download the image from Docker hub and store it locally on Docker host

```
sudo docker pull portainer/portainer
```

Tag the image and give it a custom name

```
sudo docker tag portainer/portainer portainer
```

Export the Portainer container

```
export CONT_NAME="portainer"
```

Start the Portainer container - note that this command should be copied from the instructions of the website using the browser in ```bustergraafinen``` because it won't format right if copied from Windows

```
sudo docker run -d -p 9000:9000 \
--restart always \
-v /var/run/docker.sock:/var/run/docker.sock \
-v ~/portainer:/data \
--name ${CONT_NAME} \
portainer
```

Access the web dashboard on address http://127.0.0.1:9000 and perform the following steps

- Create an admin user and provide ```Username``` and ```Password``` - I use the same credentials as in all the previous times, and then click ```Create user```
- Add a Docker environment by choosing between ```Local``` Docker engine or ```Remote``` - I’m using it to manage ```Local``` Docker engine so I choose that and press ```Connect```
- Portainer ```Dashboard``` will open where you can start managing Docker engine operations from a web UI - the default section has a summary of the number of containers, Docker version, volumes, networks etc.
- Click the ```Home``` tab and select the ```local``` engine to open its ```Dashboard``` - clicking on the ```Host``` (Formerly ```Engine```) section will give you all the information you need to know about your Docker engine
- ```App Templates``` section in Portainer tries to make the deployment of applications on Docker containers easy by providing a number of templates ready to use, this is available for both Windows and Linux, and you can search and deploy container within no time

Portainer is now installed on ```bustergraafinen```


# Installing Geany on Buster (Work in progress)


# Pulling Harri's ready to run Salt Master and Minion container on Buster (Work in progress)

For testing purposes, navigate to address https://hub.docker.com/r/darkdth/saltstacktesting/tags and check the command for pulling the latest Salt Master and Salt Minion container

Download the image and store it locally to ```bustergraafinen```

```
docker pull darkdth/saltstacktesting:minsaltstackbootstrap_v03
```
