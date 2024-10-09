# Practical Worksheet 6

Version: 1.0 Date: 12/04/2018 Author: David Glance

Date: 24/07/2024 Updated by Zhi Zhang

## Learning Objectives

1. Create a web app using Django
2. Implement nginx and load balance requests to it
3. Retrieve data from DynamoDB to display in the app

## Technologies Covered

* Ubuntu
* AWS
* AWS ELB
* RDS
* Python/Boto scripts

**NOTE**: please use your Linux environment – if you do it from any other OS (e.g., Windows, Mac – some unknow issues might occur)

## Background

The aim of this lab is to write a program that will:

[1] Understand the basis for a web architecture that incorporates scalability and security using ELB

[2] Familiarise yourself with the basics of programming using Django

## Set up an EC2 instance

### [1] Create an EC2 micro instance with Ubuntu and SSH into it
 to create ec2 instance, we will utilize a modified version of the python program we created earlier for this purpose in lab 2. the modification will Allow HTTP traffic.
```python3
import boto3
import os
import subprocess
import time

# Initialize the EC2 client
ec2 = boto3.client('ec2')

# Step 1: Create a security group
security_group = ec2.create_security_group(
    Description='security group for development environment',
    GroupName='23803313-sg',
)
print(f"Security Group Created: {security_group['GroupId']}")

# Step 2: Authorize inbound traffic for SSH and HTTP
ec2.authorize_security_group_ingress(
    GroupName='23803313-sg',
    IpPermissions=[
        # Allow SSH traffic (port 22)
        {
            'IpProtocol': 'tcp',
            'FromPort': 22,
            'ToPort': 22,
            'IpRanges': [{'CidrIp': '0.0.0.0/0'}]
        },
        # Allow HTTP traffic (port 80)
        {
            'IpProtocol': 'tcp',
            'FromPort': 80,
            'ToPort': 80,
            'IpRanges': [{'CidrIp': '0.0.0.0/0'}]
        }
    ]
)
print(f"Inbound SSH and HTTP traffic authorized for {security_group['GroupId']}")

# Step 3: Create a key pair
key_pair_name = '23803313-key'
key_pair = ec2.create_key_pair(KeyName=key_pair_name)
key_file_path = f'{key_pair_name}.pem'
with open(key_file_path, 'w') as file:
    file.write(key_pair['KeyMaterial'])

# Change the file permission to chmod 400
os.chmod(key_file_path, 0o400)
print(f'Key pair created, saved to {key_file_path}, and permissions set to 400')

# Step 4: Create the instance
instance = ec2.run_instances(
    ImageId="ami-0a70c5266db4a6202",
    SecurityGroupIds=[security_group['GroupId']],  # Use a list here
    InstanceType='t2.micro',
    KeyName=key_pair_name,
    MinCount=1,
    MaxCount=1
)

instance_id = instance['Instances'][0]['InstanceId']
print(f'EC2 Instance Created: {instance_id}')

# Step 5: Add a tag to your instance
ec2.create_tags(
    Resources=[instance_id],
    Tags=[{'Key': 'Name', 'Value': '23803313-vm2'}]
)
print(f'Tag added to instance {instance_id}')

# Step 6: Get the public IP address
response = ec2.describe_instances(InstanceIds=[instance_id])
public_ip = response['Reservations'][0]['Instances'][0]['PublicIpAddress']
print(f'Public IP Address of the instance: {public_ip}')

print('Waiting for the instance to initialize...')
time.sleep(240)  # Wait for instance initialization

# Step 7: Connect to the instance via SSH
ssh_command = f"ssh -i {key_file_path} ubuntu@{public_ip}"
print(f'Connecting to the instance via SSH: {ssh_command}')
try:
    subprocess.run(ssh_command, shell=True, check=True)
except subprocess.CalledProcessError as e:
    print(f"Failed to connect to the instance: {e}")

```
this program 
Creates a Security Group
Authorizes Inbound SSH and HTTP Traffic
Creates a Key Pair and Set Permissions
Launches the EC2 Instance and tags the instance
 Retrieves the Public IP Address 
and finally Connects to the Instance via SSH

