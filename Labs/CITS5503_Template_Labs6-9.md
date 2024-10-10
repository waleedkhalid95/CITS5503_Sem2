<div style="display: flex; flex-direction: column; justify-content: center; align-items: center; height: 100vh;">

  <h2>Labs 6-9</h2>
  
  <p>Student ID: 23803313</p>
  <p>Student Name: Waleed Khalid Siraj</p>

</div>

# Lab 6: Deploying a Django Web App with Application Load Balancer

## Summary
In this lab, I deployed a Django web application on an EC2 instance and configured an Application Load Balancer (ALB) to distribute HTTP traffic between instances. The setup ensures both scalability and high availability of the application. The key steps included creating the EC2 instance, setting up security groups to allow traffic, installing and configuring Django, and using nginx as a reverse proxy. Additionally, I created and configured the ALB to perform health checks on the instances to ensure smooth application availability. By the end of the lab, I confirmed that the Django application could be accessed through the ALB's DNS name, verifying successful traffic distribution.

## Set up an EC2 instance

### [1] Create an EC2 Instance with Ubuntu and SSH into It
I modified the Python script used in Lab 2 to create an EC2 instance with both SSH and HTTP traffic allowed. The security group ensures secure access to the instance for remote management (via SSH) and public HTTP access for serving the Django app.
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

## Setting Up Django Application

### [1] Editing Django Files
After installing Django and creating the polls app, I proceeded to modify the necessary Django files to handle HTTP requests and display content. This step verifies that the Django app is properly handling routing and responding to requests.

#### Editing `polls/views.py`
First, I needed to create a view that returns a basic HTTP response.
```bash
nano polls/views.py
```
In `polls/views.py`, I added the following code:
```python3
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello, world.")
```
**Explanation**:
- I imported the `HttpResponse` class from `django.http` to send back an HTTP response when the view is accessed.
- The `index()` function takes an HTTP request and returns an HTTP response with the text "Hello, world." This will be the basic content displayed when we access the `/polls/` URL.
  
