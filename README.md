# Linux Server Management

Linux notes and instructions for a course about Linux server management in Haaga-Helia University of Applied Sciences


# Author

Pekka Hämäläinen


# 1. Initializing Rovius CP

Open VDI from address https://vdi.haaga-helia.fi if you are working outside of Haaga-Helia lab environment, and navigate to Rovius CP in address https://vdi-lab.cp.haaga-helia.fi/client/

Enter your ```Username```, ```Password```, and ```Domain```, after which press ```Login``` - ```Username``` and ```Password``` are the same as in your student ID, and ```Domain``` is ```ITLAB```

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

Navigate back to ```Network```, choose ```pekanverkko```, press ```View IP Addresses```, choose ```x.x.x.x [Source NAT]```, press ```Configuration```, choose ```Firewall```, and configure the following parameters

- Add address ```0.0.0.0/0``` to ```Source CIDR```
- Change ```Protocol``` to ```TCP```
- Add ```1001``` to ```Start Port```
- Add ```1009``` to ```End Port```
- Press ```Add```

Lastly, navigate to ```Network```, choose ```pekanverkko```, press ```View IP Addresses```, choose ```x.x.x.x [Source NAT]```, press ```Configuration```, choose ```Port Forwarding```, and configure the following parameters for three different ports

- For the first port, add ```22``` ```22``` to ```Private Port``` and ```1001``` ```1001``` to ```Public Port```, change ```Protocol``` to ```TCP```, press ```Add```, select ```ubuntussh```, and press ```Apply```
- For the second port, add ```22``` ```22``` to ```Private Port``` and ```1002``` ```1002``` to ```Public Port```, change ```Protocol``` to ```TCP```, press ```Add```, select ```bustergraafinen```, and press ```Apply```
- Fort the third port, add ```3389``` ```3389``` to ```Private Port``` and ```1003``` ```1003``` to ```Public Port```, change ```Protocol``` to ```TCP```, press ```Add```, select ```bustergraafinen```, and press ```Apply```

The instances ```bustergraafinen``` and ```ubuntussh``` are now working correctly, as well as the network ```pekanverkko```


# 2. Remote connections to ```bustergraafinen``` and ```ubuntussh```

First, we will establish a remote desktop connection to the RDP instance ```bustergraafinen``` using the following steps

- Open ```Remote Desktop Connection``` application and press ```Show Options```
- Type ```x.x.x.x:1003``` to ```Computer:``` and press ```Connect```
- A warning prompt will pop up for which you need to press ```Yes```
- In the login window, keep ```Session``` as ```Xorg```, enter ```oppilas``` for ```username```, enter ```salainen``` for ```password```, and press ```OK```

https://www.technipages.com/unable-to-copy-and-paste-to-remote-desktop-session

Next, we will establish an SSH connection to the Ubuntu server instance ```ubuntussh``` using the terminal in ```bustergraafinen```

Open the terminal with ```Ctrl + Alt + T```

Check help for SSH syntax

```
ssh --help
```

Establish an SSH connection to ```ubuntussh```

```
ssh -l oppilas 10.208.0.83
```

After giving the correct password, you should be in ```ubuntussh``` terminal

Exit back to local ```bustergraafinen``` terminal

```
exit
```

Alternatively, you can use ```PuTTY``` to establish an SSH connection to ```ubuntussh``` using the following steps

- Open ```PuTTY``` application and select ```Session```
- Enter ```x.x.x.x``` to ```Host Name (or IP address)``` and ```1001``` to ```Port```, after which press ```Open```
- Enter ```oppilas``` to ```login as:``` prompt and ```salainen``` to ```oppilas@x.x.x.x's password:``` prompt, after which the terminal to ```ubuntussh``` should open

Remote connections to ```bustergraafinen``` and ```ubuntussh``` are now working correctly


# 3. Changing the hostname permanently on ```ubuntussh```

Navigate to address http://x.x.x.x/ first and then open the link from there to https://www.cyberciti.biz/faq/ubuntu-18-04-lts-change-hostname-permanently/ to view the instructions for changing ```ubuntussh``` Ubuntu server hostname permanently

Open the terminal in ```bustergraafinen``` and establish an SSH connection to ```ubuntussh```

```
ssh -l oppilas 10.208.0.83
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
x.x.x.x salt
x.x.x.x saltgrandmaster1

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


# 4. Installing Salt Minion on ```ubuntussh```

Navigate to address http://x.x.x.x/ to view the instructions for installing Salt Minion on ```ubuntussh```

Open the terminal in ```bustergraafinen``` and establish an SSH connection to ```ubuntussh```

```
ssh -l oppilas 10.208.0.83
```

Install Salt Minion

```
sudo apt-get -y install salt-minion
```

Give the following command - note that it is a single command, even though it might be printed on two lines, and also note that this command should be copied from the instructions of the website using the browser in ```bustergraafinen``` because it won't format right if copied from Windows

```
SALTID=$(getent passwd $USER| cut -d ':' -f 5 | cut -d ',' -f 1|tr -c '[a-zA-Z]' '_'; echo -n "_$(hostname)_"; date +'%H%M%S') echo -e "master: x.x.x.x\nid: $SALTID"|sudo tee /etc/salt/minion
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


# 5. Installing LAMP on ```ubuntussh```

Navigate to address https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-ubuntu-18-04 to view the instructions for installing LAMP (Linux, Apache2, MySQL, PHP) on ```ubuntussh```

Open the terminal in ```bustergraafinen``` and establish an SSH connection to ```ubuntussh```

```
ssh -l oppilas 10.208.0.83
```

```
sudo apt-get update
```

```
sudo apt-get install apache2
```

```
sudo ufw app list
```

```
sudo ufw app info "Apache Full"
```

```
sudo ufw allow in "Apache Full"
```

```
ip addr show eth0 | grep inet | awk '{ print $2; }' | sed 's/\/.*$//'
````

Go to address http://10.208.0.83/

```
sudo apt-get install curl
```

```
curl http://icanhazip.com
```

```
sudo apt-get install mysql-server
```

```
sudo mysql_secure_installation
```

```
VALIDATE PASSWORD PLUGIN Press y|Y for Yes, any other key for No: n
```

```
New password: salainen
```

```
Re-enter new password: salainen
```

```
Remove anonymous users? (Press y|Y for Yes, any other key for No) : y
```

```
Disallow root login remotely? (Press y|Y for Yes, any other key for No) : y
```

```
Remove test database and access to it? (Press y|Y for Yes, any other key for No) : y
```

```
Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y
```

```
sudo mysql
```

```
SELECT user,authentication_string,plugin,host FROM mysql.user;
```

```
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'salainen';
```

```
FLUSH PRIVILEGES;
```

```
SELECT user,authentication_string,plugin,host FROM mysql.user;
```

```
exit
```

```
sudo mysql -p
```

```
exit
```

Install PHP and helper packages

```
sudo apt-get install php libapache2-mod-php php-mysql
```

Navigate inside ```/etc/apache2/mods-enabled/dir.conf``` file for editing

```
sudo nano /etc/apache2/mods-enabled/dir.conf
```

Cut and move the ```index.php``` file to the first position after the ```DirectoryIndex``` specification

```
<IfModule mod_dir.c>
        DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```

```
sudo systemctl restart apache2
```

```
sudo systemctl status apache2
```

```
q
```

```
apt search php- | less
```

```
q
```

```
apt show package_name
```

```
apt show php-cli
```

```
sudo apt-get install php-cli
```

```
sudo apt-get install package1 package2 ...
```

```
sudo mkdir /var/www/ubuntussh
```

```
sudo chown -R $USER:$USER /var/www/ubuntussh
```

```
sudo chmod -R 755 /var/www/ubuntussh
```

Navigate inside ```/var/www/ubuntussh/index.html``` file for editing

```
nano /var/www/ubuntussh/index.html
```

Copy the following text inside the ```/var/www/ubuntussh/index.html``` file

```
<html>
    <head>
        <title>Welcome to ubuntussh!</title>
    </head>
    <body>
        <h1>Success! The ubuntussh server block is working!</h1>
        <p>Tremendous!</p>
    </body>
