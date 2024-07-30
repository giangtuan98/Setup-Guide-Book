# Supervisor

## Linux

## Centos
### Install

### Create supervisor worker
```
cd /etc/supervisor/conf.d
vi example.conf
```

Copy and paste the following config into the file you just create
```conf
[program:worker-name]
process_name=%(program_name)s_%(process_num)02d
command=php PATH/TO/FOLDER/artisan queue:work --queue=welcome_user_email --sleep=3 --max-time=3600
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
user=www
numprocs=1
redirect_stderr=true
stdout_logfile=/www/hoshukan_prod/api/storage/logs/worker_send_email.log
stopwaitsecs=3600
```

### Run new worker
```
sudo supervisorctl reread
 
sudo supervisorctl update
 
sudo supervisorctl start "worker-name:*"
```