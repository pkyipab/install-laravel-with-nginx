Last Update date : 21 - 08 - 2021

# Installation-laravel-with-nginx
Complete guide for installing web server & laravel on Ubuntu 20.4

# 0. (Optional) Config the SSH access to server [macOS]

### Alternative 1. Connect to server

```
ssh -i [YOUR_PEM_NAME].pem ubuntu@xx.xx.xx.xx
```

### Alternative 2. Quick connection setup for ssh & vscode using .pem

Move the .pem file under .ssh directory

```
mv ~/.ssh/[YOUR_PEM_NAME].pem  ~/[YOUR_PEM_NAME].pem
```

Store your connection setting

```
vi ~/.ssh/config
```

Insert the below code to .config file

```
Host sampleHostName
    HostName xx.xx.xx.xx
    User ubuntu
    IdentityFile ~/.ssh/[YOUR_PEM_NAME].pem
```

Now, you can simply connect to server with:

```
ssh sampleHostName
```

### Alternative 3. Quick connection setup for ssh & vscode using password

Store your connection setting

```
vi ~/.ssh/config
```
Insert the below code to .config file

```
Host sampleHostName
  HostName xx.xx.xx
  User root
```

Copy your public key to grant the access permssion

```
ssh-copy-id -i ~/.ssh/id_rsa.pub root@xx.xx.xx.xx
```

Now, you can simply connect to server with:

```
ssh sampleHostName
```

### Troubleshoot : 

#### Issue 1: If you encounter unprotected issus
```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
```
Run :

```
chmod 400 [YOUR_PEM_NAME].pem
```

#### Issue 2 : Missing public & private key ( lost.id_rsa or id_rsa.pub file )

Run the below code first & retry

```
ssh-keygen
```

# 1. Setup PHP Modules & fastCGI

### Installing PHP

For the PHP installation we recommend to use ppa:ondrej/php PPA, which provides latest PHP versions for Ubuntu systems. 
Use the below couple of commands to add the PPA to your system.

```
sudo apt install software-properties-common

sudo add-apt-repository ppa:ondrej/php
```


Then install PHP 7.4 the latest version available on the day of writing this tutorial. 
Simply execute follows commands for the installation of PHP and PHP-FPM packages.


```
sudo apt update

sudo apt install php7.4 php7.4-fpm

sudo apt install php7.4-common php7.4-mysql php7.4-xml php7.4-xmlrpc php7.4-curl php7.4-gd php7.4-imagick php7.4-cli php7.4-dev php7.4-imap php7.4-mbstring php7.4-opcache php7.4-soap php7.4-zip php7.4-intl -y
```

# 2. Setup Nginx & ufw

### Installing Nginx

Nginx packages are available under default repositories. SSH to your Ubuntu 20.04 LTS system with sudo privileges account and install Nginx web server from the official repository.

```
sudo apt-get install nginx ufw -y
```

### Enable ufw Firewall Defaults
We want ssh and Nginx to work. Nginx Full allows for both http (80) and https (443) connections.
Ensure you have ssh above otherwise you will lose your connection and not get it back.

```
sudo ufw allow ssh

sudo ufw allow 'Nginx Full'

sudo ufw enable
```

Check your status

```
sudo ufw status numbered
```

# 3. NGINX Config

### Configuring NGINX with FPM

Next, create a Nginx server block configuration file to run PHP with FPM. 
Create and Edit a VirtualHost host configuration file in a text editor. 
You can create new VirtualHost as per your requirements, so make sure to enable any new VirtualHost.

```
sudo vim /etc/nginx/sites-available/[YOUR_DOMIAN_NAME].com 
```

Use the below basic Nginx Virtual host configuration with php fpm settings. Update the configuration as followings.

```
server {
        listen 80;
        root [PATH_TO_YOUR_INDEX_FILE];
        index index.php index.html index.htm;
        server_name [YOUR_DOMIAN_NAME].com;

        location / {
            root [PATH_TO_YOUR_INDEX_FILE];
            index index.php index.html index.htm;
            try_files $uri $uri/ /index.php$is_args$args;
        }

        location ~ \.php$ {
            include snippets/fastcgi-php.conf;
            fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
        }
}
```

Save your changes to the configuration file and create a link to site enabled directory.

