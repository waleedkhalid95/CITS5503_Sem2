# Lab 6: Deploying a Django Web App with Application Load Balancer

## Summary
In this lab, I set up a Django web application on an EC2 instance and configured an Application Load Balancer (ALB) to distribute HTTP traffic between instances for scalability and high availability. The main tasks involved creating the EC2 instance and security groups, installing and configuring a Django app, setting up nginx as a reverse proxy, and configuring the ALB with health checks to ensure the application is running smoothly. By the end of the lab, I accessed the Django application through the ALB's DNS name, confirming successful load balancing and web traffic distribution.

## Set up an EC2 instance

### [1] Create an EC2 Instance with Ubuntu and SSH into It
I modified the Python script used in Lab 2 to create an EC2 instance with both SSH and HTTP traffic allowed. The steps involve creating a security group, launching an instance, and connecting via SSH.
```python3
import boto3
import os
import subprocess
import time

# Initialize the EC2 client to interact with AWS EC2 services
ec2 = boto3.client('ec2')

# Step 1: Create a security group for the EC2 instance
security_group = ec2.create_security_group(
    Description='security group for development environment',
    GroupName='23803313-sg',
)
print(f"Security Group Created: {security_group['GroupId']}")

# Step 2: Authorize inbound traffic for SSH (port 22) and HTTP (port 80)
ec2.authorize_security_group_ingress(
    GroupName='23803313-sg',
    IpPermissions=[
        # Allow SSH traffic on port 22
        {
            'IpProtocol': 'tcp',
            'FromPort': 22,
            'ToPort': 22,
            'IpRanges': [{'CidrIp': '0.0.0.0/0'}]  # Open SSH to all IPs for development
        },
        # Allow HTTP traffic on port 80
        {
            'IpProtocol': 'tcp',
            'FromPort': 80,
            'ToPort': 80,
            'IpRanges': [{'CidrIp': '0.0.0.0/0'}]  # Open HTTP for web access
        }
    ]
)
print(f"Inbound SSH and HTTP traffic authorized for {security_group['GroupId']}")

# Step 3: Create a key pair for SSH access
key_pair_name = '23803313-key'
key_pair = ec2.create_key_pair(KeyName=key_pair_name)
key_file_path = f'{key_pair_name}.pem'

# Save the private key to a file and set secure permissions (chmod 400)
with open(key_file_path, 'w') as file:
    file.write(key_pair['KeyMaterial'])
os.chmod(key_file_path, 0o400)
print(f'Key pair created, saved to {key_file_path}, and permissions set to 400')

# Step 4: Launch the EC2 instance with Ubuntu AMI and created security group
instance = ec2.run_instances(
    ImageId="ami-0a70c5266db4a6202",  # Ubuntu AMI for the ap-northeast-3 region
    SecurityGroupIds=[security_group['GroupId']],
    InstanceType='t2.micro',  # Free-tier instance type
    KeyName=key_pair_name,  # Use the created key pair for SSH
    MinCount=1,
    MaxCount=1
)

instance_id = instance['Instances'][0]['InstanceId']
print(f'EC2 Instance Created: {instance_id}')

# Step 5: Tag the instance with a recognizable name
ec2.create_tags(
    Resources=[instance_id],
    Tags=[{'Key': 'Name', 'Value': '23803313-vm2'}]  # Name tag for easy identification
)
print(f'Tag added to instance {instance_id}')

# Step 6: Retrieve the instance's public IP address for SSH access
response = ec2.describe_instances(InstanceIds=[instance_id])
public_ip = response['Reservations'][0]['Instances'][0]['PublicIpAddress']
print(f'Public IP Address of the instance: {public_ip}')

# Wait for the instance to initialize and be ready
print('Waiting for the instance to initialize...')
time.sleep(240)  # Wait for 4 minutes to allow instance initialization

# Step 7: Connect to the instance via SSH using the generated key and public IP
ssh_command = f"ssh -i {key_file_path} ubuntu@{public_ip}"
print(f'Connecting to the instance via SSH: {ssh_command}')
try:
    subprocess.run(ssh_command, shell=True, check=True)  # Execute the SSH command
except subprocess.CalledProcessError as e:
    print(f"Failed to connect to the instance: {e}")
```
**Explanation**:
- **Security Group Creation**: I created a security group to allow both SSH and HTTP traffic. This is crucial for remotely managing the instance (SSH) and serving the Django application (HTTP).
- **Key Pair**: The script generates a key pair for secure SSH access, saves the private key locally, and restricts access to it using `chmod 400`
- **Launch EC2 Instance**: An EC2 instance is created using the Ubuntu AMI. I associated it with the security group and key pair, and tagged it with a name for identification.
- **Public IP**: I retrieved the public IP of the instance, which is required for SSH access and to later access the Django app.
- **SSH Connection**: The script attempts to SSH into the instance using the private key and public IP.

