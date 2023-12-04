### Install cerbot
```
sudo yum install epel-release
sudo yum install certbot certbot-nginx
```

### Create certificate
```
sudo certbot --nginx
```

### Cronjob
```
00 6 * * * certbot renew --nginx --nginx-ctl /usr/sbin/nginx --cert-name your_domain.example --quiet
```

### Check logs
```
vi /var/log/letsencrypt/letsencrypt.log
```
