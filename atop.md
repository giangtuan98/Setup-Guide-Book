### Setup atop để tracking CPU

```
yum install -y atop
```

Setup crontab to log data.
```
0 * * * * /usr/bin/atop -a -w /var/log/atop/atop_cpu_$(date +\%Y\%m\%d_\%H) -Ccpu [second]
```

View data at specific minute
```
atop -r /var/log/atop/atop_cpu_20240121_15 -b 15:30
```