Send log to cloudwatch

Check server OS

```
uname -srv
```

Install new version awslogs

```
sudo yum install amazon-cloudwatch-agent
```

Create user cwagent and add to group adm

```
usermod -aG adm cwagent
```

Setup credential for server to grant permission to send log to cloudwatch.
If you are using EC2 instance, you can create IAM role then attach to EC2 instance instead of using credential.

```
vim /home/cwagent/.aws/credentials
aws_access_key_id = <your_access_key_id>
aws_secret_access_key = <your_secret_access_key>
```

Create config file for cloudwatch agent

```
vim /home/cwagent/.aws/config
```

## Setting AWS

### Create log group for api-production

Access to cloudwatch and create a log group with name: **api-production-log-group**

### Create policy allow send logs to cloudwatch

```
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "Statement1",
			"Effect": "Allow",
			"Action": [
				"logs:CreateLogGroup",
				"logs:DescribeLogStreams",
				"logs:PutLogEvents"
			],
			"Resource": [
				"arn:aws:logs:ap-northeast-1:674308597900:log-group:api-production-log-group:*",
				"arn:aws:logs:ap-northeast-1:674308597900:log-group:api-production-log-group"
			]
		}
	]
}
```

### Attach policy to role of EC2 production

## Telling Cloudwatch logs agent which log files to collect

### Run cloudwatch logs tool to create config file

This tool will ask you some questions to create config file.

```
/opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard
```

After answer all question, the config file will be created at **/opt/aws/amazon-cloudwatch-agent/bin/config.json**

```
vim /opt/aws/amazon-cloudwatch-agent/bin/config.json
```

Or, you can create the config file manually with below content

```
{
    "agent": {
        "metrics_collection_interval": 60,
        "run_as_user": "cwagent"  # user cwagent which we created above
    },
    "logs": {
        "logs_collected": {
            "files": {
                "collect_list": [
                    {
                        "file_path": "/home/developer/building-survey-report-system-be/storage/logs",  # path to logs file
                        "log_group_name": "api-production-log-group", # log group name which we created above
                        "log_stream_name": "{instance_id}",
                        "retention_in_days": 14
                    }
                ]
            }
        }
    }
}
```

### Enable & start cloudwatch logs agent

Enable cloudwatch logs agent to automatically start service on boot when Auto Scaling create new instance
Start cloudwatch logs agent for current instance

```
sudo systemctl enable amazon-cloudwatch-agent
sudo systemctl start amazon-cloudwatch-agent
```

### Check log amazon-cloudwatch-agent

```
tail -f /var/log/amazon/amazon-cloudwatch-agent/amazon-cloudwatch-agent.log
```
