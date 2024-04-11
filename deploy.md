# Setup laravel ec2

## Install ec2-instance-connect to allow access instance from AWS EC2 instance console
```
yum install -y ec2-instance-connect
```



## Instal environmental GIT, PHP, PHP_FPM, NGINX
### Install git
```
yum install -y git
```

### Install nginx
```
amazon-linux-extras list | grep nginx
sudo amazon-linux-extras install nginx1
```

### Install php & php-extensions
```
sudo yum -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
sudo yum -y install https://rpms.remirepo.net/enterprise/remi-release-7.rpm
sudo yum -y install yum-utils
sudo yum-config-manager --disable 'remi-php*'
sudo yum-config-manager --enable remi-php81
sudo yum -y install php81
sudo yum -y install php81-php-{fpm,pdo,pdo_mysql,dom,bcmath,xml,mbstring,gd}
php81 -v
systemctl restart php81-php-fpm (Restart php-fpm after install new php-ext)
```

#### Install ImageMagic & php-imagick
```
wget http://mirror.centos.org/centos/7/os/x86_64/Packages/LibRaw-0.19.4-1.el7.x86_64.rpm
yum install LibRaw
rpm -i LibRaw-0.19.4-1.el7.x86_64.rpm
sudo yum -y install php81-php-imagick
systemctl restart php81-php-fpm (Restart php-fpm after install new php-ext)
```

### Install composer
```
php81 -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php81 -r "if (hash_file('sha384', 'composer-setup.php') === '55ce33d7678c5a611085589f1f3ddf8b3c52d662cd01d4ba75c0ee0459970c2200a51f492d557530c71c15d8dba01eae') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php81 composer-setup.php
php81 -r "unlink('composer-setup.php');"
sudo mv composer.phar /usr/local/bin/composer
```

### Install wkhtmltopdf
```
wget https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6-1/wkhtmltox-0.12.6-1.centos7.x86_64.rpm
sudo yum localinstall wkhtmltox-0.12.6-1.centos7.x86_64.rpm
```

### Setting alias php81 -> php (Apply for current logged user)
```
vim /.bash_profile
```
Add two lines below to at the end of file
```
alias php='/usr/bin/php81'
alias composer='php81 /usr/local/bin/composer'
```

Logout and check composer alias
```
exit
sudo su - (switch user root)
composer -v
```
---


## Create user developer and decentralization configuration
### Create giang-vu user (switch to developer using password)
```
useradd -m giang-vu
passwd -e giang-vu (user will self update password)
```

### Create developer user
```
useradd -m developer (Create user with *mkdir* home directory)
passwd developer
sudo su - developer
```

### Create ssh file pem and generate key
#### Create .pem file
Go to ec2[https://ap-northeast-1.console.aws.amazon.com/ec2/v2/home?region=ap-northeast-1#KeyPairs:] to create pem file.

#### Generate public key from .pem file
```
chmod 700 /path/to/file.pem
ssh-keygen -y -f /path/to/file.pem
```
#### Create public key on ec2 server
```
1. Login ec2
2. cd /home/developer/.ssh
3. vim authorized_keys
4. paste public key is generated from .pem file
5. chmod 600 /home/developer/.ssh/authorized_keys
```

---
## Init project
### Create deploy key
1. Generate deploy key on ec2 with developer user
```
ssh-keygen -t ed25519 -C "<comment>"
vim .ssh/id_ed25519.pub
```
2. Go to setting project git repository
3. Open tab deploy key
4. Add ssh public key

### Init project
```
git clone git@github.com:ImplVN/KetHon-API.git
cd KetHon-API
composer install
cp .env.dev .env
php artisan key:gen

cd storage/app
chmod 770 -R public
mkdir -p private/cv
mkdir private/zip
```

### Grant permission to access KetHon folder for nginx user
```
usermod -a -G developer nginx
groups nginx
chmod 711 /home/developer
sudo -u nginx stat /home/developer/KetHon-API/public/index.php
cd /home/developer/building-survey-report-system-be/
chmod 755 -R storage/
```

### Change service running as developer (current running as apache/nginx)
```
vim /etc/opt/remi/php81/php-fpm.d/www.conf
user=developer
group=developer
```
### Add developer to apache group to write log files
```
sudo usermod -a -G apache developer
groups developer
```

### Add apache to developer group to write log files
```
sudo usermod -a -G developer apache
groups developer
```

### Config nginx
cd /etc/nginx/conf.d
vim marriage.conf

```
server {
    server_name 3.114.52.102;
    root /home/developer/KetHon-API/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    index index.php;

    charset utf-8;

    ## All static files will be served directly.
    location ~* ^.+\.(?:css|js|jpe?g|gif|ico|png|)$ {
        access_log off;
        expires 30d;
        add_header Cache-Control public;

        ## No need to bleed constant updates. Send the all shebang in one
        ## fell swoop.
        tcp_nodelay off;

        ## Set the OS file cache.
        open_file_cache max=3000 inactive=120s;
        open_file_cache_valid 45s;
        open_file_cache_min_uses 2;
        open_file_cache_errors off;
    }

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
            try_files $uri $uri/ /index.php$is_args$args =404;
			include fastcgi_params;
			fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
			# fastcgi_pass unix:/run/php-fpm/www.sock;
			fastcgi_pass 127.0.0.1:9000;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

### Check nginx configuration and restart nginx service
```
nginx -t
systemctl restart nginx
```

### Start php-fpm service

```
systemctl start php81-php-fpm
```

## Add alias for php & composer
```
vim ~/.bash_profile
```

Add two lines below to at the end of file
```
alias php='/usr/bin/php81'
alias composer='php81 /usr/local/bin/composer'
```

## Install php-ext for laravel-snappy (export PDF) (Init)
```
yum -y install libXrender-devel
yum -y install libXext.x86_64
yum -y install libXrender.x86_64
yum -y install libXtst.x86_64
yum -y install wqy-microhei-fonts
fc-cache -f -v
yum install -y ghostscript
```

## Generate cloudfront key

## Setup cronjob for clean command

Because php-fpm is running as apache user --> owner of log file is apache, so we need to running cronjob as apache user

### First way (Do not know why it does not work)

Add cronjob for apache user

```
sudo su -
sudo crontab -u apache -e 
```

Paste the below lines to the end of file

```
* * * * * cd /home/building-survey-report-system-be && php81 artisan schedule:run >> /dev/null 2>&1
* * * * * cd /home/staging/building-survey-report-system-be && php81 artisan schedule:run >> /dev/null 2>&1
```

### Second way (working)

Add cronjob for root user, and run command as apache user

```
sudo su -
sudo crontab -e
```

Paste the below lines to the end of file

```
* * * * * cd /home/building-survey-report-system-be && su -u apache /bin/sh -c 'php81 artisan schedule:run >> /dev/null 2>&1'
* * * * * cd /home/staging/building-survey-report-system-be && su -u apache /bin/sh -c 'php81 artisan schedule:run >> /dev/null 2>&1'
```