</html>
```

Navigate inside ```/etc/apache2/sites-available/ubuntussh.conf``` file for editing

```
sudo nano /etc/apache2/sites-available/ubuntussh.conf
```

Copy the following text inside the ```/etc/apache2/sites-available/ubuntussh.conf``` file

```
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName ubuntussh
    ServerAlias www.ubuntussh
    DocumentRoot /var/www/ubuntussh
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

```
sudo a2ensite ubuntussh.conf
```

```
sudo a2dissite 000-default.conf
```

```
sudo apache2ctl configtest
```

```
sudo systemctl restart apache2
```

Go to address http://ubuntussh/

Navigate inside ```/var/www/ubuntussh/info.php``` file for editing

```
sudo nano /var/www/ubuntussh/info.php
```

Copy the following text inside the ```/var/www/ubuntussh/info.php``` file

```
<?php
phpinfo();
?>
```

Go to address http://ubuntussh/info.php

```
sudo rm /var/www/ubuntussh/info.php
```

LAMP is now installed on ```ubuntussh```


# 6. Installing WordPress on ```ubuntussh```

Navigate to address https://www.digitalocean.com/community/tutorials/how-to-install-wordpress-with-lamp-on-ubuntu-18-04 to view the instructions for installing WordPress on ```ubuntussh```

```
mysql -u root -p
```

```
CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
```

```
GRANT ALL ON wordpress.* TO 'wordpressuser'@'localhost' IDENTIFIED BY 'salainen';
```

```
FLUSH PRIVILEGES;
```

```
EXIT;
```

```
sudo apt-get update
```

```
sudo apt-get install php-curl php-gd php-mbstring php-xml php-xmlrpc php-soap php-intl php-zip
```

```
sudo systemctl restart apache2
```

```
sudo nano /etc/apache2/sites-available/ubuntussh.conf
```

```
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName ubuntussh
    ServerAlias www.ubuntussh
    DocumentRoot /var/www/ubuntussh
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
    <Directory /var/www/ubuntussh/>
        AllowOverride All
    </Directory>
</VirtualHost>
```

```
sudo a2enmod rewrite
```

```
sudo apache2ctl configtest
```

```
sudo systemctl restart apache2
```

```
cd /tmp
```

```
curl -O https://wordpress.org/latest.tar.gz
```

```
tar xzvf latest.tar.gz
```

```
touch /tmp/wordpress/.htaccess
```

```
cp /tmp/wordpress/wp-config-sample.php /tmp/wordpress/wp-config.php
```

```
mkdir /tmp/wordpress/wp-content/upgrade
```

```
sudo cp -a /tmp/wordpress/. /var/www/ubuntussh
```

```
sudo chown -R www-data:www-data /var/www/ubuntussh
```

```
sudo find /var/www/ubuntussh/ -type d -exec chmod 750 {} \;
```

```
sudo find /var/www/ubuntussh/ -type f -exec chmod 640 {} \;
```

```
curl -s https://api.wordpress.org/secret-key/1.1/salt/
```

```
define('AUTH_KEY',         '%8CNX`|py%-^Fk$QhU>o0FpSk2Ul*fTdeTIfnV:=}<(*v@)!]`gH<]=dRK)[+i6T');
define('SECURE_AUTH_KEY',  '1@~b6R?@_Ys43v|k9wFyF;SvA:*sPbz5+=n?:rQI273fHEY3;|QL828<1%SX++TC');
define('LOGGED_IN_KEY',    'RbhYg(_{ir;=>bt~]ay-- 3oh~qSz^!C+E*!7X7Oh~w`%.S&#l<oOCG>5sPR(-Z5');
define('NONCE_KEY',        '],ZyT3k|+fe7KT~P+dbc9Kz%N`gMp0f_>0rhN+wp.ONP(0R-Xl%6o7$GM}9/~<Bh');
define('AUTH_SALT',        '.{/ZKIJ-fzTDHO:,qo)QTcAU4GsQMhiq #n&,n]G8*v?*r-UD*/oyb||j!#TC3hq');
define('SECURE_AUTH_SALT', '8$o^r2c=hQFp&Z(%crLPkV~R$=%=Y)LR]HL<+o:l!cc$Y|Rm]rd|CsAN6j9S8dQS');
define('LOGGED_IN_SALT',   'd:BC*5TWlq{Mo<NK5XWa()<ok,JU#o{U+I^+R j}w%_3/WHC-F6|(K(yv|DTQ(1;');
define('NONCE_SALT',       '| ,[ke$,=4f|FTft]29|=|q[<Gn1N4nusycSv+bzJP[@MaUt T2^^P7 +!^czRI-');
```

```
sudo nano /var/www/ubuntussh/wp-config.php
```

```
<?php
/**
 * The base configuration for WordPress
 *
 * The wp-config.php creation script uses this file during the
 * installation. You don't have to use the web site, you can
 * copy this file to "wp-config.php" and fill in the values.
 *
 * This file contains the following configurations:
 *
 * * MySQL settings
 * * Secret keys
 * * Database table prefix
 * * ABSPATH
 *
 * @link https://codex.wordpress.org/Editing_wp-config.php
 *
 * @package WordPress
 */

// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress' );

/** MySQL database username */
define( 'DB_USER', 'wordpressuser' );

/** MySQL database password */
define( 'DB_PASSWORD', 'salainen' );

/** MySQL filesystem method*/
define( 'FS_METHOD', 'direct' );

/** MySQL hostname */
define( 'DB_HOST', 'localhost' );

/** Database Charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );

/** The Database Collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );

/**#@+
 * Authentication Unique Keys and Salts.
 *
 * Change these to different unique phrases!
 * You can generate these using the {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org secret-key service}
 * You can change these at any point in time to invalidate all existing cookies. This will force all users to have to log in again.
 *
 * @since 2.6.0
 */
define('AUTH_KEY',         '%8CNX`|py%-^Fk$QhU>o0FpSk2Ul*fTdeTIfnV:=}<(*v@)!]`gH<]=dRK)[+i6T');
define('SECURE_AUTH_KEY',  '1@~b6R?@_Ys43v|k9wFyF;SvA:*sPbz5+=n?:rQI273fHEY3;|QL828<1%SX++TC');
define('LOGGED_IN_KEY',    'RbhYg(_{ir;=>bt~]ay-- 3oh~qSz^!C+E*!7X7Oh~w`%.S&#l<oOCG>5sPR(-Z5');
define('NONCE_KEY',        '],ZyT3k|+fe7KT~P+dbc9Kz%N`gMp0f_>0rhN+wp.ONP(0R-Xl%6o7$GM}9/~<Bh');
define('AUTH_SALT',        '.{/ZKIJ-fzTDHO:,qo)QTcAU4GsQMhiq #n&,n]G8*v?*r-UD*/oyb||j!#TC3hq');
define('SECURE_AUTH_SALT', '8$o^r2c=hQFp&Z(%crLPkV~R$=%=Y)LR]HL<+o:l!cc$Y|Rm]rd|CsAN6j9S8dQS');
define('LOGGED_IN_SALT',   'd:BC*5TWlq{Mo<NK5XWa()<ok,JU#o{U+I^+R j}w%_3/WHC-F6|(K(yv|DTQ(1;');
define('NONCE_SALT',       '| ,[ke$,=4f|FTft]29|=|q[<Gn1N4nusycSv+bzJP[@MaUt T2^^P7 +!^czRI-');

/**#@-*/

/**
 * WordPress Database Table prefix.
 *
 * You can have multiple installations in one database if you give each
 * a unique prefix. Only numbers, letters, and underscores please!
 */
$table_prefix = 'wp_';

