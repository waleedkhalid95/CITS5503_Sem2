# Lab 7 : Automating Django Deployment on EC2 Using Fabric

## Summary

In this lab, we automated the deployment of a Django web application on an EC2 instance using Fabric, a Python-based tool for managing and automating tasks on remote servers. The goal was to streamline the deployment process and ensure that each step of setting up the web application was automated and idempotent (able to run multiple times without errors). The deployment included creating an EC2 instance, installing and configuring Python’s virtual environment, installing Django and nginx, and setting up the Django project. Additionally, the lab covered configuring nginx as a reverse proxy to forward traffic to the Django app, creating necessary files (like `views.py`, `urls.py`), and starting the Django development server in the background. Finally, we accessed the application through the public DNS of the EC2 instance to verify that the setup was successful.

### Create an EC2 instance

The first step in this lab was to use a Python script (`create_ec2.py`) to create an EC2 instance where the Django app would be deployed. This script was based on previous labs, where we automated the creation of EC2 instances and set up security groups for HTTP and SSH access.
To run the script:
```bash
python3 create_ec2.py
```
![Screenshot 2024-10-09 195357](https://github.com/user-attachments/assets/28e5a71a-a18c-4d9f-8db2-44548aa20e83)

Once the instance is created, we retrieve the public DNS of the instance from the AWS console, which in this case was `ec2-15-152-34-101.ap-northeast-3.compute.amazonaws.com`.

### Install and configure Fabric 
Fabric is a Python tool that allows you to execute commands on remote servers via SSH, which is useful for automating tasks like software deployment.

To install Fabric, run the following command in your local machine:
```bash
pip install fabric
```

Next, configure the SSH connection by creating a configuration file in `~/.ssh/config`. This ensures that Fabric can connect to the EC2 instance using the correct key and hostname.

Create the file with the following content:
```bash
nano ~/.ssh/config
```
And add the following configuration:
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

![image](https://github.com/user-attachments/assets/5814b347-ab30-4b26-b232-b7b60ab680f1)


### Test Fabric Connection
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

### Automate Django Deployment Using Fabric

The main task in this lab was to automate the setup of a Django web application on the EC2 instance using Fabric. The Python script provided below ensures that all necessary components (Python virtual environment, nginx, Django) are installed and configured. The script also verifies that steps are not redundantly repeated, making it safe to run multiple times.

```python3
from fabric import Connection
from io import StringIO

# Connect to the EC2 instance
c = Connection('23803313-vm2')

# Step 1: Update and upgrade the system packages
c.sudo('apt-get update -y')
c.sudo('apt-get upgrade -y')

# Step 2: Install Python 3 virtual environment if not installed
c.sudo('apt-get install python3-venv -y')

# Step 3: Install nginx if not installed
c.sudo('apt-get install nginx -y')

# Step 4: Create the project directory and set ownership if not already present
if c.run('test -d /opt/wwc/mysites', warn=True).exited != 0:
    c.sudo('mkdir -p /opt/wwc/mysites')
    c.sudo('chown -R ubuntu:ubuntu /opt/wwc')
    print("Created /opt/wwc/mysites and set ownership to ubuntu.")
else:
    print("/opt/wwc/mysites already exists. Skipping directory creation.")

# Step 5: Set up the Python virtual environment if it doesn't exist
if c.run('test -d /opt/wwc/mysites/myvenv', warn=True).exited != 0:
    c.run('python3 -m venv /opt/wwc/mysites/myvenv')
    print("Created Python virtual environment.")
else:
    print("Virtual environment already exists. Skipping creation.")

# Step 6: Install Django within the virtual environment if not already installed
if c.run('/opt/wwc/mysites/myvenv/bin/pip show django', warn=True).exited != 0:
    c.run('/opt/wwc/mysites/myvenv/bin/pip install django')
    print("Installed Django.")
else:
    print("Django already installed.")

# Step 7: Create Django project 'lab' if it doesn't exist
if c.run('test -d /opt/wwc/mysites/lab', warn=True).exited != 0:
    with c.cd('/opt/wwc/mysites'):
        c.run('/opt/wwc/mysites/myvenv/bin/django-admin startproject lab')
    print("Created Django project 'lab'.")
else:
    if c.run('test -f /opt/wwc/mysites/lab/manage.py', warn=True).exited == 0:
        print("Django project 'lab' already exists.")
    else:
        c.run('rm -rf /opt/wwc/mysites/lab')
        c.run('/opt/wwc/mysites/myvenv/bin/django-admin startproject lab')
        print("Recreated incomplete Django project 'lab'.")

# Step 8: Create the 'polls' app within the project if it doesn't exist
if c.run('test -d /opt/wwc/mysites/lab/polls', warn=True).exited != 0:
    with c.cd('/opt/wwc/mysites/lab'):
        c.run('../myvenv/bin/python manage.py startapp polls')
    print("Created Django app 'polls'.")
else:
    print("Django app 'polls' already exists.")

# Step 9: Modify nginx configuration to set up a reverse proxy
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
    c.sudo('tee /etc/nginx/sites-enabled/default', in_stream=StringIO(nginx_config))
    c.sudo('service nginx restart')
    print("Updated and restarted nginx.")
else:
    print("nginx configuration is already up-to-date.")

# Step 10: Modify views.py for the 'polls' app
views_py = '''
from django.http import HttpResponse

def index(request):
    return HttpResponse('Hello, world.')
'''
views_py_path = '/opt/wwc/mysites/lab/polls/views.py'
if c.run(f'cat {views_py_path}', warn=True).stdout.strip() != views_py.strip():
    c.put(StringIO(views_py), views_py_path)
    print("Updated views.py for 'polls' app.")

# Step 11: Modify the URLs for the 'polls' app
polls_urls = '''
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
'''
polls_urls_path = '/opt/wwc/mysites/lab/polls/urls.py'
if c.run(f'cat {polls_urls_path}', warn=True).stdout.strip() != polls_urls.strip():
    c.put(StringIO(polls_urls), polls_urls_path)
    print("Updated urls.py for 'polls' app.")

# Step 12: Modify the main project URLs to include the 'polls' app
main_urls = '''
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('admin/', admin.site.urls),
    path('polls/', include('polls.urls')),
]
'''
main_urls_path = '/opt/wwc/mysites/lab/lab/urls.py'
if c.run(f'cat {main_urls_path}', warn=True).stdout.strip() != main_urls.strip():
    c.put(StringIO(main_urls), main_urls_path)
    print("Updated main project URLs.")

# Step 13: Start Django development server in the background
if c.run("ps aux | grep 'manage.py runserver' | grep -v grep", warn=True).exited != 0:
    with c.cd('/opt/wwc/mysites/lab'):
        c.run('nohup ../myvenv/bin/python manage.py runserver 0.0.0.0:8000 &')
    print("Started Django development server.")
else:
    print("Django development server is already running.")
```

**Explanation**
- **System Updates:** The script ensures the EC2 instance has the latest updates and upgrades for security and performance.
- **Virtual Environment:** It checks if Python’s virtual environment module is installed and then creates a virtual environment if it doesn't already exist.
- **Django Installation:** Django is installed only if it isn’t already present in the virtual environment.
- **Django Project and App Creation:** The script verifies the presence of the Django project and app, avoiding redundant creation and handling incomplete setups by removing and recreating them.
- **nginx Setup:** nginx is configured as a reverse proxy, ensuring all HTTP traffic to the instance is directed to Django's development server running on port 8000.
- **Starting the Server:** Finally, the script ensures the Django development server is running in the background using `nohup`.

![image](https://github.com/user-attachments/assets/54727085-65bd-4151-9715-6a82cacfc029)

### Verify the Application

Once the Django development server is running, you can access the web application by visiting the EC2 instance’s public DNS at `http://15.152.34.101/polls/`.

![image](https://github.com/user-attachments/assets/0446ec88-03d6-47ee-a4dd-3bda0c4aba13)

This confirms that the Django application is successfully deployed and running on the EC2 instance.
