# Setup nodejs on ec2

## Install ec2-instance-connect to allow access instance from AWS EC2 instance console

```
yum install -y ec2-instance-connect
```

## Install environmental GIT, PHP, PHP_FPM, NGINX

### Install git

```
yum install -y git
```

### Install nginx

We will need nginx as a reverse proxy for our Node.js application.
It will listen on port 80 and pass all requests to our application that is listening on port 3000.
It also handle SSL/TLS termination.

```
sudo yum install nginx
```

### Install dependencies for chrome

```
sudo yum install atk-devel gtk3 nss xorg-x11-server-Xvfb mesa-libgbm alsa-lib
```

### Install ghostscript & ImageMagick (for convert pdf to image)

```
sudo yum install -y ghostscript ImageMagick GraphicsMagick
```

### Install qpdf (for decrypt password protected pdf)

```bash
sudo yum install -y qpdf
```

## Create developer user

```
useradd -m developer (Create user with *mkdir* home directory)
passwd developer
sudo su - developer
```

## Setup project in developer user

### Install nodejs 18.x

https://docs.aws.amazon.com/sdk-for-javascript/v2/developer-guide/setting-up-node-on-ec2-instance.html

```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
. ~/.nvm/nvm.sh
nvm install --lts
node -v
```

### Generate deploy key

1. Generate deploy key on ec2 with developer user

```
sudo su - developer
ssh-keygen -t ed25519 -C "<comment>"
vim .ssh/id_ed25519.pub
```

2. Go to setting project git repository
3. Open tab deploy key
4. Add ssh public key

### Install yarn

```
npm install -g yarn
yarn -v
```

### Install pm2 (must use npm)

```
npm i -g pm2
```

### Init project

1. Read file README.md in project
2. Config file .env **(PORT = 3000)**

### Start project with pm2

```
pm2 start dist/server.js --name <export-pdf>
pm2 save
pm2 startup
sudo env PATH=$PATH:/home/developer/.nvm/versions/node/v18.16.1/bin /home/developer/.nvm/versions/node/v18.16.1/lib/node_modules/pm2/bin/pm2 startup systemd -u developer --hp /home/developer
```

**Important**:
You need to restart pm2 each time release new version

```
pm2 restart <export-pdf>
```

### Setup nginx to reverse proxy to our application

1. Create file /etc/nginx/conf.d/export-pdf.conf

```
server {
    listen 80;
    server_name pdf.asbestos.sat-co.info;
    
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

2. Check nginx syntax

```
sudo nginx -t
```

3. Start/Restart nginx

```
sudo systemctl start nginx
```

4. Check nginx status

```
sudo systemctl status nginx
```

5. Enable nginx to start on boot

```
sudo systemctl enable nginx
```