/**
 * For developers: WordPress debugging mode.
 *
 * Change this to true to enable the display of notices during development.
 * It is strongly recommended that plugin and theme developers use WP_DEBUG
 * in their development environments.
 *
 * For information on other constants that can be used for debugging,
 * visit the Codex.
 *
 * @link https://codex.wordpress.org/Debugging_in_WordPress
 */
define( 'WP_DEBUG', false );

/* That's all, stop editing! Happy publishing. */

/** Absolute path to the WordPress directory. */
if ( ! defined( 'ABSPATH' ) ) {
        define( 'ABSPATH', dirname( __FILE__ ) . '/' );
}

/** Sets up WordPress vars and included files. */
require_once( ABSPATH . 'wp-settings.php' );
```

Go to address http://ubuntussh/

English (United States)

Continue

```Site Title``` Pekan WordPress

```Username``` ubuntussh

```Password``` salainen

```Confirm Password``` Check ```Confirm use of weak password```

```Your Email``` pekka.hamalainen@myy.haaga-helia.fi ```Double-check your email address before continuing.```

```Search Engine Visibility``` Uncheck ```Discourage search engines from indexing this site It is up to search engines to honor this request.```

Install WordPress

Log In

Appearance

Themes

Twenty Seventeen

Activate

Posts

Hello world!

Trash

Add New

```
First Post on Pekan Wordpress

You will find my Linux notes and instructions for a course about Linux server management in Haaga-Helia University of Applied Sciences at https://github.com/Pece17/LinuxServerManagement
```

Publish

View Post

Hover over ```Howdy, ubuntussh```

Log Out

WordPress is now installed on ```ubuntussh```


# 7. Configuring SSH key-based authentication on ```bustergraafinen``` and ```ubuntussh```

Navigate to address https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-freebsd-server to view the instructions for configuring SSH key-based authentication on ```bustergraafinen``` and ```ubuntussh```

Open ```bustergraafinen``` terminal and generate a key pair with this command

ssh-keygen

Enter file in which to save the key (/home/oppilas/.ssh/id_rsa): Press Enter for default key location

/home/oppilas/.ssh/id_rsa already exists.
Overwrite (y/n)? Press y to agree

Enter passphrase (empty for no passphrase): Press Enter for empty

Enter same passphrase again: Press Enter for empty

```
Your identification has been saved in /home/oppilas/.ssh/id_rsa.
Your public key has been saved in /home/oppilas/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:atI1MNNBrLmHmYPeS9i/PZqfSu0vwwQANET2sT52bk4 oppilas@deb10xfcews1
The key's randomart image is:
+---[RSA 2048]----+
|   +B..oo        |
|   . o.+..       |
|      *+.        |
|     .o+.        |
|     .+=S.       |
|    .=**.o.      |
|   .o.*oEo.      |
|    .+.* +=.     |
|      ..B===.    |
+----[SHA256]-----+
```

SSH to ```ubuntussh```

```
ssh -l oppilas 10.208.0.83
```

Create the following folder

mkdir -p ~/.ssh

Open ```bustergraafinen``` terminal aside and enter this command into the terminal to print your public SSH key

cat ~/.ssh/id_rsa.pub

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDBQhk63HMYl8d8A4mrp++xAUsEcOjwm5aZOwPt8qBXHvVucA7W/e965CAqN3TKjfBQrJSIw3Is4TQRl5kWGgY/lqUoRdDUdEZ2u3mD7wJKaimBEuU6GvjgfIQvt+C7tlxmvEsSQJ+OHdVApOPTK/cUMvhQZwCdi6bCk3wRP6D/ZXejjFuUQoFMPf3IYun2rvB4VWQg7EFAWq7VwRyq0EY9i1xCVsFO6I+f1X9TE7uKPEl76W0ou7O3lGXNzod9kFXMVfhSh3lH1TnZRJcIqZj2FBKuB44jyXPNcMsw1uVH3+BGh8cCp3qq2EpsjcYny1iO8082wi1koRC56TeoiUoj oppilas@deb10xfcews1
```

Open ```ubuntussh``` terminal and open the ```authorized_keys``` file in the text editor of your choice

sudo nano ~/.ssh/authorized_keys

