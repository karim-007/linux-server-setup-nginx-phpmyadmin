# Linux server setup with php, Nginx, Mariadb, Phpmyadmin

## login in to your server

```bash
ssh username@ip_or_host_name

#update your server
sudo apt update
```

### Install Nginx

```bash
    sudo apt install nginx
```


### Enable ufw

```bash
    sudo ufw allow 'Nginx HTTP'
    sudo ufw allow 'Nginx HTTPS'
    sudo ufw allow 'OpenSSH'
    sudo ufw enable
    sudo ufw status
    sudo ufw app list #enbaled service list show
```


### MariaDB installation

```bash
    sudo apt install mariadb-server
    sudo mysql_secure_installation
    sudo systemctl status mariadb
```

### install php

```bash
    #you can install any version from there
    #for php 7.4
    sudo apt install php7.4-common php7.4-mysql php7.4-xml php7.4-xmlrpc php7.4-curl php7.4-gd php7.4-imagick php7.4-cli php7.4-dev php7.4-imap php7.4-mbstring php7.4-opcache php7.4-soap php7.4-zip php7.4-intl -y
    sudo apt install php7.4-fpm
    #for php 8.1
    sudo apt-get install -y php8.1-cli php8.1-common php8.1-mysql php8.1-zip php8.1-gd php8.1-mbstring php8.1-curl php8.1-xml php8.1-bcmath
    sudo apt install php8.1-fpm
```


### fpm status
```bash
  #it will show enabled php-fpm status
  sudo systemctl status php*-fpm.service
```

### now you will see the nginx default page
```bash
  your_ip #got to your browser type ip and hit enter then you will see the nginx default page 
```

### phpmyadmin Installation
```bash
  sudo apt install phpmyadmin
```

### now need to create database user

```bash

  #instade of your_user_name use your on username and your_password use your own password
  sudo mysql -u root
  USE mysql;
  CREATE USER 'your_user_name'@'localhost' IDENTIFIED BY 'your_password';
  GRANT ALL PRIVILEGES ON *.* TO 'your_user_name'@'localhost';
  #update auth_socket to mysql_native_password
  UPDATE user SET plugin='mysql_native_password' WHERE User='your_user_name';
  FLUSH PRIVILEGES;
  #user list show
  SELECT user FROM mysql.user;
  exit;
```

### you can now access your database and update/edit/import/export anything you can do
```bash
#your_user_name is previous created username
mysql -u your_user_name -p

#databse create
CREATE DATABASE your_databse_name;

#max allow you can incress
set global net_buffer_length=1000000;
set global max_allowed_packet=1000000000;
SET foreign_key_checks = 0;

#for use database
use DATABASE_NAME;

source PATH_TO_.sql; #import database

SET foreign_key_checks = 1; #after import enable forign key
```

### phpmyadmin link setup

```bash
    sudo ln -s /usr/share/phpmyadmin/ /var/www/html/
    sudo nginx -t
    sudo systemctl reload nginx
```

### nginx config edit
```bash
  sudo nano nano /etc/nginx/sites-available/default

    # Add index.php to the list if you are using PHP
    index index.php index.html index.htm index.nginx-debian.html;
    
    #add bellow line to your server block
	location ~ \.php$ {
        try_files $fastcgi_script_name =404;
        include fastcgi_params;
        fastcgi_pass  unix:/run/php/php7.4-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param DOCUMENT_ROOT  $realpath_root;
        fastcgi_param SCRIPT_FILENAME   $realpath_root$fastcgi_script_name;
    }

  sudo nginx -t #if have any error it will show
  sudo systemctl restart nginx
```

### now you will see the phpmyadmin
```bash
  your_ip/phpmyadmin #got to your browser type ip/phpmyadmin and hit enter then you will see the phpmyadmin login page 
  #till you dont see phpmyadmin then follow the next step
```


### create phpmyadmin.conf

```bash
sudo nano /etc/nginx/snippets/phpmyadmin.conf


#past bellow line and save and exit
location /phpmyadmin {
    root /usr/share/;
    index index.php index.html index.htm;
    location ~ ^/phpmyadmin/(.+\.php)$ {
    try_files $uri =404;
    root /usr/share/;
    fastcgi_pass unix:/run/php/php8.0-fpm.sock;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include /etc/nginx/fastcgi_params;
}

    location ~* ^/phpmyadmin/(.+\.(jpg|jpeg|gif|css|png|js|ico|html|xml|txt))$ {
        root /usr/share/;
    }
}
```

#Edit your server block configuration which will be located inside /etc/nginx/sites-available and include the snippet so your configuration looks something similar to the one below.
```bash
  sudo nano /etc/nginx/sites-available
  #in server block past include snippets/phpmyadmin.conf;
  # it will show like this
    server {
    . . .
    
      include snippets/phpmyadmin.conf;
    
    . . .
    }
    
    sudo nginx -t #if have any error it will show
    sudo systemctl restart nginx #restart the server
```

### now you will see the phpmyadmin
```bash
  your_ip/phpmyadmin #got to your browser type ip/phpmyadmin and hit enter then you will see the phpmyadmin login page 
  #till you dont see phpmyadmin then follow the next step
```

### file upload size change
```bash
sudo nano /etc/php/7.4/fpm/php.ini
#find and change following this
memory_limit = 512M 
post_max_size = 50M  
upload_max_filesize = 50M
sudo service nginx restart
```

### server security setup

```bash
sudo apt install unattended-upgrades
sudo dpkg-reconfigure --priority=low unattended-upgrades

cd /etc/apt/apt.conf.d/
cat 20auto-upgrades #you can customize your need wise
sudo unattended-upgrade --dry-run --debug
```

### install black list ip blocker
```bash
ss -ltpn
sudo apt install fail2ban
sudo systemctl enable fail2ban --now
sudo systemctl status fail2ban
sudo fail2ban status
sudo fail2ban-client status
sudo fail2ban-client status sshd
sudo apparmor_status
sudo fail2ban-client status sshd
```

### certboat ssl install nginx
```bash
sudo apt install certbot python3-certbot-nginx
sudo ufw allow 'Nginx Full'
sudo ufw delete allow 'Nginx HTTP'
sudo ufw status
sudo certbot --nginx
```

### cron tab
```bash
crontab -e
crontab -l
sudo /etc/init.d/cron restart
```

### supervisor setup
```bash
cd /etc/supervisor/conf.d
sudo supervisorctl status
sudo supervisorctl restart process_name
```

### Basic info view
```bash
#Check free memory before
free -m

#disk use
df -h

#ind directory size
du -hs /path/to/directory

#single dev size view
du -h --max-depth=1

#details view
du -h | sort -h
```

### Swap memory create
```bash
mkdir -p /var/_swap_
cd /var/_swap_

#Here, 2G ~ 2GB of swap memory. Feel free to add MORE
sudo fallocate -l 2G swapfile

chmod 600 swapfile
mkswap swapfile
swapon swapfile
#Automatically mount this swap partition at startup
echo "/var/_swap_/swapfile none swap sw 0 0" >> /etc/fstab

#Check free memory after
free -m
```

### git repo dublicate
```bash
https://docs.github.com/en/repositories/creating-and-managing-repositories/duplicating-a-repository
git clone --bare githubrepurl
git push --mirror githubrepurl
```

### gitub url change
```bash
git remote set-url origin githubrepurl
or
git remote add origin githubrepurl
git remote -v
```


Copyright 2022 [md abdul karim](https://github.com/karim-007).