![image](https://github.com/user-attachments/assets/e5af6d87-5581-4f0f-ac3b-cf5929d6b91e)

Once the instance was up and running, I confirmed its creation in the AWS console:

![image](https://github.com/user-attachments/assets/5fc535cf-6a1f-4a6b-b42c-9dd9b806bddd)


### [2] Install Python 3 Virtual Environment
After connecting to the instance, I updated the instance and installed the necessary Python 3 virtual environment package:
```bash
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install python3-venv
```

![image](https://github.com/user-attachments/assets/0fb98d38-9b62-433b-b3d5-cb92af715886)

I encountered some issues during the upgrade due to outdated libraries, which required restarting certain services:

![image](https://github.com/user-attachments/assets/806c061f-7be5-4399-b109-5a652dd94678)

### [3] Create Project Directory and Set Up Virtual Environment  

I then created the project directory, `/opt/wwc/mysites`, and set up a Python virtual environment inside it:
```bash
sudo bash  # Switch to root
mkdir -p /opt/wwc/mysites  # Create the project directory
cd /opt/wwc/mysites
python3 -m venv myvenv  # Create virtual environment
source myvenv/bin/activate  # Activate the virtual environment
```

![image](https://github.com/user-attachments/assets/2edd7cf5-85a8-462d-b31d-61044f557631)


### [4] Install Django, Create Django Project, and Django App
Inside the virtual environment, I installed Django, started a new Django project called `lab`, and created an app called `polls`:
```bash
pip install django
django-admin startproject lab
cd lab
python3 manage.py startapp polls
```
I checked the contents to ensure everything was set up properly:
```bash
ls -l
```
![image](https://github.com/user-attachments/assets/0f2ece15-fd1f-48b8-9bbc-50d109a00eb3)

## Setting Up nginx as Reverse Proxy

### [5] Install nginx
To handle incoming HTTP requests and forward them to the Django app, I installed nginx:
```
sudo apt install nginx
```
### [6] Configure nginx
I edited the nginx configuration file to set it up as a reverse proxy that forwards traffic from port 80 to the Django app running on port 8000:
```bash
sudo nano /etc/nginx/sites-enabled/default
```
Replace the file content with:
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

This configuration ensures that any traffic received on port 80 is forwarded to the Django app running locally on port 8000.

### [7] Restart nginx
After editing the configuration, I restarted the nginx service to apply the changes:
```
service nginx restart
```

### [8] Access the EC2 Instance

After restarting nginx, we need to run the Django development server to serve the web application. To do this, I navigated to the Django project directory and executed the runserver command:
```bash
cd /opt/wwc/mysites/lab
```
This is where the Django project is located.
Then Run the Django development server on port 8000:
```bash
python3 manage.py runserver 8000
```
This command starts the Django web server locally, binding it to port 8000. The application will now be accessible through nginx, which proxies requests from port 80 (public HTTP) to port 8000 (Django).

![image](https://github.com/user-attachments/assets/59e6cb3c-aa4a-4715-9f28-5921fa6628b8)

Open the browser and enter the public IP address of the EC2 instance to verify the setup. The IP address can be retrieved from the AWS console or via the output from the earlier Python script. Once entered into the browser, nginx forwards the request to Django, and the app is displayed.

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