Paste your public key into the ```authorized_keys``` file, then save and exit

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDBQhk63HMYl8d8A4mrp++xAUsEcOjwm5aZOwPt8qBXHvVucA7W/e965CAqN3TKjfBQrJSIw3Is4TQRl5kWGgY/lqUoRdDUdEZ2u3mD7wJKaimBEuU6GvjgfIQvt+C7tlxmvEsSQJ+OHdVApOPTK/cUMvhQZwCdi6bCk3wRP6D/ZXejjFuUQoFMPf3IYun2rvB4VWQg7EFAWq7VwRyq0EY9i1xCVsFO6I+f1X9TE7uKPEl76W0ou7O3lGXNzod9kFXMVfhSh3lH1TnZRJcIqZj2FBKuB44jyXPNcMsw1uVH3+BGh8cCp3qq2EpsjcYny1iO8082wi1koRC56TeoiUoj oppilas@deb10xfcews1
```

Exit from ```ubuntussh``` terminal

exit

SSH back to ```ubuntussh``` terminal from ```bustergraafinen``` terminal - now you should be able to login with SSH key-based authentication without needing to enter the password

ssh -l oppilas 10.208.0.83

SSH key-based authentication is now configured on ```bustergraafinen``` and ```ubuntussh```


# 8. Installing and securing phpMyAdmin on ```ubuntussh``` (Work in progress)

Navigate to address https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-phpmyadmin-on-ubuntu-18-04 to view the instructions for installing and securing phpMyAdmin on ```ubuntussh```


# 9. Installing Docker on ```bustergraafinen```

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


# 10. Installing Docker Compose on ```bustergraafinen```

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


# 11. Installing Portainer on ```bustergraafinen```

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


# 12. Installing Geany IDE on ```bustergraafinen```

Navigate to address https://tecadmin.net/install-geany-ide-ubuntu/ to view the instructions for installing Geany IDE (Integrated Development Environment) on ```bustergraafinen```

Open ```bustergraafinen``` terminal and configure PPA (Personal Package Archive) of Geany IDE to your system

```
sudo add-apt-repository ppa:geany-dev/ppa
```

Update the package lists

```
sudo apt-get update
```

Install Geany IDE

```
sudo apt-get install geany geany-plugins-common
```

Geany IDE is now installed on ```bustergraafinen```


# 13. Pulling Harri's ready-to-run Salt Master and Salt Minion container on ```bustergraafinen``` (Work in progress)

For testing purposes, navigate to address https://hub.docker.com/r/darkdth/saltstacktesting/tags and check the command for pulling the latest Salt Master and Salt Minion container

Download the image and store it locally to ```bustergraafinen```

```
docker pull darkdth/saltstacktesting:minsaltstackbootstrap_v03
```

```
docker images
```

```
mkdir koe
```

```
cd koe
```

```
mkdir -p srv/salt
```

```
docker run -v /Users/hja/koe/srv/salt/:/srv/salt -it darkdth/saltstacktesting:minsaltstackbootstrap_v03 /bin/bash
```

```
nano /etc/hosts
```

```
127.0.0.1       salt     
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.3      51ffae81496b
```

```
/etc/init.d/salt-master start
```

```
/etc/init.d/salt-minion start
```

```
salt '*' test.ping
```

```
salt '*' sys.doc saltstates
```

```
salt '*' sys.list_salt_functions
```

```
salt '*' grains.get os
```

```
echo -e "nginx:\n  pkg.installed" > /srv/salt/nginx_installed.sls
```

```
salt '*' state.apply nginx_installed
```


# 14. Initializing three more instances in Rovius CP

Open VDI from address https://vdi.haaga-helia.fi if you are working outside of Haaga-Helia lab environment, and navigate to Rovius CP in address https://vdi-lab.cp.haaga-helia.fi/client/

Enter your ```Username```, ```Password```, and ```Domain```, after which press ```Login``` - ```Username``` and ```Password``` are the same as in your student ID, and ```Domain``` is ```ITLAB```

Change ```Project``` to ```ICT4TN 022-3006 Pekka Hämäläinen``` from upper left corner

We previously created two instances called ```bustergraafinen``` and ```ubuntussh```, but now we have to create three more instances called ```ubuntumaster```, ```centosminion```, and ```ubuntuminion``` - this brings the total amount of instances to five

Navigate to ```Instances``` and press ```Add Instance```

Configure the following settings for the third instance that will be an Ubuntu Salt Master that is running Ubuntu 14.04 operating system

1. Choose ```HH01```, ```Template```, and press ```Next```
2. Choose ```UBUsrvSaltMaster1AH``` and press ```Next```
3. Choose ```Linux Normal``` and press ```Next```
4. Choose ```No thanks``` and press ```Next```
5. Press ```Next```
6. Choose previously made ```pekanverkko``` under ```Networks``` and press ```Next```
7. Press ```Next```
8. Change ```Name``` to ```ubuntumaster```, choose ```Standard (US) keyboard```, and press ```Launch VM```

Configure the following settings for the fourth instance that will be a CentOS Salt Minion that is running CentOS 7.2 operating system

1. Choose ```HH01```, ```Template```, and press ```Next```
2. Choose ```CENTOS7cloudSaltMinidb1dsk1AH``` and press ```Next```
3. Choose ```Linux Normal``` and press ```Next```
4. Choose ```No thanks``` and press ```Next```
5. Press ```Next```
6. Choose previously made ```pekanverkko``` under ```Networks``` and press ```Next```
7. Press ```Next```
8. Change ```Name``` to ```centosminion```, choose ```Standard (US) keyboard```, and press ```Launch VM```

Configure the following settings for the fifth instance that will be an Ubuntu Salt Minion that is running Ubuntu 14.04 operating system

1. Choose ```HH01```, ```Template```, and press ```Next```
2. Choose ```UBUsrvSaltMaster1AH``` and press ```Next```
3. Choose ```Linux Normal``` and press ```Next```
4. Choose ```No thanks``` and press ```Next```
5. Press ```Next```
6. Choose previously made ```pekanverkko``` under ```Networks``` and press ```Next```
7. Press ```Next```
8. Change ```Name``` to ```ubuntuminion```, choose ```Standard (US) keyboard```, and press ```Launch VM```

Lastly, navigate to ```Network``` > ```pekanverkko``` > ```View IP Addresses``` > ```x.x.x.x [Source NAT]``` > ```Configuration``` > ```Port Forwarding``` > configure the following parameters for three different ports

The instances ```ubuntumaster```, ```centosminion```, and ```ubuntuminion``` are now working correctly


# 15. Remote connections to ```ubuntumaster```, ```centosminion```, and ```ubuntuminion```

Connecting to ```ubuntumaster``` from ```bustergraafinen``` terminal

```
ssh -l oppilas 10.208.0.56
```

Connecting to ```centosminion``` from ```bustergraafinen``` terminal

```
ssh -l root 10.208.0.55
```

Connecting to ```ubuntuminion``` from ```bustergraafinen``` terminal

```
ssh -l oppilas 10.208.0.43
```


# 16. Changing the hostname permanently on ```ubuntumaster```, ```centosminion```, and ```ubuntuminion```

Navigate to address https://www.cyberciti.biz/faq/ubuntu-18-04-lts-change-hostname-permanently/ to view the instructions for changing the hostname permanently on ```ubuntumaster```, ```centosminion```, and ```ubuntuminion```

Open the terminal in ```bustergraafinen``` and establish an SSH connection to ```ubuntumaster```

```
ssh -l oppilas 10.208.0.56
```

Display disk usage of the file system in human readable format

```
df -h
```

Install available upgrades for all packages currently installed

```
sudo apt-get upgrade
```

Replace the old hostname with ```ubuntumaster```

```
sudo hostnamectl set-hostname ubuntumaster
```

Check the new hostname

```
hostname
```

Navigate inside ```/etc/hosts``` file for editing

```
sudo nano /etc/hosts
```

Add new line ```127.0.1.1 ubuntumaster``` inside the file

```
127.0.0.1 localhost
127.0.1.1 ubuntumaster

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

Reboot the system

```
sudo reboot
```

Open the terminal in ```bustergraafinen``` and establish an SSH connection to ```centosminion```

```
ssh -l root 10.208.0.55
```

Install available upgrades for all packages currently installed

```
yum upgrade
```

Replace the old hostname with ```centosminion```

```
sudo hostnamectl set-hostname centosminion
```

Check the new hostname

```
hostname
```

Install GNU (GNU’s Not Unix) nano text editor

```
yum install nano
```

Navigate inside ```/etc/hosts``` file for editing

```
sudo nano /etc/hosts
```

Add new line ```127.0.1.1 centosminion``` inside the file

```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

127.0.1.1 centosminion
```

Reboot the system

```
sudo reboot
```

Open the terminal in ```bustergraafinen``` and establish an SSH connection to ```centosminion```

```
ssh -l root 10.208.0.55
```

Ping the loopback address ```127.0.1.1```

```
ping 127.0.1.1
```

Ping ```centosminion```

```
ping centosminion
```

Exit back to ```bustergraafinen``` terminal

```
exit
```

Open the terminal in ```bustergraafinen``` and establish an SSH connection to ```ubuntuminion```

```
ssh -l oppilas 10.208.0.43
```

Install available upgrades for all packages currently installed

```
sudo apt-get upgrade
```

Replace the old hostname with ```ubuntuminion```

```
sudo hostnamectl set-hostname ubuntuminion
```

Check the new hostname

```
hostname
```

Navigate inside ```/etc/hosts``` file for editing

```
sudo nano /etc/hosts
```

Add new line ```127.0.1.1 ubuntuminion``` inside the file

```
127.0.0.1 localhost
127.0.1.1 ubuntuminion    

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

Reboot the system

```
sudo reboot
```

The hostname is now changed permanently on ```ubuntumaster```, ```centosminion```, and ```ubuntuminion```


# 17. Bootstrapping and configuring Salt Master on ```ubuntumaster```, and Salt Minion on ```centosminion``` and ```ubuntuminion```

Navigate to address http://repo.saltstack.com/ to view the instructions for bootstrapping Salt Master on ```ubuntumaster```, and Salt Minion on ```centosminion``` and ```ubuntuminion```

Open the terminal in ```bustergraafinen``` and establish an SSH connection to ```ubuntumaster```

```
ssh -l oppilas 10.208.0.56
```

```
curl -L https://bootstrap.saltstack.com -o install_salt.sh
```

Install Salt Master

```
sudo sh install_salt.sh -P -M
```

```
sudo salt --versions-report
```

```
sudo nano /etc/salt/minion
```

Change ```# master: salt``` to ```master: localhost``` and remove the hashtag

```
##### Primary configuration settings #####
##########################################
# This configuration file is used to manage the behavior of the Salt Minion.
# With the exception of the location of the Salt Master Server, values that are
# commented out but have an empty line after the comment are defaults that need
# not be set in the config. If there is no blank line after the comment, the
# value is presented as an example and is not the default.

# Per default the minion will automatically include all config files
# from minion.d/*.conf (minion.d is a directory in the same directory
# as the main minion config file).
#default_include: minion.d/*.conf

# Set the location of the salt master server. If the master server cannot be
# resolved, then the minion will fail to start.
master: localhost

# Set http proxy information for the minion when doing requests
#proxy_host:
```

Change ```# id:``` to ```id: myminion``` and remove the hashtag