![image](https://github.com/user-attachments/assets/db289496-0d65-406b-9ebc-a8dba65d2202)

#### Editing `polls/urls.py` 
Next, I needed to map the view to a URL pattern in the `polls` app.
```bash
nano polls/urls.py
```
In `polls/urls.py`, I added the following code:
```
from django.urls import path
from . import views  # Import the views from the same directory

# Define the URL patterns for the polls app
urlpatterns = [
    path('', views.index, name='index'),  # Map the root URL to the index view
]
```
**Explanation**
- The `urlpatterns` list maps URLs to views. Here, I used the `path` function to route the root URL (`''`) of the `polls app` to the `index()` view we just created in `polls/views.py`.
- The `name='index'` is a URL name that can be referenced in templates or other Django components.

![image](https://github.com/user-attachments/assets/24579883-73a3-44e5-a46e-bb1207f0f5b4)


#### Editing `lab/urls.py`
```bash
nano lab/urls.py
```
In `lab/urls.py`, I added the following code:
```
from django.urls import include, path
from django.contrib import admin

# Define the URL patterns for the project
urlpatterns = [
    path('polls/', include('polls.urls')),  # Include the polls app's URLs
    path('admin/', admin.site.urls),  # Default admin URL
]
```
**Explanation**
- The `path('polls/', include('polls.urls'))` line includes the URL patterns from the `polls` app whenever the `/polls/` URL is accessed. This means that any URL beginning with `/polls/` will be handled by the `polls` app.
- The `admin/` path is the default Django admin interface URL.
- 
![image](https://github.com/user-attachments/assets/9cc66874-f293-42cc-9c81-a2f34b4368fd)

### [2] Running the Django Web Server
Now that the `polls` app has been set up and the URLs are correctly routed, I ran the Django development server to test the application.
```
python3 manage.py runserver 8000
```
This command runs the Django development server on port 8000, binding the server to localhost. However, since we configured nginx as a reverse proxy, nginx will forward traffic from port 80 (HTTP) to port 8000 (Django).
### [3] Accessing the EC2 Instance
With the Django development server running, I accessed the application using the public IP of the EC2 instance and navigated to the `/polls/` endpoint. This verifies that the application is properly set up and working.

I opened a browser and entered the public IP, http://13.208.165.62/polls/, of the EC2 instance with the `/polls/` URL to verify changes

This resulted in the expected "Hello, world." message from the `index()` view.

![image](https://github.com/user-attachments/assets/4bedbd35-4f3e-4a3d-a052-4f0bacdbb46e)


## Setting Up Application Load Balancer

### [1] Creating an Application Load Balancer

To improve scalability and ensure that traffic is distributed across multiple instances, I set up an Application Load Balancer (ALB). The load balancer routes incoming requests to healthy instances, and the health checks ensure that only working instances receive traffic

```python3
import boto3
import time

# Initialize EC2 and ELBv2 clients in the specified region
ec2 = boto3.client('ec2', region_name='ap-northeast-3')
elbv2 = boto3.client('elbv2', region_name='ap-northeast-3')

# Step 1: Get Instance Details and Subnet IDs
# Use filters to find instances created with a specific key name and security group
try:
    response = ec2.describe_instances(
        Filters=[
            {'Name': 'key-name', 'Values': ['23803313-key']},
            {'Name': 'instance.group-name', 'Values': ['23803313-sg']},
            {'Name': 'instance-state-name', 'Values': ['running']}  # Filter for running instances
        ]
    )
    
    # Extract instance IDs and subnet IDs
    instance_ids = []
    subnet_ids = []
    
    for reservation in response['Reservations']:
        for instance in reservation['Instances']:
            instance_id = instance['InstanceId']
            subnet_id = instance['SubnetId']
            instance_ids.append(instance_id)
            subnet_ids.append(subnet_id)
            print(f"Instance ID: {instance_id}, Subnet ID: {subnet_id}")

    if not instance_ids:
        print("No running instances found.")
    else:
        print(f"Found {len(instance_ids)} running instance(s).")

except Exception as e:
    print(f"An error occurred while retrieving instance details: {e}")

# Step 2: Retrieve the Security Group ID
try:
    response_sg = ec2.describe_security_groups(GroupNames=['23803313-sg'])
    security_group_id = response_sg['SecurityGroups'][0]['GroupId']
    print(f"Existing Security Group found: {security_group_id}\n")
except ec2.exceptions.ClientError as e:
    print(f"Unexpected error: {e}\n")

# Step 3: Create an Application Load Balancer (ALB)
try:
    load_balancer = elbv2.create_load_balancer(
        Name='23803313-alb',
        Subnets=subnet_ids,  # Attach to the subnets of the running EC2 instances
        SecurityGroups=[security_group_id],  # Use the same security group
        Scheme='internet-facing',  # Publicly accessible load balancer
        Tags=[
            {'Key': 'Name', 'Value': '23803313-alb'}
        ],
        Type='application',  # Specify this as an application load balancer
        IpAddressType='ipv4'
    )
    lb_arn = load_balancer['LoadBalancers'][0]['LoadBalancerArn']
    print(f"Load Balancer created: {lb_arn}")

except Exception as e:
    print(f"An error occurred while creating the load balancer: {e}")

# Step 4: Create a Target Group for the Load Balancer
try:
    instance_details = ec2.describe_instances(InstanceIds=[instance_ids[0]])
    vpc_id = instance_details['Reservations'][0]['Instances'][0]['VpcId']
    
    # Create a target group for the load balancer
    target_group = elbv2.create_target_group(
        Name='23803313-tg',
        Protocol='HTTP',
        Port=80,
        VpcId=vpc_id,  # Use the VPC of the instances
        HealthCheckProtocol='HTTP',
        HealthCheckPort='80',
        HealthCheckPath='/polls/',  # Specify the path for health checks
        TargetType='instance'
    )
    target_group_arn = target_group['TargetGroups'][0]['TargetGroupArn']
    print(f"Target Group created: {target_group_arn}")

except Exception as e:
    print(f"An error occurred while creating the target group: {e}")

# Step 5: Register EC2 Instances in the Target Group
if 'target_group_arn' in locals():
    try:
        elbv2.register_targets(
            TargetGroupArn=target_group_arn,
            Targets=[{'Id': instance_id} for instance_id in instance_ids]
        )
        print(f"Instances registered to target group: {target_group_arn}")

    except Exception as e:
        print(f"An error occurred while registering targets: {e}")

# Step 6: Create a Listener for the Load Balancer
if 'lb_arn' in locals() and 'target_group_arn' in locals():
    try:
        listener = elbv2.create_listener(
            LoadBalancerArn=lb_arn,  # ARN of the load balancer
            Protocol='HTTP',
            Port=80,
            DefaultActions=[
                {
                    'Type': 'forward',
                    'TargetGroupArn': target_group_arn  # Forward traffic to the target group
                }
            ]
        )
        listener_arn = listener['Listeners'][0]['ListenerArn']
        print(f"Listener created: {listener_arn}")

    except Exception as e:
        print(f"An error occurred while creating the listener: {e}")
else:
    print("Load balancer or target group creation failed. Skipping listener creation.")

```
**Explanation**
- **Instance Details:** The script retrieves the details of running EC2 instances based on filters (key pair, security group, and instance state). This helps gather information such as instance IDs and subnet IDs.
- **Security Group:** The existing security group is retrieved to attach it to the load balancer for consistent access control.
- **Create ALB:** The Application Load Balancer is created with an internet-facing scheme, allowing public access. The subnets of the EC2 instances are associated with the ALB to distribute traffic across them.
- **Target Group:** A target group is created to hold the EC2 instances. It uses the `/polls/` path for health checks, ensuring the app is responsive before directing traffic.
- **Register Instances:** The EC2 instances are registered as targets in the target group. This ensures that incoming traffic is distributed among these instances.
- **Listener:** A listener is created to forward HTTP traffic from port 80 to the target group.

![image](https://github.com/user-attachments/assets/d8395956-f6d7-4cb3-943f-3b59dc3a921e)

### [2] Health Check Configuration

For the target group, I configured the ALB to check the health of the EC2 instances by making a request to the `/polls/` endpoint every 30 seconds. If the response is successful, the instance is marked as healthy.

![image](https://github.com/user-attachments/assets/97a19c78-6b7e-407a-b16f-a31512bd7184)

![image](https://github.com/user-attachments/assets/03dbdb4c-427b-4724-badc-a95b41a5ee30)

### [3] Accessing the Application via ALB
Finally, I accessed the Django app using the DNS name of the ALB. This confirms that the ALB is distributing traffic to the instances correctly and that the app is responsive.

Access the URL: http://23803313-alb-1046759913.ap-northeast-3.elb.amazonaws.com/polls/

![image](https://github.com/user-attachments/assets/70c46249-efbb-4792-b9d8-8364b85e1c39)


<div style="page-break-after: always;"></div>

# Lab 7 : Automating Django Deployment on EC2 Using Fabric

## Summary

In this lab, we focused on automating the deployment of a Django web application on an AWS EC2 instance using Fabric, a Python-based tool for remote server management. The goal was to streamline the deployment process, ensuring that each step is automated and idempotent—able to run multiple times without errors. This involved creating an EC2 instance, installing and configuring necessary software like Python's virtual environment, Django, and nginx, and setting up the Django project and application. We also configured nginx to act as a reverse proxy, forwarding traffic to the Django app running on port 8000. Finally, we verified the deployment by accessing the application through the EC2 instance's public DNS.

### Create an EC2 instance

The first step in this lab was to use a Python script (`create_ec2.py`) to create an EC2 instance where the Django app would be deployed. This script was based on previous labs, where we automated the creation of EC2 instances and set up security groups for HTTP and SSH access.
To run the script:
```bash
python3 create_ec2.py
```
![Screenshot 2024-10-09 195357](https://github.com/user-attachments/assets/28e5a71a-a18c-4d9f-8db2-44548aa20e83)

Once the instance is created, we retrieve the public DNS of the instance from the AWS console, which in this case was `ec2-15-152-34-101.ap-northeast-3.compute.amazonaws.com`. This DNS address will be used to connect to the instance and to access the deployed application.

### Install and configure Fabric 
Fabric is a powerful tool that allows us to automate tasks on remote servers via SSH. It simplifies the deployment process by allowing us to write Python scripts that execute shell commands on the remote server.

To install Fabric, run the following command in your local machine:
```bash
pip install fabric
```

Next, configure the SSH connection by creating a configuration file in `~/.ssh/config`. This ensures that Fabric can connect to the EC2 instance using the correct key and hostname.

Create the file with the following content:
To allow Fabric to connect to the EC2 instance without manual intervention, we configured SSH by creating a file at `~/.ssh/config`:
```bash
nano ~/.ssh/config
```
And added the following content:
```
Host 23803313-vm2
    Hostname ec2-15-152-34-101.ap-northeast-3.compute.amazonaws.com
    User ubuntu
    UserKnownHostsFile /dev/null
    StrictHostKeyChecking no
    PasswordAuthentication no
    IdentityFile "/home/kali/Desktop/cloud-lab/Lab 7/23803313-key.pem"
```
This configuration allows Fabric to securely connect to the EC2 instance without being prompted for a password or having to manually accept the host key.
This configuration specifies:
- Host: An alias (`23803313-vm2`) to refer to the EC2 instance.
- Hostname: The public DNS of the EC2 instance.
- User: The username to log in as (`ubuntu` for Ubuntu instances).
- IdentityFile: The path to the private key file for SSH authentication.

Other settings like `UserKnownHostsFile` and `StrictHostKeyChecking` are set to avoid SSH prompts that could interfere with automation.

![image](https://github.com/user-attachments/assets/5814b347-ab30-4b26-b232-b7b60ab680f1)


### Testing the Fabric Connection
To test the connection, use the following Fabric commands to connect to the EC2 instance and run a basic command to ensure the connection works:
```bash
python3
>>> from fabric import Connection
>>> c = Connection('23803313-vm2')
>>> result = c.run('uname -s')
Linux
>>>
```

![image](https://github.com/user-attachments/assets/3cad348b-db9a-4c6c-ba59-953a430d4623)

This confirmed that we could execute commands on the remote server using Fabric.
### Automating Deployment with Fabric

We wrote a Python script using Fabric to automate the deployment of the Django application. The script ensures that all necessary components are installed and configured, and it includes checks to prevent redundant actions, making it safe to run multiple times.

```python3
from fabric import Connection
from io import StringIO

# Connect to the EC2 instance using the configured host alias
c = Connection('23803313-vm2')

# Step 1: Update and upgrade the system packages
# This ensures the system is up-to-date and secure
c.sudo('apt-get update -y')
c.sudo('apt-get upgrade -y')

# Step 2: Install Python 3 virtual environment
# Installs the 'python3-venv' package if not already installed
c.sudo('apt-get install python3-venv -y')

# Step 3: Install nginx
# Installs nginx web server to act as a reverse proxy
c.sudo('apt-get install nginx -y')

# Step 4: Create the project directory and set ownership
# Check if the project directory exists
if c.run('test -d /opt/wwc/mysites', warn=True).exited != 0:
    # Directory does not exist; create it
    c.sudo('mkdir -p /opt/wwc/mysites')
    # Change ownership to 'ubuntu' to avoid permission issues
    c.sudo('chown -R ubuntu:ubuntu /opt/wwc')
    print("Created /opt/wwc/mysites and set ownership to ubuntu.")
else:
    print("/opt/wwc/mysites already exists. Skipping directory creation.")

# Step 5: Set up the Python virtual environment
# Check if the virtual environment exists
if c.run('test -d /opt/wwc/mysites/myvenv', warn=True).exited != 0:
    # Create the virtual environment
    c.run('python3 -m venv /opt/wwc/mysites/myvenv')
    print("Created Python virtual environment.")
else:
    print("Virtual environment already exists. Skipping creation.")

# Step 6: Install Django within the virtual environment
# Check if Django is installed
if c.run('/opt/wwc/mysites/myvenv/bin/pip show django', warn=True).exited != 0:
    # Install Django
    c.run('/opt/wwc/mysites/myvenv/bin/pip install django')
    print("Installed Django.")
else:
    print("Django already installed.")

# Step 7: Create the Django project 'lab'
# Check if the project directory exists
if c.run('test -d /opt/wwc/mysites/lab', warn=True).exited != 0:
    # Create the Django project
    with c.cd('/opt/wwc/mysites'):
        c.run('/opt/wwc/mysites/myvenv/bin/django-admin startproject lab')
    print("Created Django project 'lab'.")
else:
    # Check if 'manage.py' exists to confirm the project is complete
    if c.run('test -f /opt/wwc/mysites/lab/manage.py', warn=True).exited == 0:
        print("Django project 'lab' already exists.")
    else:
        # Incomplete project; remove and recreate
        c.run('rm -rf /opt/wwc/mysites/lab')
        with c.cd('/opt/wwc/mysites'):
            c.run('/opt/wwc/mysites/myvenv/bin/django-admin startproject lab')
        print("Recreated incomplete Django project 'lab'.")

# Step 8: Create the 'polls' app within the project
# Check if the 'polls' app exists
if c.run('test -d /opt/wwc/mysites/lab/polls', warn=True).exited != 0:
    with c.cd('/opt/wwc/mysites/lab'):
        c.run('../myvenv/bin/python manage.py startapp polls')
    print("Created Django app 'polls'.")
else:
    print("Django app 'polls' already exists.")

# Step 9: Configure nginx as a reverse proxy
nginx_config = '''
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    location / {
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_pass http://127.0.0.1:8000;
    }
}
'''

# Check if nginx configuration needs updating
current_nginx_config = c.sudo('cat /etc/nginx/sites-enabled/default', hide=True).stdout.strip()
if current_nginx_config != nginx_config.strip():
    # Update nginx configuration
    c.sudo('tee /etc/nginx/sites-enabled/default', in_stream=StringIO(nginx_config))
    # Restart nginx to apply changes
    c.sudo('service nginx restart')
    print("Updated and restarted nginx.")
else:
    print("nginx configuration is already up-to-date.")

# Step 10: Modify views.py for the 'polls' app
views_py_content = '''
from django.http import HttpResponse

def index(request):
    return HttpResponse('Hello, world.')
'''
views_py_path = '/opt/wwc/mysites/lab/polls/views.py'

# Check if views.py needs updating
existing_views_py = c.run(f'cat {views_py_path}', warn=True, hide=True).stdout.strip()
if existing_views_py != views_py_content.strip():
    # Update views.py
    c.put(StringIO(views_py_content), views_py_path)
    print("Updated views.py for 'polls' app.")
else:
    print("views.py is already up-to-date.")

# Step 11: Modify the URLs for the 'polls' app
polls_urls_content = '''
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
'''
polls_urls_path = '/opt/wwc/mysites/lab/polls/urls.py'

# Check if urls.py needs updating
existing_polls_urls = c.run(f'cat {polls_urls_path}', warn=True, hide=True).stdout.strip()
if existing_polls_urls != polls_urls_content.strip():
    # Update urls.py
    c.put(StringIO(polls_urls_content), polls_urls_path)
    print("Updated urls.py for 'polls' app.")
else:
    print("urls.py is already up-to-date.")

# Step 12: Modify the main project URLs to include the 'polls' app
main_urls_content = '''
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('admin/', admin.site.urls),
    path('polls/', include('polls.urls')),
]
'''
main_urls_path = '/opt/wwc/mysites/lab/lab/urls.py'

# Check if main urls.py needs updating
existing_main_urls = c.run(f'cat {main_urls_path}', warn=True, hide=True).stdout.strip()
if existing_main_urls != main_urls_content.strip():
    # Update main urls.py
    c.put(StringIO(main_urls_content), main_urls_path)
    print("Updated main project URLs.")
else:
    print("Main project URLs are already up-to-date.")

# Step 13: Start the Django development server in the background
# Check if the server is already running
if c.run("pgrep -f 'manage.py runserver'", warn=True).exited != 0:
    # Server is not running; start it
    with c.cd('/opt/wwc/mysites/lab'):
        c.run('nohup ../myvenv/bin/python manage.py runserver 0.0.0.0:8000 &', hide=True)
    print("Started Django development server.")
else:
    print("Django development server is already running.")
```

**Explanation**
- **Connection Setup:** We connect to the EC2 instance using the alias `23803313-vm2` specified in the SSH config.
- **System Updates:** We update and upgrade system packages using `apt-get` to ensure the instance is secure and up-to-date.
- Software Installation: We install `python3-venv` for creating virtual environments and `nginx` to serve as a reverse proxy.
- **Project Directory Setup:** We check if the project directory `/opt/wwc/mysites` exists. If not, we create it and set the ownership to the `ubuntu` user to avoid permission issues.
- **Virtual Environment Setup:** We check for the existence of the virtual environment directory. If it doesn't exist, we create it using `python3 -m venv`.
- **Django Installation:** We verify if Django is installed within the virtual environment by attempting to display its package information with `pip show django`. If not installed, we install it.
- **Django Project and App Creation:**
  - We check if the Django project `lab` exists by verifying the presence of `/opt/wwc/mysites/lab`.
  - If the project directory exists but is incomplete (missing `manage.py`), we remove it and recreate the project.
  - We check for the `polls` app and create it if it doesn't exist.
- **nginx Configuration:**
  - We define the desired nginx configuration to set it up as a reverse proxy.
  - We compare the existing nginx configuration with the desired one. If they differ, we update the configuration and restart nginx.
- **Django Application Configuration:**
  - We update `views.py`, `urls.py` for the `polls` app, and the main `urls.py` file to ensure they contain the correct code.
  - Before updating, we check if the existing files already have the desired content to avoid unnecessary writes.
- Starting the Django Development Server:
  - We check if the Django development server is already running using `pgrep`.
  - If not running, we start it in the background using `nohup`.

Note: We use the `warn=True` parameter in `c.run()` commands to prevent the script from stopping when a command exits with a non-zero status, which is essential for existence checks.

![image](https://github.com/user-attachments/assets/54727085-65bd-4151-9715-6a82cacfc029)

### Verify the Application

After running the Fabric script, we verified that the Django application was successfully deployed and running.

We accessed the application using the EC2 instance's public IP address and the /polls/ endpoint: `http://15.152.34.101/polls/`

![image](https://github.com/user-attachments/assets/0446ec88-03d6-47ee-a4dd-3bda0c4aba13)

The page displayed "Hello, world.", confirming that the application was working as expected.


<div style="page-break-after: always;"></div>

# Lab 8: Hyperparameter Tuning with AWS SageMaker

## Summary
In this lab, we explored how to perform hyperparameter tuning using AWS SageMaker to optimize a machine learning model. We set up and ran hyperparameter tuning jobs on AWS SageMaker, utilizing the XGBoost algorithm on the bank marketing dataset from the UCI Machine Learning Repository. The process involved installing necessary libraries, preparing the dataset, configuring the hyperparameter tuning job, and analyzing the results.

## Setting Up the Environment
### Install and run jupyter notebooks
First, we need to install Jupyter Notebook, which provides an interactive environment to write and execute Python code.
```
pip install notebook
python3 -m notebook
```
This will start the Jupyter Notebook server, and you can access it through your web browser.

### Install ipykernel
To ensure that Jupyter Notebook can properly handle Python environments and kernels, we install the `ipykernel` package.
```
pip install ipykernel
```
This package allows us to manage different Python kernels within Jupyter Notebook.

### Install Necessary Libraries in Jupyter Notebook
Within the Jupyter Notebook, we need to install the essential libraries required for this lab: SageMaker, pandas, and numpy.

In a Jupyter Notebook cell, run:
```python3
# Install SageMaker via jupyter notebook
!pip install sagemaker
# Install pandas and numpy jupyter notebook
!pip install pandas
!pip install numpy
```
These libraries are crucial for interacting with AWS SageMaker and handling data processing tasks

## AWS IAM Role Setup

In order to interact with AWS SageMaker, we need an IAM role with the necessary permissions. In the AWS Console, navigate to **IAM > Roles**, and find the existing `SageMakerRole`. This role should have the required permissions to run SageMaker jobs.

![image](https://github.com/user-attachments/assets/c6f3c855-0ab3-4a4a-8cec-39e313f33c14)

Note: If you don't have permissions to create a new role, you can proceed with an existing one.

## Preparing the AWS SageMaker Session
We set up a SageMaker session to communicate with the SageMaker service and define key variables like the AWS region, IAM role, and S3 bucket
```python3
import sagemaker
import boto3
import numpy as np
import pandas as pd
from time import gmtime, strftime
import os

# Initialize SageMaker client
smclient = boto3.Session().client("sagemaker")

# Initialize IAM client
iam = boto3.client('iam')

# Get SageMaker role ARN
sagemaker_role = iam.get_role(RoleName='SageMakerRole')['Role']['Arn']

# Specify AWS region
region = 'ap-northeast-3'  # Replace with your AWS region

# Define your student ID
student_id = "23803313"  # Replace with your student ID

# Define S3 bucket name
bucket = f'{student_id}-lab8'  # Format: <studentid>-lab8

# Define prefix for S3 paths
prefix = f"sagemaker/{student_id}-hpo-xgboost-dm"
```
**Explanation**
- We import the necessary libraries.
- `smclient` and `iam` are clients to interact with SageMaker and IAM services
- `sagemaker_role` retrieves the ARN of the SageMaker role.
- `region`, `student_id`, `bucket`, and `prefix` are variables we'll use throughout the lab.

## Create S3 Bucket
We need an S3 bucket to store the training and validation data for our SageMaker jobs.
```python3
# Initialize S3 resource
s3 = boto3.resource('s3')

# Attempt to create the S3 bucket
try:
    s3.create_bucket(Bucket=bucket, CreateBucketConfiguration={'LocationConstraint': region})
    print(f"S3 bucket '{bucket}' created.")
except s3.meta.client.exceptions.BucketAlreadyOwnedByYou:
    print(f"S3 bucket '{bucket}' already exists and is owned by you.")

# Create a folder (prefix) in the S3 bucket
s3.Bucket(bucket).put_object(Key=(prefix + '/'))
print(f"S3 prefix '{prefix}/' created.")
```
**Explanation**
- We initialize an S3 resource.
- We attempt to create a new S3 bucket with the specified name and region.
- If the bucket already exists and is owned by us, we catch the exception and proceed.
- We create a folder in the bucket using the specified prefix.
- 
![image](https://github.com/user-attachments/assets/3603c86b-bfab-4e19-a5d1-fab41410958e)

## Download and Load Dataset
We use the bank marketing dataset from the UCI Machine Learning Repository.
```python3
# Download the dataset
!wget -N https://archive.ics.uci.edu/ml/machine-learning-databases/00222/bank-additional.zip

# Unzip the dataset
!unzip -o bank-additional.zip

# Load the dataset into a Pandas DataFrame
data = pd.read_csv("./bank-additional/bank-additional-full.csv", sep=";")

# Display all columns and rows
pd.set_option("display.max_columns", None)
pd.set_option("display.max_rows", None)

# Preview the first few rows
data.head()
```
**Explanation**
- We use `wget` to download the dataset zip file
- We unzip the file to extract the CSV data.
- We read the CSV file into a Pandas DataFrame.
- We adjust Pandas settings to display all columns and rows for better visibility.
- We display the first few rows to get an initial look at the data.
  
![image](https://github.com/user-attachments/assets/df8a9261-17be-42aa-813b-8a15fc0304ee)

## Data Preprocessing
### Handling Categorical Variables
We identify categorical and numeric variables and process the data by adding new indicator columns and converting categorical variables into dummy variables for modeling.
```python3
# Identify categorical variables
categorical_vars = data.select_dtypes(include=['object']).columns.tolist()
print("Categorical Variables:", categorical_vars[:4])  # Display first four

# Identify numerical variables
numerical_vars = data.select_dtypes(include=[np.number]).columns.tolist()
print("Numerical Variables:", numerical_vars[:4])  # Display first four
```
**Explanation**
- We use `select_dtypes` to select columns of specific data types
- `include=['object']` selects columns with string data types, which are our categorical variables.
- `include=[np.number]` selects columns with numerical data types.
  
![image](https://github.com/user-attachments/assets/7956c9f3-ea82-4aa2-b291-61b6d529ca2d)

### Adding Indicator Variables
Process the data by adding two new indicator columns and then expands categorical columns into binary dummy columns for modelling purposes
```python3
# Add indicator columns
data["no_previous_contact"] = np.where(data["pdays"] == 999, 1, 0)
data["not_working"] = np.where(data["job"].isin(["student", "retired", "unemployed"]), 1, 0)

# Convert categorical variables to dummy/indicator variables
model_data = pd.get_dummies(data)

# Display the first few rows of the processed data
model_data.head()
```
**Explanation**
- `pdays` equal to 999 indicates that the client was not previously contacted; we create a binary variable `no_previous_contact`.
- We identify clients who are students, retired, or unemployed and create a binary variable `not_working`.
- `pd.get_dummies(data)` automatically converts all categorical variables into one-hot encoded variables
- This process creates new binary columns for each category in the categorical variables
  
![image](https://github.com/user-attachments/assets/406317e7-6758-43ef-b499-fd2fdce0ceb7)

### Removing Unnecessary Variables
We also removed economic variables and checked for any non-numeric values, converting boolean columns to integers as necessary.

```python3
model_data = model_data.drop(
    ["duration", "emp.var.rate", "cons.price.idx", "cons.conf.idx", "euribor3m", "nr.employed"],
    axis=1,
)
model_data.head()
```
**Explanation**
- These variables are removed because they may not be available in a real-world scenario when making predictions, or they might not be relevant for our model.

![image](https://github.com/user-attachments/assets/f99d9941-3acd-480b-9d33-4aae5414262b)

### Ensuring All Data is Numeric

We ensure that all variables are numeric, converting any non-numeric columns.
```python3
# Check for non-numeric columns
non_numeric_columns = model_data.select_dtypes(include=['object', 'bool']).columns.tolist()
print("Non-numeric columns:", non_numeric_columns)

# Convert boolean columns to integers if any
for col in non_numeric_columns:
    if model_data[col].dtype == 'bool':
        model_data[col] = model_data[col].astype(int)
    elif model_data[col].dtype == 'object':
        # Convert 'True'/'False' strings to 1/0
        model_data[col] = model_data[col].map({'True': 1, 'False': 0})

# Verify all columns are now numeric
non_numeric_columns = model_data.select_dtypes(include=['object', 'bool']).columns.tolist()
print("Non-numeric columns after conversion:", non_numeric_columns)
```
**Explanation**
- We identify any non-numeric columns.
- We convert boolean columns to integers (True -> 1, False -> 0).
- For object columns, we map 'yes' and 'no' to 1 and 0, respectively.
- We verify that all columns are now numeric.
  
![image](https://github.com/user-attachments/assets/19786a9d-adae-491e-a205-8f3765e656c7)

## Split Data into Training, Validation, and Test Sets
We split the dataset into training (70%), validation (20%), and test (10%) datasets and convert the datasets to an appropriate format. We will use the training and validation datasets during training. Test dataset will be used to evaluate model performance after it is deployed to an endpoint.

Amazon SageMaker's XGBoost algorithm expects data in the libSVM or CSV data format. In this lab, we use the CSV format. Note that the first column must be the target variable and the CSV should not include headers. Also, notice that although repetitive it’s easier to do this after the train|validation|test split rather than before. This avoids any misalignment issues due to random reordering.
```python3
train_data, validation_data, test_data = np.split(
    model_data.sample(frac=1, random_state=1729),
    [int(0.7 * len(model_data)), int(0.9 * len(model_data))],
)

pd.concat([train_data["y_yes"], train_data.drop(["y_no", "y_yes"], axis=1)], axis=1).to_csv(
    "train.csv", index=False, header=False
)
pd.concat(
    [validation_data["y_yes"], validation_data.drop(["y_no", "y_yes"], axis=1)], axis=1
).to_csv("validation.csv", index=False, header=False)
pd.concat([test_data["y_yes"], test_data.drop(["y_no", "y_yes"], axis=1)], axis=1).to_csv(
    "test.csv", index=False, header=False
)
```
**Explanation**
- We use `np.split` to split the data into 70% training, 20% validation, and 10% test sets.
- `model_data.sample(frac=1, random_state=1729)` shuffles the data before splitting.
- We specify indices to split the data appropriately.
- We place the target variable y_yes as the first column
- We drop the y_no column since y_yes and y_no are complements
- We save the datasets to CSV files without headers and indices, as required by XGBoost in CSV format
  
## Upload Data to S3
We upload the processed training and validation datasets to the S3 bucket for use in SageMaker training jobs.

```python3
# Upload training data to S3
boto3.Session().resource("s3").Bucket(bucket).Object(
    os.path.join(prefix, "train/train.csv")
).upload_file("train.csv")

# Upload validation data to S3
boto3.Session().resource("s3").Bucket(bucket).Object(
    os.path.join(prefix, "validation/validation.csv")
).upload_file("validation.csv")
```
**Explanation**
- We use the `upload_file` method to upload the local CSV files to the specified S3 paths.
- The data is stored in the `train` and `validation` folders within the S3 bucket.

## Configuring the Hyperparameter Tuning Job
We define the configuration for the hyperparameter tuning job, including parameter ranges, resource limits, and tuning strategy.
```python3
from time import gmtime, strftime, sleep

# Define a unique tuning job name
tuning_job_name = f"{student_id}-xgboost-tuningjob-02"

print(tuning_job_name)

# Define the hyperparameter ranges
tuning_job_config = {
    "ParameterRanges": {
        "CategoricalParameterRanges": [],
        "ContinuousParameterRanges": [
            {
                "MaxValue": "1",
                "MinValue": "0",
                "Name": "eta",
            },
            {
                "MaxValue": "10",
                "MinValue": "1",
                "Name": "min_child_weight",
            },
            {
                "MaxValue": "2",
                "MinValue": "0",
                "Name": "alpha",
            },
        ],
        "IntegerParameterRanges": [
            {
                "MaxValue": "10",
                "MinValue": "1",
                "Name": "max_depth",
            }
        ],
    },
    "ResourceLimits": {"MaxNumberOfTrainingJobs": 2, "MaxParallelTrainingJobs": 2},
    "Strategy": "Bayesian",
    "HyperParameterTuningJobObjective": {"MetricName": "validation:auc", "Type": "Maximize"},
}
```
**Explanation**
- We create a unique tuning job name using the current time.
- We define the hyperparameters to tune:
  - `eta`: Learning rate.
  - `min_child_weight`: Minimum sum of instance weight needed in a child.
  - `alpha`: L1 regularization term on weights.
  - `max_depth`: Maximum depth of a tree.
- We set the resource limits:
  - `MaxNumberOfTrainingJobs`: Total number of training jobs to run.
  - `MaxParallelTrainingJobs`: Number of training jobs to run in parallel.
- We specify the optimization strategy and objective metric.
  
## Specifying the XGBoost Algorithm 
We specify the details of the training job definition, including the algorithm and data sources
```python3
from sagemaker.image_uris import retrieve

# Retrieve the XGBoost image URI
training_image = retrieve(framework="xgboost", region=region, version="latest")

# Define S3 input paths for training and validation data
s3_input_train = f"s3://{bucket}/{prefix}/train"
s3_input_validation = f"s3://{bucket}/{prefix}/validation"

# Define the training job parameters
training_job_definition = {
    "AlgorithmSpecification": {
        "TrainingImage": training_image,
        "TrainingInputMode": "File"
    },
    "InputDataConfig": [
        {
            "ChannelName": "train",
            "DataSource": {
                "S3DataSource": {
                    "S3Uri": s3_input_train,
                    "S3DataType": "S3Prefix",
                    "S3DataDistributionType": "FullyReplicated"
                }
            },
            "ContentType": "csv",
            "CompressionType": "None"
        },
        {
            "ChannelName": "validation",
            "DataSource": {
                "S3DataSource": {
                    "S3Uri": s3_input_validation,
                    "S3DataType": "S3Prefix",
                    "S3DataDistributionType": "FullyReplicated"
                }
            },
            "ContentType": "csv",
            "CompressionType": "None"
        }
    ],
    "OutputDataConfig": {
        "S3OutputPath": f"s3://{bucket}/{prefix}/output"
    },
    "ResourceConfig": {
        "InstanceType": "ml.m5.xlarge",
        "InstanceCount": 1,
        "VolumeSizeInGB": 10
    },
    "RoleArn": sagemaker_role,
    "StaticHyperParameters": {
        "objective": "binary:logistic",
        "num_round": "100",
        "eval_metric": "auc",
        "verbosity": "1"
    },
    "StoppingCondition": {
        "MaxRuntimeInSeconds": 3600
    },
}
```
**Explanation**
- We retrieve the XGBoost image URI for the specified region and version.
- We define the input data configurations for the training and validation datasets.
- We specify the output data configuration, resource configuration, and static hyperparameters.
- The static hyperparameters include:
  - `objective`: The learning task and the corresponding learning objective.
  - `num_round`: Number of boosting rounds.
  - `eval_metric`: Evaluation metric.
- We set the stopping condition to limit the maximum runtime.
  
## Launching the Hyperparameter Tuning Job
We launch the hyperparameter tuning job using the configurations defined.
```pyhton3
# Launch the hyperparameter tuning job
smclient.create_hyper_parameter_tuning_job(
    HyperParameterTuningJobName=tuning_job_name,
    HyperParameterTuningJobConfig=tuning_job_config,
    TrainingJobDefinition=training_job_definition,
)

print(f"Hyperparameter tuning job '{tuning_job_name}' launched successfully.")
```
**Explanation**
- We call `create_hyper_parameter_tuning_job` with the tuning job name, configuration, and training job definition.

![image](https://github.com/user-attachments/assets/9b1298d8-c420-48c6-ad91-2180f160188d)

## Verifying Job Completion
I verified the successful completion of the hyperparameter tuning job in the SageMaker console.

![image](https://github.com/user-attachments/assets/16e04331-f559-43c4-b898-1911b96c1cbd)

## Check Tuning Job Output
I reviewed the output of the tuning job to evaluate its performance and ensure the optimal hyperparameters were identified.

![image](https://github.com/user-attachments/assets/16a98776-ea69-4802-8611-1787a51396f5)


<div style="page-break-after: always;"></div>

# Lab 9: Working with AWS Comprehend and Rekognition
## Summary

In this lab, we explored the capabilities of **AWS Comprehend** and **AWS Rekognition** services using Boto3 in a Jupyter Notebook environment.
- **AWS Comprehend:** We performed various natural language processing (NLP) tasks such as detecting the dominant language of a text, analyzing sentiment, detecting entities, extracting key phrases, and analyzing syntax using example texts in different languages like English, Spanish, French, and Italian.
- **AWS Rekognition:** We created an S3 bucket to store images and utilized image processing tasks such as label detection, facial analysis, and text extraction. We encountered some image format compatibility issues, which we resolved by converting images to JPEG format, and we also handled exceptions to troubleshoot errors during image processing tasks.

## AWS Comprehend

**AWS Comprehend** is a natural language processing service that uses machine learning to discover insights from text. It can perform tasks such as detecting the dominant language, sentiment analysis, entity recognition, key phrase extraction, and syntax analysis.

AWS Comprehend is not available in the `ap-northeast-3` region, so we used the `ap-south-1` region for this lab.

### Setting Up AWS Comprehend
First, we need to import the Boto3 library and set up the AWS Comprehend client.
```python3
import boto3

# Create a Comprehend client in the 'ap-south-1' region
client = boto3.client('comprehend', region_name='ap-south-1')
```
**Explanation**
- We import the `boto3` library, which is AWS's SDK for Python.
- We create a client for AWS Comprehend, specifying the `region_name` as `ap-south-1`.
  
### Detecting Dominant Language
To get started with AWS Comprehend, we wrote a script to detect the dominant language of a given text. We used an example text about the French Revolution and checked the predicted language using the `detect_dominant_language` function.

```python
# Detect the dominant language of the text
response = client.detect_dominant_language(
    Text="The French Revolution was a period of social and political upheaval in France and its colonies beginning in 1789 and ending in 1799.",
)

# Print the detected languages and their confidence scores
print(response['Languages'])
```
**Explanation**
- We call the `detect_dominant_language` method with our sample text.
- The method returns a list of detected languages with their language codes and confidence scores.

By executing the code above, AWS Comprehend returns a response indicating that the text is in English (`'en'`), with a confidence score of over 99%. 

![image](https://github.com/user-attachments/assets/4a402681-05f6-46a1-9db4-8fb7f4e4c3e4)

### Testing Language Detection in Different Languages

We tested AWS Comprehend with texts in different languages (English, Spanish, French, and Italian) to detect the dominant language. Here's how we implemented this:

```python3
import boto3

def detect_language(text):
    # Create a Comprehend client
    client = boto3.client('comprehend', region_name='ap-south-1')
    # Detect the dominant language
    response = client.detect_dominant_language(Text=text)
    # Extract the language code and confidence score
    lang_code = response['Languages'][0]['LanguageCode']
    confidence = response['Languages'][0]['Score'] * 100
    # Map language codes to language names
    languages = {
        'en': 'English',
        'es': 'Spanish',
        'fr': 'French',
        'it': 'Italian'
    }
    predicted_language = languages.get(lang_code, lang_code)
    # Print the detected language and confidence
    print(f"{predicted_language} detected with {int(confidence)}% confidence")

# Test texts in different languages
texts = [
    "The French Revolution was a period of social and political upheaval in France.",
    "El Quijote es la obra más conocida de Miguel de Cervantes.",
    "Moi je n'étais rien Et voilà qu'aujourd'hui Je suis le gardien.",
    "L'amor che move il sole e l'altre stelle."
]

# Detect language for each text
for text in texts:
    detect_language(text)
```
**Explanation**
- We define a function `detect_language` that takes a text input.
- Within the function, we create a Comprehend client.
- We call `detect_dominant_language` with the input text.
- We use `client.detect_dominant_language` to get the language code and confidence score.
- We map the language codes to human-readable language names.
- We print the text, detected language, language code, and confidence percentage.
- The function is tested on four sample texts in different languages.

![image](https://github.com/user-attachments/assets/b00a2547-c150-4af2-8508-17af96cee384)

This correctly identified the languages as English, Spanish, French, and Italian, respectively.

### Sentiment Analysis 

Next, we used AWS Comprehend's sentiment analysis capability to determine whether a text's sentiment is positive, negative, neutral, or mixed.

```python 3
def analyze_sentiment(text):
    # Create a Comprehend client
    client = boto3.client('comprehend')
    # Analyze the sentiment of the text
    response = client.detect_sentiment(Text=text, LanguageCode='en')
    # Return the detected sentiment
    return f"Sentiment: {response['Sentiment']}"
for text in texts:
    print(analyze_sentiment(text))
```
**Explanation:**
- Define a function `analyze_sentiment` that takes a text string as input.
- Create a Comprehend client.
- Call `detect_sentiment` with the input text and specify the language code as `'en'` (English).
- We extract the sentiment from the response.
- We return the sentiment as a string.
- We use a list of English texts to test sentiment analysis.
- We print the sentiment analysis results.

AWS Comprehend's sentiment analysis currently supports English, Spanish, French, German, Italian, Portuguese, and Japanese. Therefore, we ensure that the texts used for sentiment analysis are in supported languages.

![image](https://github.com/user-attachments/assets/9367425e-535d-4a4b-9f0a-e86f3e718c81)

This provided sentiment classifications for each of the English texts.

### Entity Detection
`Entities` are specific items of information in a text that represent real-world objects or concepts like people, places, organizations, dates, quantities, and other tangible things mentioned in a sentence.
Entity detection identifies named entities like people, places, organizations, dates, and quantities within the text. We applied entity detection to the texts using AWS Comprehend.
```python3
def detect_entities(text):
    client = boto3.client('comprehend')
    response = client.detect_entities(Text=text, LanguageCode='en')
    return [(entity['Text'], entity['Type'], int(entity['Score'] * 100)) for entity in response['Entities']]
for text in texts:
    print(detect_entities(text))
```
**Explanation**
- Define a function detect_entities that takes a text string as input.
- Create a Comprehend client.
- Call `detect_entities` with the input text and specify the language code as `'en'`.
- We extract the list of entities from the response.
- We return a list of tuples containing the entity text, type, and confidence score.
- We use an example text about Amazon Web Services for entity detection.
- We print each detected entity with its type and confidence score.

This function identified entities such as names, dates, and places in the English text.

![image](https://github.com/user-attachments/assets/1c0ef251-0962-4db3-aa05-ae12ba180759)

### Key Phrase Detection
`Key phrases` are the main ideas or significant expressions in a text that capture the meaning of what is being communicated. They are words or combinations of words that are meaningful or relevant.
We used AWS Comprehend to detect key phrases within a text. Key phrases help identify the most important information in a sentence.
```python3
def detect_key_phrases(text):
    client = boto3.client('comprehend')
    response = client.detect_key_phrases(Text=text, LanguageCode='en')
    return [(phrase['Text'], int(phrase['Score'] * 100)) for phrase in response['KeyPhrases']]
for text in texts:
    print(detect_key_phrases(text))
```
**Explanation:**
- Define a function `detect_key_phrases` that takes a text string as input.
- Create a Comprehend client.
- Call `detect_key_phrases` with the input text and specify the language code as `'en'`.
- Extract the list of key phrases from the response.
- Return a list of tuples containing the key phrase text and confidence score.
- We use an example text about machine learning for key phrase detection.
- We print each detected key phrase with its confidence score.
  
![image](https://github.com/user-attachments/assets/fd5f7ad6-1526-48d3-92b6-59e2cef28b8f)

### Syntax Analysis
`Syntax` refer to the grammatical structure of sentences—the way words are arranged and connected to convey meaning. Syntax analysis involves identifying the part of speech for each word in a sentence, such as nouns, verbs, adjectives, and so on.

```pyhton3
def detect_syntax(text):
    client = boto3.client('comprehend')
    response = client.detect_syntax(Text=text, LanguageCode='en')
    return [(token['Text'], token['PartOfSpeech']['Tag']) for token in response['SyntaxTokens']]
for text in texts:
    print(detect_syntax(text))
```
**Explanation:**
- Define a function `detect_syntax` that takes a text string as input.
- Create a Comprehend client.
- Call `detect_syntax` with the input text and specify the language code as `'en'`.
- Extract the list of syntax tokens from the response.
- We return a list of tuples containing the token text and its part of speech tag.
- We use an example text about natural language processing for syntax analysis.
- We print each token with its part of speech.

![image](https://github.com/user-attachments/assets/d3b333c7-f824-4640-9f8e-b20ae1aff266)

AWS Comprehend identified the parts of speech for each word in the sentence.

## AWS Rekognition

**AWS Rekognition** is a service that analyzes images and videos using machine learning. It can detect objects, scenes, activities, text, faces, and facial expressions in images.

### Creating an S3 Bucket and Uploading Images
Before we can use AWS Rekognition, we need to store our images in an S3 bucket.

#### Set Up S3 Bucket and Upload Images
We created an S3 bucket to store images and uploaded four images to test various Rekognition features: label detection, facial analysis, and text detection
```python3
import boto3

# Initialize the S3 resource
s3 = boto3.resource('s3')

# Define the bucket name and region
bucket_name = '23803313-lab9'  # Use your student ID to name the bucket
region = 'ap-south-1'  # Replace with your AWS region

# Create the S3 bucket
try:
    s3.create_bucket(Bucket=bucket_name, CreateBucketConfiguration={'LocationConstraint': region})
    print(f"S3 bucket '{bucket_name}' created.")
except s3.meta.client.exceptions.BucketAlreadyOwnedByYou:
    print(f"S3 bucket '{bucket_name}' already exists.")
except Exception as e:
    print(f"Error creating bucket: {e}")

# List of images to upload
images = ['urban.jpg', 'beach.jpg', 'faces.jpg', 'text.jpeg']

# Upload images to S3 bucket
for image in images:
    try:
        s3.Bucket(bucket_name).upload_file(image, image)
        print(f"Uploaded {image} to {bucket_name}")
    except Exception as e:
        print(f"Error uploading {image}: {e}")
```
**Explanation:**
- Import the `boto3` library and initialize the S3 resource.
- Define the S3 bucket name and region.
- We attempt to create the S3 bucket. If the bucket already exists and is owned by us, we handle the exception.
- Define a list of image filenames that we want to upload to the S3 bucket.
- Loop over the images and upload each one to the S3 bucket.
- Handle any exceptions that may occur during the upload process.
  
Note: We had to change the format of one of the images to JPEG due to compatibility issues.

![image](https://github.com/user-attachments/assets/9ddff8f5-aca7-41db-92ef-b49c829e3a21)

### Label Detection, Facial Analysis, Text Detection    
We utilized AWS Rekognition to perform label detection, facial analysis, and text detection.

```python3
import boto3

# Initialize Rekognition client
rekognition = boto3.client('rekognition')

# Function to detect labels
def detect_labels(image_name, bucket_name):
    # Call Rekognition to detect labels in the image
    response = rekognition.detect_labels(
        Image={'S3Object': {'Bucket': bucket_name, 'Name': image_name}},
        MaxLabels=10
    )
    # Extract label names and confidence scores
    labels = [(label['Name'], int(label['Confidence'])) for label in response['Labels']]
    return labels

# Function to detect faces
def detect_faces(image_name, bucket_name):
    # Call Rekognition to detect faces in the image
    response = rekognition.detect_faces(
        Image={'S3Object': {'Bucket': bucket_name, 'Name': image_name}},
        Attributes=['ALL']
    )
    # Extract confidence and emotions for each face
    faces = [(int(face['Confidence']), [emotion['Type'] for emotion in face['Emotions']]) for face in response['FaceDetails']]
    return faces

# Function to detect text
def detect_text(image_name, bucket_name):
    # Call Rekognition to detect text in the image
    response = rekognition.detect_text(
        Image={'S3Object': {'Bucket': bucket_name, 'Name': image_name}}
    )
    # Extract detected text and confidence scores
    texts = [(text['DetectedText'], int(text['Confidence'])) for text in response['TextDetections']]
    return texts

# List of images to analyze
images = ['urban.jpg', 'beach.jpg', 'faces.jpg', 'text.jpeg']

# Analyze each image
for image in images:
    # Detect labels in the image
    labels = detect_labels(image, bucket_name)
    print(f"Labels in {image}: {labels}")
    
    # If the image is 'faces.jpg', perform facial analysis
    if image == 'faces.jpg':
        faces = detect_faces(image, bucket_name)
        print(f"Facial analysis for {image}: {faces}")
    
    # If the image is 'text.jpeg', perform text detection
    if image == 'text.jpeg':
        texts = detect_text(image, bucket_name)
        print(f"Text detected in {image}: {texts}")
```
**Explanation**
- Initialized the Rekognition client.
- Defined three functions:
  - `detect_labels`: Detects labels in an image and returns a list of label names and confidence scores.
    - Calls `rekognition.detect_labels` with the specified image from S3.
    - Specifies `MaxLabels=10` to limit the number of labels returned.
    - Extracts the `Name` and `Confidence` for each label detected
    - Returns List of tuples containing label names and confidence scores
  - `detect_faces`: Detects faces in an image and returns confidence scores and detected emotions for each face.
    - Calls `rekognition.detect_faces` with the specified image from S3.
    - Specifies `Attributes=['ALL']` to get detailed facial attributes.
    - Extracts the `Confidence` and Emotions for each face detected.
    - Returns List of tuples containing confidence scores and list of emotions.
  - `detect_text`: Detects text in an image and returns detected text and confidence scores.
    - Calls `rekognition.detect_text` with the specified image from S3.
    - Extracts the `DetectedText` and `Confidence` for each text detected.
    - Returns List of tuples containing detected text and confidence scores.
- Defined a list of images to analyze.
- Iterated over each image:
  - Detected labels in the image and printed the results.
  - If the image is `'faces.jpg'`, performed facial analysis and printed the results.
  - If the image is `'text.jpeg'`, performed text detection and printed the results.
    
![image](https://github.com/user-attachments/assets/14c534c7-1f24-43b6-aed3-04ad37bff392)

We obtained the labels, facial analysis, and text detection results for the respective images.


