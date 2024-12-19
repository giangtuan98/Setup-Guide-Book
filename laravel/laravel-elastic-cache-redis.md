Amazon Linux 2023

### Instal PHP-Redis extension

#### Solution 1
https://github.com/amazonlinux/amazon-linux-2023/issues/328
```
# Install php-devel, gcc, make and git
sudo dnf install php8.2-devel gcc make git
# Git clone phpredis repo
cd
git clone https://github.com/phpredis/phpredis.git
cd phpredis
# Checkout the latest stable version of 6.0.2
git checkout tags/6.0.2
# Run phpize
phpize
# Run configure
./configure
# Run compile
make && sudo make install
# Adding extention to PHP config
sudo echo "extension = redis.so" >  /etc/php.d/50-redis.ini
```

#### Solution 2
Using pickle - https://github.com/FriendsOfPHP/pickle