```
# The directory to store the pki information in
#pki_dir: /etc/salt/pki/minion

# Explicitly declare the id for this minion to use, if left commented the id
# will be the hostname as returned by the python call: socket.getfqdn()
# Since salt uses detached ids it is possible to run multiple minions on the
# same machine but with different ids, this can be useful for salt compute
# clusters.
id: myminion

# Cache the minion id to a file when the minion's id is not statically defined
# in the minion config. Defaults to "True". This setting prevents potential
# problems when automatic minion id resolution changes, which can cause the
# minion to lose connection with the master. To turn off minion id caching,
# set this config to ``False``.
#minion_id_caching: True
```

Restart Salt Minion

```
sudo service salt-minion restart
```

Restart Salt Master

```
sudo service salt-master restart
```

```
sudo salt-key -f myminion
```

```
sudo salt-call --local key.finger
```

```
sudo salt-key -L
```

```
sudo salt-key -A
```

Enter ```y``` to the following prompt

```
Proceed? [n/Y] y
```

```
sudo salt-key
```

Send a message to all the minions and tell them to return ```True``` to check which minions are alive

```
sudo salt '*' test.ping
```

```
sudo salt 'myminion' sys.list_functions test
```

```
sudo salt '*' test.fib
```

```
sudo salt '*' sys.doc test.fib
```

```
sudo salt '*' test.fib 30
```

```
sudo salt '*' sys.doc test
```

```
sudo salt '*' sys.list_functions sys
```

```
sudo salt '*' grains.get os
```

Find out the IP address of ```ubuntumaster```

```
ip address
```

Exit back to ```bustergraafinen``` terminal

```
exit
```

Open the terminal in ```bustergraafinen``` and establish an SSH connection to ```centosminion```

```
ssh -l root 10.208.0.55
```

```
curl -L https://bootstrap.saltstack.com -o install_salt.sh
```

Install Salt Minion

```
sudo sh install_salt.sh -P
```

Navigate inside ```/etc/salt/minion``` file for editing

```
sudo nano /etc/salt/minion
```

Remove the hashtag from ```# master: salt```

```
##### Primary configuration settings #####
##########################################
# This configuration file is used to manage the behavior of the Salt Minion.
# With the exception of the location of the Salt Master Server, values that are
# commented out but have an empty line after the comment are defaults that need
# not be set in the config. If there is no blank line after the comment, the
# value is presented as an example and is not the default.

# Per default the minion will automatically include all config files
# from minion.d/*.conf (minion.d is a directory in the same directory
# as the main minion config file).
#default_include: minion.d/*.conf

# Set the location of the salt master server. If the master server cannot be
# resolved, then the minion will fail to start.
master: salt

# Set http proxy information for the minion when doing requests
#proxy_host:
```

Navigate inside ```/etc/hosts``` file for editing

```
sudo nano /etc/hosts
```

Add new line ```10.208.0.56 salt``` inside the file

```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

127.0.1.1 centosminion
10.208.0.56 salt
```

Restart Salt Minion

```
sudo systemctl restart salt-minion
```

Exit back to ```bustergraafinen``` terminal

```
exit
```

Open the terminal in ```bustergraafinen``` and establish an SSH connection to ```ubuntuminion```

```
ssh -l oppilas 10.208.0.43
```

```
curl -L https://bootstrap.saltstack.com -o install_salt.sh
```

Install Salt Minion

```
sudo sh install_salt.sh -P
```

Navigate inside ```/etc/salt/minion``` file for editing

```
sudo nano /etc/salt/minion
```

Remove the hashtag from ```# master: salt```

```
##### Primary configuration settings #####
##########################################
# This configuration file is used to manage the behavior of the Salt Minion.
# With the exception of the location of the Salt Master Server, values that are
# commented out but have an empty line after the comment are defaults that need
# not be set in the config. If there is no blank line after the comment, the
# value is presented as an example and is not the default.

# Per default the minion will automatically include all config files
# from minion.d/*.conf (minion.d is a directory in the same directory
# as the main minion config file).
#default_include: minion.d/*.conf

# Set the location of the salt master server. If the master server cannot be
# resolved, then the minion will fail to start.
master: salt

# Set http proxy information for the minion when doing requests
#proxy_host:
```

Navigate inside ```/etc/hosts``` file for editing

```
sudo nano /etc/hosts
```

Add new line ```10.208.0.56 salt``` inside the file

```
127.0.0.1 localhost
127.0.1.1 ubuntuminion
10.208.0.56 salt

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

Restart Salt Minion

```
sudo service salt-minion restart
```

Exit back to ```bustergraafinen``` terminal

```
exit
```

After installing Salt Minion to ```centosminion``` and ```ubuntuminion```, we must accept them on ```ubuntumaster``` Salt Master

Open the terminal in ```bustergraafinen``` and establish an SSH connection to ```ubuntumaster```

```
ssh -l oppilas 10.208.0.56
```

List all public keys

```
sudo salt-key -L
```

Accept all pending keys

```
sudo salt-key -A
```

Enter ```y``` to the following prompt

```
The following keys are going to be accepted:
Unaccepted Keys:
centosminion
Proceed? [n/Y] y
```

Send a message to all the minions and tell them to return ```True``` to check which minions are alive

```
sudo salt '*' test.ping
```

Following text will appear

```
ubuntuminion:
    True
myminion:
    True
centosminion:
    True
```

Add user ```pekka``` to ```ubuntumaster```, ```centosminion```, and ```ubuntuminion```

```
sudo salt '*' user.add pekka shell=/bin/bash
```

Exit back to ```bustergraafinen``` terminal

```
exit
```

Salt Master is now bootstrapped and configured on ```ubuntumaster```, and Salt Minion is now bootstrapped and configured on ```centosminion``` and ```ubuntuminion```


# 18. Taking VM Snapshots in Rovius CP

Open VDI from address https://vdi.haaga-helia.fi if you are working outside of Haaga-Helia lab environment, and navigate to Rovius CP in address https://vdi-lab.cp.haaga-helia.fi/client/

Enter your ```Username``` > ```Password``` > ```Domain``` > ```Login``` - ```Username``` and ```Password``` are the same as in your student ID, and ```Domain``` is ```ITLAB```

Change ```Project``` to ```ICT4TN 022-3006 Pekka Hämäläinen``` from upper left corner

Take VM Snapshot from ```bustergraafinen```

- Go to ```Instances``` > ```bustergraafinen``` > ```Take VM Snapshot```
- Enter ```Name:``` > ```Description:``` > uncheck ```Snapshot memory:``` > ```OK```

Take VM Snapshot from ```ubuntussh```

- Go to ```Instances``` > ```ubuntussh``` > ```Take VM Snapshot```
- Enter ```Name:``` > ```Description:``` > uncheck ```Snapshot memory:``` > ```OK```

Take VM Snapshot from ```ubuntumaster```

- Go to ```Instances``` > ```ubuntumaster``` > ```Take VM Snapshot```
- Enter ```Name:``` > ```Description:``` > uncheck ```Snapshot memory:``` > ```OK```

Take VM Snapshot from ```centosminion```

- Go to ```Instances``` > ```centosminion``` > ```Take VM Snapshot```
- Enter ```Name:``` > ```Description:``` > uncheck ```Snapshot memory:``` > ```OK```

Take VM Snapshot from ```ubuntuminion```

- Go to ```Instances``` > ```ubuntuminion``` > ```Take VM Snapshot```
- Enter ```Name:``` > ```Description:``` > uncheck ```Snapshot memory:``` > ```OK```

Go to ```Instances``` > ```bustergraafinen```, ```ubuntussh```, ```ubuntumaster```, ```centosminion```, and ```ubuntuminion``` respectively > ```View Snapshots``` > check that VM Snapshots have been taken for the respective instances

VM Snapshots are now taken from instances ```bustergraafinen```, ```ubuntussh```, ```ubuntumaster```, ```centosminion```, and ```ubuntuminion```


# 19. Installing Visual Studio Code on ```bustergraafinen```

Navigate to address https://snapcraft.io/install/code/ubuntu to view the instructions for installing Visual Studio Code on ```bustergraafinen```

Open ```bustergraafinen``` terminal and update the package lists

```
sudo apt-get update
```

Install snapd

```
sudo apt-get install snapd
```

Install Visual Studio Code

```
sudo snap install code --classic
```

Go to Programming > ```Visual Studio Code``` > Extensions > Install ```SaltStack``` and ```salt-lint```

Visual Studio Code is now installed on ```bustergraafinen```


# 20. Changing SSH Server Port with Salt on ```ubuntuminion```

Navigate to address http://terokarvinen.com/2018/pkg-file-service-control-daemons-with-salt-change-ssh-server-port to view the instructions for changing SSH server port with Salt on ```ubuntuminion```

Open the terminal in ```bustergraafinen``` and establish an SSH connection to ```ubuntumaster```

```
ssh -l oppilas 10.208.0.56
```

Create ```/srv/salt``` directory

```
sudo mkdir /srv/salt
```

Create ```/srv/salt/sshd.sls``` file

```
sudo nano /srv/salt/sshd.sls
```

Copy the following text inside ```/srv/salt/sshd.sls``` file

```
openssh-server:
 pkg.installed
