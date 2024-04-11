# Send RAM usage metrics to CloudWatch

Because exporting PDF uses RAM as main resource, we can send RAM usage metrics to CloudWatch to create Alarms to scale
up/down the EC2 instance.

## Attach permissions to EC2 instance Role

1. Go to IAM > Roles > Select the role of the EC2 instance > Attach policies
2. Search for `CloudWatchAgentServerPolicy` and attach it

## Install CloudWatch Agent

1. Connect to the EC2 instance
2. Install CloudWatch Agent

```bash
sudo yum install -y amazon-cloudwatch-agent
```

3. Create a configuration file (select 1 of 2 options)

    - A, Run wizard file
        ```bash
        sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard
        ```

    - B, Create a configuration file manually
        ```bash
        vim /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json
       ```
4. Configuration file

```bash
{
  "metrics": {
    "namespace": "ExportPDF/MemoryUtilization",
    "append_dimensions": {
      "AutoScalingGroupName": "${aws:AutoScalingGroupName}"
    },
    "metrics_collected": {
      "mem": {
        "measurement": [
          "mem_used_percent"
        ],
        "append_dimensions": {
          "Environment": "Production"
        },
        "metrics_collection_interval": 60
      }
    }
  }
}
```

4. Enable & start cloudwatch logs agent

Enable cloudwatch logs agent to automatically start service on boot when Auto Scaling create new instance Start
cloudwatch logs agent for current instance

```bash
sudo systemctl enable amazon-cloudwatch-agent
sudo systemctl start amazon-cloudwatch-agent
```

## Check metrics on CloudWatch

Go to CloudWatch > Metrics > All metrics > Custom Namespaces > CWAgent > InstanceId > mem_used_percent

## Create custom Auto Scaling Target tracking policy

### 1. Create a policy config file (example aws-asg-policy.json)

Note: All dimensions exists in the configuration file will need to be added to the policy

```json
{
  "TargetValue": 60.0,
  "CustomizedMetricSpecification": {
    "MetricName": "MemoryUtilization",
    "Namespace": "ExportPDF/MemoryUtilization",
    "Dimensions": [
      {
        "Name": "AutoScalingGroupName",
        "Value": "production_api_export_pdf_sg"
      },
      {
        "Name": "Environment",
        "Value": "Production"
      }
    ],
    "Statistic": "Average",
    "Unit": "Percent"
  }
}

```

### 2. Setup AWS CLI & login

```bash
aws configure --profile sat
```

### 3. Create a policy

Replace your attribute to this sample

```bash
aws --profile sat autoscaling put-scaling-policy \
--policy-name [policy-name] \
--auto-scaling-group-name "[ASG-Name]"  // Replace by your ASG name \
--policy-type "TargetTrackingScaling" \
--target-tracking-configuration file://[file-config.json]
```

Example

```
aws --profile sat autoscaling put-scaling-policy --policy-name MemoryUtilization60 --auto-scaling-group-name "production_api_export_pdf_sg" --policy-type "TargetTrackingScaling" --target-tracking-configuration file://aws-asg-policy.json 
```

### 4. Check policy created

Login into AWS Console > EC2 > Auto Scaling Groups > Select your ASG > Click on the tab "Automatic Scaling"
You will see the policy you just created

### 5. Test policy scale ability by stress tool

#### Install stress tool

```bash
sudo amazon-linux-extras install epel -y
sudo yum install stress -y
```

#### Run stress tool

Depending on the instance type, you need to calculate the number of threads and memory to use for the stress tool.

**Example:**

You have an instance type t3a.small with 2GB RAM.
You want to use 60% of RAM for the stress tool.
You need to calculate the number of threads and memory to use for the stress tool as below:

- 60% of 2GB = 1.2GB
- 1.2GB = 1200MB
- 1200MB = 6(malloc) * 200M (each malloc)

To ensure the RAM usage is over 60%, you can run the command below:
You can increase the number of threads or memory.

```bash
stress --vm 6 --vm-bytes 260M
```

### ~~5. Test policy scale ability by using command to put custom metric data to CloudWatch~~

AWS provides a command to put custom metric data to CloudWatch.
We will use this command to make the RAM usage of the instance increase to 60% to test the policy.
Metric data was defined in the **metric.json** file.

```bash
aws --profile sat cloudwatch put-metric-data --metric-data file://metric.json
```

### 6. Check ASG scale up

After about 3 minutes, you will see the number of instances in the ASG increase to 2.
And 1 AWS CloudWatch Alarm is created.

=> The policy works as expected.

### 7. Check ASG scale down

You can stop EC2 stress tool to make the RAM usage decrease below 60% to test the policy scale down.
