## Config free SSL certificate with certbot on Amazon Linux 2023

### Install certbot (Develop & Staging environment)

``` 
sudo python3 -m venv /opt/certbot/
sudo /opt/certbot/bin/pip install --upgrade pip
sudo /opt/certbot/bin/pip install certbot
```

### Generate SSL certificate

You can refer to this link:

- https://itnext.io/node-express-letsencrypt-generate-a-free-ssl-certificate-and-run-an-https-server-in-5-minutes-a730fbe528ca
- https://medium.com/@ashishgabhane1/enable-https-for-nodejs-app-running-on-elastic-beanstalk-using-certbot-523c1c9294ce

1. Generate certificate key (choose one solution)

   #### First solution: generate certificate manually

    ```
    certbot certonly --manual
    ```

    1. Create file .well-known/acme-challenge/xxxxx with content which certbot show
    2. Allow express serve static file from .well-known/acme-challenge. Add to app.ts

    ```
    app.use(express.static(path.resolve(__dirname, '..'), {dotfiles: 'allow'}));
    ```

    3. Test access file from browser
    4. On Certbot process, Press Enter to continue

   #### Second solution: Generate certificate manually with challenge

    1. Run command on EC2 instance

    ```
    sudo certbot certonly --manual --preferred-challenges dns
    ```

    2. Certbot will provide a TXT record to add to your DNS configuration. Add TXT record to DNS configuration
    3. On EC2 instance, press Enter to continue


2. Add logic to read certbot file in development

```nodejs
const privateKey = fs.readFileSync('/etc/letsencrypt/live/yourdomain.com/privkey.pem', 'utf8');
const certificate = fs.readFileSync('/etc/letsencrypt/live/yourdomain.com/cert.pem', 'utf8');
const ca = fs.readFileSync('/etc/letsencrypt/live/yourdomain.com/chain.pem', 'utf8');

const credentials = {
	key: privateKey,
	cert: certificate,
	ca: ca
};

app.use((req, res) => {
	res.send('Hello there !');
});

// Starting both http & https servers
const httpServer = http.createServer(app);
const httpsServer = https.createServer(credentials, app);

httpServer.listen(80, () => {
	console.log('HTTP Server running on port 80');
});

httpsServer.listen(443, () => {
	console.log('HTTPS Server running on port 443');
});
```

==============CERTBOT NGINX================

### Install certbot nginx extension

```
sudo /opt/certbot/bin/pip install certbot-nginx
```

### Generate SSL certificate

```
sudo certbot --nginx
```

### Auto renew certificate by cronjob

#### Install cronjob (have to install from Amazon Linux 2023)

```
yum install cronie cronie-anacron
```

#### Add cronjob

```
sudo crontab -e
```

Paste this line to crontab file

```
00 6 * * * certbot renew --nginx --nginx-ctl /usr/sbin/nginx --quiet
```

(renew certificate at 6:00 AM everyday)

#### Check cronjob log (EC2 Amazon Linux 2023)

``` 
sudo grep CRON /var/log/cron
```

#### Check log renew certificate

```
tail -f /var/log/letsencrypt/letsencrypt.log
```