/etc/ssh/sshd_config:
 file.managed:
   - source: salt://sshd_config
sshd:
 service.running:
   - watch:
     - file: /etc/ssh/sshd_config
```

Copy ```/etc/ssh/sshd_config``` file to ```/srv/salt``` directory

```
sudo cp /etc/ssh/sshd_config /srv/salt
```

Edit ```/srv/salt/sshd_config``` file

```
sudo nano /srv/salt/sshd_config
```

Change line ```#Port 22``` to ```Port 8888```

```
#       $OpenBSD: sshd_config,v 1.101 2017/03/14 07:19:07 djm Exp $

# This is the sshd server system-wide configuration file.  See
# sshd_config(5) for more information.

# This sshd was compiled with PATH=/usr/bin:/bin:/usr/sbin:/sbin

# The strategy used for options in the default sshd_config shipped with
# OpenSSH is to specify options with their default value where
# possible, but leave them commented.  Uncommented options override the
# default value.

Port 8888
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::
```

Apply ```/srv/salt/sshd.sls``` state ONLY to ```ubuntuminion```

```
sudo salt 'ubuntuminion' state.apply sshd
```

The following output appears

```
ubuntuminion:
----------
          ID: openssh-server
    Function: pkg.installed
      Result: True
     Comment: All specified packages are already installed
     Started: 14:27:13.524117
    Duration: 81.499 ms
     Changes:   
----------
          ID: /etc/ssh/sshd_config
    Function: file.managed
      Result: True
     Comment: File /etc/ssh/sshd_config updated
     Started: 14:27:13.610800
    Duration: 73.37 ms
     Changes:   
              ----------
              diff:
                  --- 
                  +++ 
                  @@ -10,7 +10,7 @@
                   # possible, but leave them commented.  Uncommented options override the
                   # default value.
                   
                  -#Port 22
                  +Port 8888
                   #AddressFamily any
                   #ListenAddress 0.0.0.0
                   #ListenAddress ::
----------
          ID: sshd
    Function: service.running
      Result: True
     Comment: Service restarted
     Started: 14:27:13.784459
    Duration: 135.649 ms
     Changes:   
              ----------
              sshd:
                  True

Summary for ubuntuminion
------------
Succeeded: 3 (changed=2)
Failed:    0
------------
Total states run:     3
Total run time: 290.518 ms
```

Test the previously created state using ```ubuntuminion``` as a target 

```
nc -vz ubuntuminion 8888
```

Establish an SSH connection to ```ubuntuminion```

```
ssh -p 8888 oppilas@ubuntuminion
```

```
exit
```

```
sudo nano /srv/salt/sshd_config
```

```
#       $OpenBSD: sshd_config,v 1.101 2017/03/14 07:19:07 djm Exp $

# This is the sshd server system-wide configuration file.  See
# sshd_config(5) for more information.

# This sshd was compiled with PATH=/usr/bin:/bin:/usr/sbin:/sbin

# The strategy used for options in the default sshd_config shipped with
# OpenSSH is to specify options with their default value where
# possible, but leave them commented.  Uncommented options override the
# default value.

Port 22
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::

#HostKey /etc/ssh/ssh_host_rsa_key
#HostKey /etc/ssh/ssh_host_ecdsa_key
#HostKey /etc/ssh/ssh_host_ed25519_key

# Ciphers and keying
#RekeyLimit default none

# Logging
#SyslogFacility AUTH
#LogLevel INFO

# Authentication:

#LoginGraceTime 2m
```

```
sudo salt 'ubuntuminion' state.apply sshd
```

```
nc -vz ubuntuminion 22
```

```
ssh -l oppilas 10.208.0.43
```

SSH server port 8888 is now changed with Salt and connection is tested, after which SSH server port is reverted back to 22 with Salt on ```ubuntuminion```


# 21. Configuring nano text editor to support YAML and Jinja syntax highlighting on ```bustergraafinen```

Navigate to address https://ourcodeworld.com/articles/read/796/how-to-enable-syntax-highlighting-for-yaml-yml-files-in-gnu-nano to view the instructions for configuring nano text editor to support YAML and Jinja syntax highlighting on ```bustergraafinen```

```
ls /usr/share/nano/
```

```
sudo nano /usr/share/nano/yaml.nanorc
```

```
# Supports `YAML` files
syntax "YAML" "\.ya?ml$"
header "^(---|===)" "%YAML"

## Keys
color magenta "^\s*[\$A-Za-z0-9_-]+\:"
color brightmagenta "^\s*@[\$A-Za-z0-9_-]+\:"

## Values
color white ":\s.+$"
## Booleans
icolor brightcyan " (y|yes|n|no|true|false|on|off)$"
## Numbers
color brightred " [[:digit:]]+(\.[[:digit:]]+)?"
## Arrays
color red "\[" "\]" ":\s+[|>]" "^\s*- "
## Reserved
color green "(^| )!!(binary|bool|float|int|map|null|omap|seq|set|str) "

## Comments
color brightwhite "#.*$"

## Errors
color ,red ":\w.+$"
color ,red ":'.+$"
color ,red ":".+$"
color ,red "\s+$"

## Non closed quote
color ,red "['\"][^['\"]]*$"

## Closed quotes
color yellow "['\"].*['\"]"

## Equal sign
color brightgreen ":( |$)"
```

```
sudo nano testi.yml
```

```
# app/config/config_prod.yml
imports:
    - { resource: config.yml }

monolog:
    handlers:
        main:
            type:         fingers_crossed
            action_level: critical
            handler:      grouped
        grouped:
            type:    group
            members: [streamed, deduplicated]
        streamed:
            type:  stream
            path:  '%kernel.logs_dir%/%kernel.environment%.log'
            level: debug
        deduplicated:
            type:    deduplication
            handler: swift
        swift:
            type:       swift_mailer
            from_email: 'from_email@test.com'
            # Or multiple receivers:
            # to_email:   ['to_email1@ourcodeworld.com', 'to_email2@ourcodeworld.com']
            to_email:   'to_email@ourcodeworld.com'
            subject:    'An Error Occurred! %%message%%'
            level:      debug
            formatter:  monolog.formatter.html
            content_type: text/html
```


# 22. Installing Apache2 on ```ubuntuminion``` and ```centosminion``` with ```ubuntumaster``` Salt states (Work in progress)

Go to address https://www.linode.com/docs/applications/configuration-management/configure-apache-with-salt-stack/ to view the instructions for installing LAMP on ```ubuntuminion``` and ```centosminion``` with ```ubuntumaster``` Salt states

I managed to install Apache to ```ubuntuminion``` from ```ubuntumaster```, I'll write the instructions soon.

Open the terminal in ```bustergraafinen``` and establish an SSH connection to ```ubuntumaster```

