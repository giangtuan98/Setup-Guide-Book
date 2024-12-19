CentOS 7

```
curl -sS https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer
```

### Install pdftotext
```
yum install poppler-utils
```

### Install ffmpeg
```
rpm -Uvh http://li.nux.ro/download/nux/dextop/el7/x86_64/nux-dextop-release-0-5.el7.nux.noarch.rpm
yum install ffmpeg ffmpeg-devel -y
```

### Install node 16
```
yum install -y gcc-c++ make 
curl -sL https://rpm.nodesource.com/setup_16.x | sudo -E bash - 
sudo yum install -y nodejs
```

### Install EFS
```
sudo yum -y install git rpm-build make rust cargo openssl-devel
git clone https://github.com/aws/efs-utils
cd efs-utils
make rpm
sudo yum -y install build/amazon-efs-utils*rpm
```