![image](https://github.com/user-attachments/assets/e5af6d87-5581-4f0f-ac3b-cf5929d6b91e)


We can confirm that the instance was created in the console

![image](https://github.com/user-attachments/assets/5fc535cf-6a1f-4a6b-b42c-9dd9b806bddd)


### [2] Install the Python 3 virtual environment package
then we install python 3 virtual invironment on our newly ceated instance
```bash
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install python3-venv
```

![image](https://github.com/user-attachments/assets/0fb98d38-9b62-433b-b3d5-cb92af715886)

we ran into some issues related to daemons using outdated libraries and had to resart some services.

![image](https://github.com/user-attachments/assets/806c061f-7be5-4399-b109-5a652dd94678)

now we change the bash to operate as sudo

```
sudo bash
```

### [3] Access a directory  

next we Create a directory with a path `/opt/wwc/mysites` and `cd` into the directory.
```bash
mkdir -p /opt/wwc/mysites
cd /opt/wwc/mysites
```

### [4] Create and activate virtual environment

```bash
python3 -m venv myvenv
source myvenv/bin/activate
```

![image](https://github.com/user-attachments/assets/2edd7cf5-85a8-462d-b31d-61044f557631)


### [5] Install Django,  Create Django project and django app

```

pip install django

django-admin startproject lab

cd lab

python3 manage.py startapp polls
```

now we check the contents 
```bash
cd lab
ls -l
```
(myvenv) root@ip-172-31-40-111:/opt/wwc/mysites/lab# python3 manage.py startapp polls
(myvenv) root@ip-172-31-40-111:/opt/wwc/mysites/lab# ls -l
total 12
drwxr-xr-x 3 root root 4096 Oct  8 11:34 lab
-rwxr-xr-x 1 root root  659 Oct  8 11:34 manage.py
drwxr-xr-x 3 root root 4096 Oct  8 11:34 polls
(myvenv) root@ip-172-31-40-111:/opt/wwc/mysites/lab# cd lab
(myvenv) root@ip-172-31-40-111:/opt/wwc/mysites/lab/lab# ls -l
total 20
-rw-r--r-- 1 root root    0 Oct  8 11:34 __init__.py
drwxr-xr-x 2 root root 4096 Oct  8 11:34 __pycache__
-rw-r--r-- 1 root root  383 Oct  8 11:34 asgi.py
-rw-r--r-- 1 root root 3212 Oct  8 11:34 settings.py
-rw-r--r-- 1 root root  759 Oct  8 11:34 urls.py
-rw-r--r-- 1 root root  383 Oct  8 11:34 wsgi.py

![image](https://github.com/user-attachments/assets/0f2ece15-fd1f-48b8-9bbc-50d109a00eb3)

### [6] Install nginx

```
apt install nginx
```

### [7] Configure nginx

edit `/etc/nginx/sites-enabled/default` and replace the contents of the file with

```bash
sudo nano /etc/nginx/sites-enabled/default
```

```
server {
  listen 80 default_server;
  listen [::]:80 default_server;

  location / {
    proxy_set_header X-Forwarded-Host $host;
    proxy_set_header X-Real-IP $remote_addr;

    proxy_pass http://127.0.0.1:8000;
  }
}
```

![image](https://github.com/user-attachments/assets/1e3b8111-51eb-4642-b02d-a276a9f1e2e4)

### [8] Restart nginx

```
service nginx restart
```


### [9] Access your EC2 instance

In your app directory: `/opt/wwc/mysites/lab`, run:

```
python3 manage.py runserver 8000
```

![image](https://github.com/user-attachments/assets/59e6cb3c-aa4a-4715-9f28-5921fa6628b8)


then i Open a browser and enter the IP address of your EC2 instance. 

![image](https://github.com/user-attachments/assets/cbe7da26-7834-4047-b381-21c2c94054fa)


## Set up Django inside the created EC2 instance

### [1] Edit the following files (create them if not exist)

edit polls/views.py

```bash
nano polls/views.py
```

```
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello, world.")
```
![image](https://github.com/user-attachments/assets/db289496-0d65-406b-9ebc-a8dba65d2202)

edit polls/urls.py 

```bash
nano polls/urls.py
```

```
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```
![image](https://github.com/user-attachments/assets/24579883-73a3-44e5-a46e-bb1207f0f5b4)


edit lab/urls.py
nano lab/urls.py

```
from django.urls import include, path
from django.contrib import admin

urlpatterns = [
    path('polls/', include('polls.urls')),
    path('admin/', admin.site.urls),
]
```
![image](https://github.com/user-attachments/assets/9cc66874-f293-42cc-9c81-a2f34b4368fd)


### [2] Run the web server again

```
python3 manage.py runserver 8000
```

### [3] Access the EC2 instance

Access the URL: http://13.208.165.62/polls/, to verify changes

![image](https://github.com/user-attachments/assets/4bedbd35-4f3e-4a3d-a052-4f0bacdbb46e)


## Set up an ALB

### [1] Create an application load balancer

23803313-alb

Specify the region subnet where your EC2 instance resides.

Create a listener with a default rule Protocol: HTTP and Port 80 forwarding.

Choose the security group, allowing HTTP traffic. 

Add your instance as a registered target.

for this we use a modifed version of the python program we creted in lab 5
```python3
import boto3
import time

# Initialize EC2 and ELBv2 clients in the specified region
ec2 = boto3.client('ec2', region_name='ap-northeast-3')
elbv2 = boto3.client('elbv2', region_name='ap-northeast-3')

# Define parameters for the key pair and security group
key_name = '23803313-key'
security_group_name = '23803313-sg'

# Step 1: Get Instance Details
# Use describe_instances to find the instances created with the specified key name and security group

try:
    # Use filters to find instances created with a specific key name and security group
    response = ec2.describe_instances(
        Filters=[
            {'Name': 'key-name', 'Values': [key_name]},
            {'Name': 'instance.group-name', 'Values': [security_group_name]},
            {'Name': 'instance-state-name', 'Values': ['running']}  # Filter for running instances
        ]
    )
    
    instance_ids = []
    subnet_ids = []
    
    # Loop through the response to extract the instance IDs and subnet IDs
    for reservation in response['Reservations']:
        for instance in reservation['Instances']:
            instance_id = instance['InstanceId']
            subnet_id = instance['SubnetId']
            instance_ids.append(instance_id)
            subnet_ids.append(subnet_id)
            print(f"Instance ID: {instance_id}, Subnet ID: {subnet_id}")

    if len(instance_ids) == 0:
        print("No running instances found.")
    else:
        print(f"Found {len(instance_ids)} running instance(s).")

except Exception as e:
    print(f"An error occurred while retrieving instance details: {e}")

# Step 2: Retrieve the Security Group ID
try:
    # Attempt to describe the security group by name to see if it already exists
    response_sg = ec2.describe_security_groups(GroupNames=[security_group_name])
    security_group_id = response_sg['SecurityGroups'][0]['GroupId']
    print(f"Existing Security Group found: {security_group_id}\n")
except ec2.exceptions.ClientError as e:
    # Handle unexpected errors
    print(f"Unexpected error: {e}\n")

# Step 3: Create an Application Load Balancer (ALB)
try:
    # Use the security group ID instead of the group name
    load_balancer = elbv2.create_load_balancer(
        Name='23803313-alb',
        Subnets=subnet_ids,  # Use the subnets of the created instances to attach to the load balancer
        SecurityGroups=[security_group_id],  # Attach the security group ID
        Scheme='internet-facing',   # Make the load balancer accessible from the internet
        Tags=[
            {'Key': 'Name', 'Value': '23803313-alb'}
        ],
        Type='application',         # Specify that this is an application load balancer
        IpAddressType='ipv4'
    )
    # Extract the ARN (Amazon Resource Name) for the load balancer
    lb_arn = load_balancer['LoadBalancers'][0]['LoadBalancerArn']
    print(f"Load Balancer created: {lb_arn}")

except Exception as e:
    print(f"An error occurred while creating the load balancer: {e}")

# Step 4: Create a Target Group for the Load Balancer
try:
    # Correctly fetch the VPC ID from the instance description
    instance_details = ec2.describe_instances(InstanceIds=[instance_ids[0]])
    vpc_id = instance_details['Reservations'][0]['Instances'][0]['VpcId']
    
    # Create a target group for the load balancer
    target_group = elbv2.create_target_group(
        Name='23803313-tg',
        Protocol='HTTP',
        Port=80,
        VpcId=vpc_id,
        HealthCheckProtocol='HTTP',
        HealthCheckPort='80',
        HealthCheckPath='/polls/',
        TargetType='instance'
    )
    
    target_group_arn = target_group['TargetGroups'][0]['TargetGroupArn']
    print(f"Target Group created: {target_group_arn}")

except Exception as e:
    print(f"An error occurred while creating the target group: {e}")

# Step 5: Register the EC2 Instances as Targets in the Target Group
if 'target_group_arn' in locals():
    try:
        # Register the EC2 instances with the target group
        elbv2.register_targets(
            TargetGroupArn=target_group_arn,
            Targets=[{'Id': instance_id} for instance_id in instance_ids]
        )
        print(f"Instances registered to target group: {target_group_arn}")

    except Exception as e:
        print(f"An error occurred while registering targets: {e}")
else:
    print("Target group creation failed. Skipping instance registration.")

# Step 6: Create a Listener for the Load Balancer
if 'lb_arn' in locals() and 'target_group_arn' in locals():
    try:
        # Create a listener that will forward incoming HTTP requests on port 80 to the target group
        listener = elbv2.create_listener(
            LoadBalancerArn=lb_arn,  # ARN of the load balancer to attach the listener to
            Protocol='HTTP',
            Port=80,
            DefaultActions=[
                {
                    'Type': 'forward',
                    'TargetGroupArn': target_group_arn
                }
            ]
        )
        # Extract the ARN of the listener
        listener_arn = listener['Listeners'][0]['ListenerArn']
        print(f"Listener created: {listener_arn}")

    except Exception as e:
        print(f"An error occurred while creating the listener: {e}")
else:
    print("Load balancer or target group creation failed. Skipping listener creation.")
)
```

![image](https://github.com/user-attachments/assets/d8395956-f6d7-4cb3-943f-3b59dc3a921e)


### [2] Health check

For the target group, specify /polls/ for a path for the health check.

Confirm the health check fetch the /polls/ page every 30 seconds.

![image](https://github.com/user-attachments/assets/97a19c78-6b7e-407a-b16f-a31512bd7184)


![image](https://github.com/user-attachments/assets/03dbdb4c-427b-4724-badc-a95b41a5ee30)

### [3] Access

Access the URL: http://23803313-alb-1046759913.ap-northeast-3.elb.amazonaws.com/polls/, and output what you've got.

![image](https://github.com/user-attachments/assets/70c46249-efbb-4792-b9d8-8364b85e1c39)


**NOTE**: When you are done, delete the instance and ALB you created.

## [Unmarked] Web interface for CloudStorage application

You need to create an AWS DynamoDB table copied from the local DynamoDB of the previous lab 3 as well as a copy of your AWS credentials.

In views.py, add boto3 code to scan the AWS DynamoDB table. Display the results in the calling page.

In Django, you can use a template to properly format a web page using supplied variables – you can do that to make the table look nice. To use a template, you need to create a folder called templates under polls and add to the TEMPLATES section of lab/settings.py

```
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [
            'polls/templates/'
        ],
```

In the templates directory, add a file files.html with the following contents:

```
<html>
<head>
    <title>Files</title>
</head>
<body>
    <h1>Files </h1>


    <ul>
        {% for item in items %}
          <li>{{ item.fileName }}</li>
	{% endfor %}
    </ul>

</body>
</html>
```


Finally in views.py, you can pass variables from your DynamoDB call and render the template in the following way:

```
from django.shortcuts import render
from django.template import loader
from django.http import HttpResponse
import boto3
import json
from boto3.dynamodb.conditions import Key, Attr
from botocore.exceptions import ClientError

def index(request):
    template = loader.get_template('files.html')

    dynamodb = boto3.resource('dynamodb', region_name='ap-southeast-2',
                              aws_access_key_id='Your Access Key',
                              aws_secret_access_key='Your Secret')

    table = dynamodb.Table("UserFiles")

    items = []
    try:
        response = table.scan()

    except ClientError as e:
        print(e.response['Error']['Message'])
    else:    
        context = {'items': response['Items'] }

        return HttpResponse(template.render(context, request))
```


You can add variables to the template and more formatting functionality to display the information correctly.

Lab Assessment:

A structured presentation (15%). A clear step-by-step with detailed descriptions (85%). 