```
ssh -l oppilas 10.208.0.56
```

Create ```/srv/salt``` directory if it does not already exist

```
sudo mkdir /srv/salt
```

Create ```/srv/salt/top.sls``` file

```
sudo nano /srv/salt/top.sls
```

Copy the following text inside ```/srv/salt/top.sls``` file

```
base:
  'G@os_family:Debian':
    - match: compound
    - apache-debian

  'G@os:CentOS':
    - match: compound
    - apache-centos
```

Create ```/srv/pillar``` directory

```
sudo mkdir /srv/pillar
```

Create ```/srv/pillar/top.sls``` file

```
sudo nano /srv/pillar/top.sls
```

Copy the following text inside ```/srv/pillar/top.sls``` file

```
base:
  '*':
    - apache
```

Create ```/srv/pillar/apache.sls``` file

```
sudo nano /srv/pillar/apache.sls
```

Copy the following text inside ```/srv/pillar/apache.sls``` file

```
domain: example.com
```

Create ```/srv/salt/example.com``` directory

```
sudo mkdir /srv/salt/example.com
```

Create ```/srv/salt/example.com/index.html``` file

```
sudo nano /srv/salt/example.com/index.html
```

Copy the following text inside ```/srv/salt/example.com/index.html``` file

```
<html>
  <body>
    <h1>Server Up and Running!</h1>
  </body>
</html>
```

Create ```/srv/salt/files``` directory

```
sudo mkdir /srv/salt/files
```

Create ```/srv/salt/files/tune_apache.conf``` file

```
sudo nano /srv/salt/files/tune_apache.conf
```

Copy the following text inside ```/srv/salt/files/tune_apache.conf``` file

```
<IfModule mpm_prefork_module>
StartServers 4
MinSpareServers 20
MaxSpareServers 40
MaxClients 200
MaxRequestsPerChild 4500
</IfModule>
```

Create ```/srv/salt/files/include_sites_enabled.conf``` file

```
sudo nano /srv/salt/files/include_sites_enabled.conf
```

Copy the following text inside ```/srv/salt/files/include_sites_enabled.conf``` file

```
IncludeOptional sites-enabled/*.conf
```

Create ```/srv/salt/apache-debian.sls``` file for ```ubuntuminion```

```
sudo nano /srv/salt/apache-debian.sls
```

Copy the following text inside ```/srv/salt/apache-debian.sls``` file

```
apache2:
  pkg.installed

apache2 Service:
  service.running:
    - name: apache2
    - enable: True
    - require:
      - pkg: apache2

Turn Off KeepAlive:
  file.replace:
    - name: /etc/apache2/apache2.conf
    - pattern: 'KeepAlive On'
    - repl: 'KeepAlive Off'
    - show_changes: True
    - require:
      - pkg: apache2

/etc/apache2/conf-available/tune_apache.conf:
  file.managed:
    - source: salt://files/tune_apache.conf
    - require:
      - pkg: apache2

Enable tune_apache:
  apache_conf.enabled:
    - name: tune_apache
    - require:
      - pkg: apache2

/var/www/html/{{ pillar['domain'] }}:
  file.directory

/var/www/html/{{ pillar['domain'] }}/log:
  file.directory

/var/www/html/{{ pillar['domain'] }}/backups:
  file.directory

/var/www/html/{{ pillar['domain'] }}/public_html:
  file.directory

000-default:
  apache_site.disabled:
    - require:
      - pkg: apache2

/etc/apache2/sites-available/{{ pillar['domain'] }}.conf:
  apache.configfile:
    - config:
      - VirtualHost:
          this: '*:80'
          ServerName:
            - {{ pillar['domain'] }}
          ServerAlias:
            - www.{{ pillar['domain'] }}
          DocumentRoot: /var/www/html/{{ pillar['domain'] }}/public_html
          ErrorLog: /var/www/html/{{ pillar['domain'] }}/log/error.log
          CustomLog: /var/www/html/{{ pillar['domain'] }}/log/access.log combined

{{ pillar['domain'] }}:
  apache_site.enabled:
    - require:
      - pkg: apache2

/var/www/html/{{ pillar['domain'] }}/public_html/index.html:
  file.managed:
    - source: salt://{{ pillar['domain'] }}/index.html
```

Create ```/srv/salt/apache-centos.sls``` file for ```centosminion```

```
sudo nano /srv/salt/apache-centos.sls
```

Copy the following text inside ```/srv/salt/apache-centos.sls``` file

```
httpd:
  pkg.installed

httpd Service:
  service.running:
    - name: httpd
    - enable: True
    - require:
      - pkg: httpd
    - watch:
      - file: /etc/httpd/sites-available/{{ pillar['domain'] }}.conf

Turn off KeepAlive:
  file.replace:
    - name: /etc/httpd/conf/httpd.conf
    - pattern: 'KeepAlive On'
    - repl: 'KeepAlive Off'
    - show_changes: True
    - require:
      - pkg: httpd

Change DocumentRoot:
  file.replace:
    - name: /etc/httpd/conf/httpd.conf
    - pattern: 'DocumentRoot "/var/www/html"'
    - repl: 'DocumentRoot "/var/www/html/{{ pillar['domain'] }}/public_html"'
    - show_changes: True
    - require:
      - pkg: httpd

/etc/httpd/conf.d/tune_apache.conf:
  file.managed:
    - source: salt://files/tune_apache.conf
    - require:
      - pkg: httpd

/etc/httpd/conf.d/include_sites_enabled.conf:
  file.managed:
    - source: salt://files/include_sites_enabled.conf
    - require:
      - pkg: httpd

/etc/httpd/sites-available:
  file.directory

/etc/httpd/sites-enabled:
  file.directory

/var/www/html/{{ pillar['domain'] }}:
  file.directory

/var/www/html/{{ pillar['domain'] }}/backups:
  file.directory

/var/www/html/{{ pillar['domain'] }}/public_html:
  file.directory

/etc/httpd/sites-available/{{ pillar['domain'] }}.conf:
  apache.configfile:
    - config:
      - VirtualHost:
          this: '*:80'
          ServerName:
            - {{ pillar['domain'] }}
          ServerAlias:
            - www.{{ pillar['domain'] }}
          DocumentRoot: /var/www/html/{{ pillar['domain'] }}/public_html
  file.symlink:
    - target: /etc/httpd/sites-enabled/{{ pillar['domain'] }}.conf
    - force: True

/var/www/html/{{ pillar['domain'] }}/public_html/index.html:
  file.managed:
    - source: salt://{{ pillar['domain'] }}/index.html

Configure Firewall:
  firewalld.present:
    - name: public
    - ports:
      - 22/tcp
      - 80/tcp
      - 443/tcp
```

Apply the created states to ```ubuntuminion``` and ```centosminion```

```
sudo salt '*' state.apply
```

Apache was installed successfully on ```ubuntuminion```, but for some reason the installation failed on ```centosminion``` - to be continued - httpd instead of Apache2?


# 23. Changing the hostname permanently on ```bustergraafinen```

Go to address https://www.cyberciti.biz/faq/ubuntu-18-04-lts-change-hostname-permanently/ to view the instructions for changing ```bustergraafinen``` hostname permanently

Replace the old hostname ```deb10xfcews1``` with ```bustergraafinen``` 

```
sudo hostnamectl set-hostname bustergraafinen
```

Check the new hostname

```
hostname
```

Edit ```/etc/hosts``` file

```
sudo nano /etc/hosts
```

Replace the old hostname ```deb10xfcews1``` after the line ```127.0.1.1``` with the new hostname ```bustergraafinen``` 