```
sudo ln -s /etc/nginx/sites-available/[YOUR_DOMIAN_NAME].com /etc/nginx/sites-enabled/[YOUR_DOMIAN_NAME].com 
```

Then restart Nginx service to reload the changes.

```
sudo systemctl restart nginx
```

# 4. Setup SSL with certBot

### Ensure that your version of snapd is up to date
Execute the following instructions on the command line on the machine to ensure that you have the latest version of snapd.

```
sudo snap install core;

sudo snap refresh core
```

### Install Certbot
Run this command on the command line on the machine to install Certbot.

```
sudo snap install --classic certbot
```

### Prepare the Certbot command
Execute the following instruction on the command line on the machine to ensure that the certbot command can be run.

```
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

### Run Certbot
Run this command to get a certificate and have Certbot edit your Nginx configuration automatically to serve it, turning on HTTPS access in a single step.

```
sudo certbot --nginx
```
### Automatic renewal

The Certbot packages on your system come with a cron job or systemd timer that will renew your certificates automatically before they expire. You will not need to run Certbot again, unless you change your configuration.
```
sudo certbot renew --dry-run
```

# 5. Setup MariaDB

###  Installing MariaDB
As of this writing, Ubuntu 20.04’s default APT repositories include MariaDB version 10.3.

To install it, update the package index on your server with apt & install package

```
sudo apt update

sudo apt install mariadb-server

```

### Configuring MariaDB
For new MariaDB installations, the next step is to run the included security script. This script changes some of the less secure default options for things like remote root logins and sample users.

Run the security script:

```
sudo mysql_secure_installation
```

### Creating an Administrative User that Employs Password Authentication

We will create a new account called admin with the same capabilities as the root account, but configured for password authentication. 
Open up the MariaDB prompt from your terminal:

```
sudo mariadb
```

Then create a new user with root privileges and password-based access. Be sure to change the username and password to match your preferences:

```
MariaDB[(none)] > GRANT ALL ON *.* TO 'root'@'localhost' IDENTIFIED BY 'Password' WITH GRANT OPTION;
MariaDB[(none)] > GRANT ALL ON *.* TO ubuntu@'localhost' IDENTIFIED BY 'Password' WITH GRANT OPTION;
MariaDB[(none)] > FLUSH PRIVILEGES;
MariaDB[(none)] > exit;
```

### Login to mariadb
```
mysql -uubuntu -pPassword
```

# 6. Setup Composer 

```
cd ~

curl -sS https://getcomposer.org/installer -o composer-setup.php

HASH=`curl -sS https://composer.github.io/installer.sig`

php -r "if (hash_file('SHA384', 'composer-setup.php') === '$HASH') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"

sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer
```

# 7. Setup NodeJS

```
cd ~

curl -sL https://deb.nodesource.com/setup_16.x -o nodesource_setup.sh

sudo bash nodesource_setup.sh

sudo apt install nodejs
```

# 8. Setup Yarn

```
sudo apt install curl

curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -

echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list

sudo apt update

sudo apt install yarn
```

# 9. Start Laravel

### Installation

Open the terminal in your root directory(vuexy-bootstrap-laravel-admin-template) & to install the composer packages run the following command

```
composer install
```

In the root directory, you will find a file named .env.example, rename the given file name to .env and run the following command to generate the key (and you can also edit your database credentials here).

```
php artisan key:generate‬‬
```

By running the following command, you will be able to get all the dependencies in your node_modules folder

```
yarn
```

If encounter : The engine "node" is incompatible with this module, run :

```
yarn --ignore-engines
```

To run the project, you need to run following command in the project directory. It will compile the php files & all the other project files. 
If you are making any changes in any of the php file then you need to run the given command again.

```
yarn mix
```

To serve the application you need to run the following command in the project directory. (This will give you an address with port number 8000.)

Now navigate to the given address you will see your application is running.

```
php artisan serve
```

(Optional) To change the port address, run the following command: 




```
php artisan serve --port=8080 // For port 8080
sudo php artisan serve --port=80 // If you want to run it on port 80, you probably need to sudo.
```
# 10. Useful command

### Permission Update

```
sudo chown -R ubuntu /home
sudo chmod -R 755 /home
```

### Listing services
```
sudo systemctl list-unit-files
```

### Service control
```
sudo systemctl start nginx      //Start Nginx service
sudo systemctl restart nginx    // Restart Nginx service
```
