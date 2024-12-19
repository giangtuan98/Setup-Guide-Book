Amazon Linux 2023

```
yum install -y git
yum install -y nginx

ssh-keygen -t ed25519 -C "deploy"
eval "$(ssh-agent -s)"

yum install php8.2
php -v
yum install php-{fpm,pdo,pdo_mysql,dom,bcmath,xml,mbstring,gd}

# Install composer
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('sha384', 'composer-setup.php') === 'dac665fdc30fdd8ec78b38b9800061b4150413ff2e3b6f88543c636f7cd84f6db9189d43a81e5503cda447da73c7e5b6') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php
php -r "unlink('composer-setup.php');"

```

Install ffmpeg
```
cd /usr/local/bin/
sudo wget https://github.com/BtbN/FFmpeg-Builds/releases/download/latest/ffmpeg-master-latest-linuxarm64-gpl.tar.xz
sudo tar -xf ffmpeg-master-latest-linuxarm64-gpl.tar.xz
sudo mv ffmpeg-master-latest-linuxarm64-gpl/ ffmpeg/
sudo rm ffmpeg-master-latest-linuxarm64-gpl.tar.xz
sudo chown -R root.root ffmpeg/
sudo ln -s /usr/local/bin/ffmpeg/bin/ffmpeg /usr/bin/ffmpeg
sudo ln -s /usr/local/bin/ffmpeg/bin/ffprobe /usr/bin/ffprobe
```