```
127.0.0.1       localhost
127.0.1.1       deb10xfcews1.haagahelia.amk     bustergraafinen

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

Reboot the system

```
sudo reboot
```

The hostname is now changed permanently on ```bustergraafinen```


# 24. Installing cowsay on ```ubuntuminion``` and ```centosminion``` with ```ubuntumaster``` Salt state

Open the terminal in ```bustergraafinen``` and establish an SSH connection to ```ubuntumaster```

```
ssh -l oppilas 10.208.0.56
```

Install Tree, a recursive directory listing program that produces a depth-indented listing of files

```
sudo apt-get install tree
```

Test Tree

```
tree /home/
```

The following output appears

```
/home/
├── oppilas
│   └── install_salt.sh
└── pekka

2 directories, 1 file
```

Create ```/srv/salt/cowsanoopi.sls``` file

```
sudo nano /srv/salt/cowsanoopi.sls
```

Copy the following text inside ```/srv/salt/cowsanoopi.sls``` file 

```
cowsay:
  pkg.installed
```

Apply ```/srv/salt/cowsanoopi.sls``` state

```
sudo salt '*' state.apply cowsanoopi test=True
```

Execute cowsay on Salt Minions, but replace ```runas=pekka``` with your own user on Salt Minions

```
sudo salt '*' cmd.run 'cowsay "Kukkuu -salttia pukkaa"' runas=pekka
```

The following output appears

```
ubuntuminion:
     ________________________
    < Kukkuu -salttia pukkaa >
     ------------------------
            \   ^__^
             \  (oo)\_______
                (__)\       )\/\
                    ||----w |
                    ||     ||
myminion:
     ________________________
    < Kukkuu -salttia pukkaa >
     ------------------------
            \   ^__^
             \  (oo)\_______
                (__)\       )\/\
                    ||----w |
                    ||     ||
centosminion:
     ________________________
    < Kukkuu -salttia pukkaa >
     ------------------------
            \   ^__^
             \  (oo)\_______
                (__)\       )\/\
                    ||----w |
                    ||     ||
```

cowsay is now installed on ```ubuntuminion``` and ```centosminion``` with ```ubuntumaster``` Salt states


# 25. Adding the roles of ```ubuntumaster```, ```centosminion```, and ```ubuntuminion``` into the grains

Go to address https://docs.saltstack.com/en/latest/topics/grains/ to view the instructions for adding the roles of ```ubuntumaster```, ```centosminion```, and ```ubuntuminion``` into the grains

Open the terminal in ```bustergraafinen``` and establish an SSH connection to ```ubuntumaster```

```
ssh -l oppilas 10.208.0.56
```

Create ```/etc/salt/grains``` file

```
sudo nano /etc/salt/grains
```

Copy the following text inside ```/etc/salt/grains``` file

```
roles:
  - SaltMaster1
```

Establish an SSH connection to ```centosminion```

```
ssh -l root 10.208.0.55
```

Create ```/etc/salt/grains``` file

```
sudo nano /etc/salt/grains
```

Copy the following text inside ```/etc/salt/grains``` file

```
roles:
  - DB_tier
```

Establish an SSH connection to ```ubuntuminion```

```
ssh -l oppilas 10.208.0.43
```

Create ```/etc/salt/grains``` file

```
sudo nano /etc/salt/grains
```

Copy the following text inside ```/etc/salt/grains``` file

```
roles:
  - Web_tier
```

Establish an SSH connection to ```ubuntumaster```

```
ssh -l oppilas 10.208.0.56
```

Restart Salt Minions

```
sudo salt '*' cmd.run 'systemctl restart salt-minion'
```

Check the roles of Salt Minions

```
sudo salt '*' grains.item roles
```

The following output appears

```
ubuntuminion:
    ----------
    roles:
        - Web_tier
myminion:
    ----------
    roles:
        - SaltMaster1
centosminion:
    ----------
    roles:
        - DB_tier
```

The roles of ```ubuntumaster```, ```centosminion```, and ```ubuntuminion``` are now added into the grains


# 26. Installing Apache2, Vim, Links, Wget, cURL, and tmux on ```ubuntuminion``` and ```centosminion``` with ```ubuntumaster``` Salt states

Open the terminal in ```bustergraafinen``` and establish an SSH connection to ```ubuntumaster```

```
ssh -l oppilas 10.208.0.56
```

Create ```/srv/salt/DeployToUbuntu.sls``` file

```
sudo nano /srv/salt/DeployToUbuntu.sls
```

Copy the following text inside ```/srv/salt/DeployToUbuntu.sls``` file

```
apache2:
  pkg.installed

vim:
  pkg.installed

links:
  pkg.installed

wget:
  pkg.installed

curl:
  pkg.installed

tmux:
  pkg.installed
```

Apply ```/srv/salt/DeployToUbuntu.sls``` state

```
sudo salt '*' state.apply DeployToUbuntu
```

The output shows that all installations succeeded on ```ubuntuminion```

```
Summary for ubuntuminion
------------
Succeeded: 6
Failed:    0
------------
Total states run:     6
Total run time: 248.056 ms
```

The output shows that all installations succeeded on the local Salt Minion of ```ubuntumaster```

```
Summary for myminion
------------
Succeeded: 6
Failed:    0
------------
Total states run:     6
Total run time: 635.518 ms
```

Create ```/srv/salt/DeployToCentos.sls``` file

```
sudo nano /srv/salt/DeployToCentos.sls
```

Copy the following text inside ```/srv/salt/DeployToCentos.sls``` file

```
httpd:
  pkg.installed

vim-enhanced:
  pkg.installed

links:
  pkg.installed

wget:
  pkg.installed

curl:
  pkg.installed

tmux:
  pkg.installed
```

Apply ```/srv/salt/DeployToCentos.sls``` state

```
sudo salt '*' state.apply DeployToCentos
```

The output shows that all installations succeeded on ```centosminion```

```
Summary for centosminion
------------
Succeeded: 6
Failed:    0
------------
Total states run:     6
Total run time:   3.850 s
```

Apache2, Vim, Links, Wget, cURL, and tmux are now installed on ```ubuntuminion``` and ```centosminion``` with ```ubuntumaster``` Salt states


# 27. Installing MariaDB instead of MySQL on ```centosminion``` with ```ubuntumaster``` Salt state

I didn't find a workable way to install MySQL on ```centosminion```, so instead, I installed MariaDB which is a commercially supported fork of MySQL

Open the terminal in ```bustergraafinen``` and establish an SSH connection to ```ubuntumaster```

```
ssh -l oppilas 10.208.0.56
```

Create ```/srv/salt/SQL.sls``` file

```
sudo nano /srv/salt/SQL.sls
```

Copy the following text inside ```/srv/salt/SQL.sls``` file

```
mariadb:
  pkg.installed
```

Apply ```/srv/salt/SQL.sls``` state

```
sudo salt 'centosminion' state.apply SQL
```

The following output appears

```
centosminion:
----------
          ID: mariadb
    Function: pkg.installed
      Result: True
     Comment: All specified packages are already installed
     Started: 21:22:33.202503
    Duration: 3527.107 ms
     Changes:

Summary for centosminion
------------
Succeeded: 1
Failed:    0
------------
Total states run:     1
Total run time:   3.527 s
```

Start MariaDB

```
sudo salt 'centosminion' cmd.run 'systemctl start mariadb'
```

Establish an SSH connection to ```centosminion```

```
ssh -l root 10.208.0.55
```

Open MariaDB

```
sudo mysql
```

The following output appears

```
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 7
Server version: 5.5.64-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]>
```

MariaDB is now installed on ```centosminion``` with ```ubuntumaster``` Salt state


# 28. Creating four users on ```ubuntuminion``` and ```centosminion``` with ```ubuntumaster``` Salt state (Work in progress)

Delete the previously created user with your first name, ```pekka```, because it will be created again with its first letter as a capital letter

```
sudo salt '*' user.delete pekka remove=True force=True
```

The following output appears

```
ubuntuminion:
    True
myminion:
    True
centosminion:
    True
```

Check that user ```pekka``` was deleted

```
sudo salt '*' user.info pekka
```

