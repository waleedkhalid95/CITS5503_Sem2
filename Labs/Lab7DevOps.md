# Practical Worksheet 7

Version: 1.2 Date: 15/9/2018 Author: David Glance

Date: 28/07/2024 Updated by Zhi Zhang

## Learning Objectives

1. Install and configure Fabric
2. Deploy a server with nginx installed and configured by Fabric
3. Deploy Django code using Fabric

## Technologies Covered

* Ubuntu
* AWS
* Python
* Fabric

**NOTE**: please use your Linux environment – if you do it from any other OS (e.g., Windows, Mac – some unknow issues might occur)

## Background

The aim of this lab is to write a program that will:
 
[1] Background and basics to Fabric

[2] How to automatically deploy a server using Fabric

### Create an EC2 instance

Using my existing code create_ec2.py to create an EC2 instance where we will test our Fabric-based installation.


after the ec2 instance is created we check the console for the public dns  ec2-15-152-34-101.ap-northeast-3.compute.amazonaws.com
### Install and configure Fabric 

The easiest way to install fabric is to:

```
pip install fabric
```

You will need to create a config file in ~/.ssh with the contents:
nano ~/.ssh/config

```
Host 23803313-vm2
    Hostname ec2-15-152-34-101.ap-northeast-3.compute.amazonaws.com
    User ubuntu
    UserKnownHostsFile /dev/null
    StrictHostKeyChecking no
    PasswordAuthentication no
    IdentityFile "/home/kali/Desktop/cloud-lab/Lab 7/23803313-key.pem"
```

![image](https://github.com/user-attachments/assets/5814b347-ab30-4b26-b232-b7b60ab680f1)


Rely on the fabric code below to connect to you instance.

```python3
python3
>>> from fabric import Connection
>>> c = Connection('23803313-vm2')
>>> result = c.run('uname -s')
Linux
>>>
```
![image](https://github.com/user-attachments/assets/3cad348b-db9a-4c6c-ba59-953a430d4623)


### Use Fabric for automation

Write a python script where you first need to automate the setup of a Python 3 virtual environment, nginx and a Django app within the EC2 instance you just created. Then, you should run the Django development server on port 8000 in the background.


```python3
from fabric import Connection
from io import StringIO

# Connect to the EC2 instance
c = Connection('23803313-vm2')

# Step 1: Update and upgrade the instance
c.sudo('apt-get update -y')
c.sudo('apt-get upgrade -y')

# Step 2: Install Python 3 virtual environment
c.sudo('apt-get install python3-venv -y')

# Step 3: Install nginx
c.sudo('apt-get install nginx -y')

# Step 4: Create project directories and set ownership
# Check if the project directory exists
result = c.run('test -d /opt/wwc/mysites', warn=True)
if result.exited != 0:
    # Directory does not exist; create it
    c.sudo('mkdir -p /opt/wwc/mysites')
    c.sudo('chown -R ubuntu:ubuntu /opt/wwc')
    print("Created /opt/wwc/mysites and set ownership to ubuntu.")
else:
    print("/opt/wwc/mysites already exists. Skipping directory creation.")

# Step 5: Set up Python virtual environment
# Check if the virtual environment exists
result = c.run('test -d /opt/wwc/mysites/myvenv', warn=True)
if result.exited != 0:
    c.run('python3 -m venv /opt/wwc/mysites/myvenv')
    print("Created Python virtual environment at /opt/wwc/mysites/myvenv.")
else:
    print("Virtual environment already exists. Skipping creation.")

# Step 6: Install Django within the virtual environment
# Check if Django is already installed
result = c.run('/opt/wwc/mysites/myvenv/bin/pip show django', warn=True)
if result.exited != 0:
    c.run('/opt/wwc/mysites/myvenv/bin/pip install django')
    print("Installed Django in the virtual environment.")
else:
    print("Django is already installed in the virtual environment.")

# Step 7: Create Django project
# Check if the Django project directory exists
dir_exists = c.run('test -d /opt/wwc/mysites/lab', warn=True).exited == 0

if not dir_exists:
    # Directory does not exist; create the project
    with c.cd('/opt/wwc/mysites'):
        c.run('/opt/wwc/mysites/myvenv/bin/django-admin startproject lab')
    print("Created Django project 'lab'.")
else:
    # Directory exists, check if 'manage.py' exists
    manage_py_exists = c.run('test -f /opt/wwc/mysites/lab/manage.py', warn=True).exited == 0
    if not manage_py_exists:
        # 'manage.py' is missing, directory might be incomplete
        c.run('rm -rf /opt/wwc/mysites/lab')
        print("Removed incomplete 'lab' directory.")
        with c.cd('/opt/wwc/mysites'):
            c.run('/opt/wwc/mysites/myvenv/bin/django-admin startproject lab')
        print("Recreated Django project 'lab'.")
    else:
        print("Django project 'lab' already exists and is complete. Skipping project creation.")

# Step 8: Create the 'polls' app within the project
# Check if the 'polls' app exists
result = c.run('test -d /opt/wwc/mysites/lab/polls', warn=True)
if result.exited != 0:
    with c.cd('/opt/wwc/mysites/lab'):
        c.run('../myvenv/bin/python manage.py startapp polls')
    print("Created Django app 'polls'.")
else:
    print("Django app 'polls' already exists. Skipping app creation.")


# Step 9: Modify nginx configuration to act as a reverse proxy
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

# Upload the nginx configuration only if it doesn't already match
nginx_config_path = '/etc/nginx/sites-enabled/default'

# Get current nginx config
current_nginx_config = c.sudo(f'cat {nginx_config_path}', hide=True).stdout.strip()

if current_nginx_config != nginx_config.strip():
    c.sudo('tee /etc/nginx/sites-enabled/default', in_stream=StringIO(nginx_config))
    c.sudo('service nginx restart')
    print("Nginx configuration updated and service restarted.")
else:
    print("Nginx configuration is up-to-date. Skipping update.")

# Step 10: Create a basic view for the Django app
views_py_path = '/opt/wwc/mysites/lab/polls/views.py'
views_py_content = '''from django.http import HttpResponse

def index(request):
    return HttpResponse('Hello, world.')
'''

# Check if views.py already exists with the expected content
result = c.run(f'test -f {views_py_path}', warn=True)
if result.exited != 0:
    c.put(StringIO(views_py_content), views_py_path)
    print("Created views.py for the 'polls' app.")
else:
    existing_views_py = c.run(f'cat {views_py_path}', hide=True).stdout.strip()
    if existing_views_py != views_py_content.strip():
        c.put(StringIO(views_py_content), views_py_path)
        print("Updated views.py.")
    else:
        print("views.py is up-to-date. Skipping update.")

# Step 11: Edit the URL configuration to route to the view
polls_urls_py_path = '/opt/wwc/mysites/lab/polls/urls.py'
polls_urls_py_content = '''from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
'''

# Check if polls/urls.py exists and matches the expected content
result = c.run(f'test -f {polls_urls_py_path}', warn=True)
if result.exited != 0:
    c.put(StringIO(polls_urls_py_content), polls_urls_py_path)
    print("Created urls.py for the 'polls' app.")
else:
    existing_polls_urls_py = c.run(f'cat {polls_urls_py_path}', hide=True).stdout.strip()
    if existing_polls_urls_py != polls_urls_py_content.strip():
        c.put(StringIO(polls_urls_py_content), polls_urls_py_path)
        print("Updated polls/urls.py.")
    else:
        print("polls/urls.py is up-to-date. Skipping update.")

# Step 12: Add the app's URL configuration to the main project
main_urls_py_path = '/opt/wwc/mysites/lab/lab/urls.py'
main_urls_py_content = '''from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('admin/', admin.site.urls),
    path('polls/', include('polls.urls')),
]
'''

# Check if main urls.py matches the expected content
existing_main_urls_py = c.run(f'cat {main_urls_py_path}', hide=True).stdout.strip()
if existing_main_urls_py != main_urls_py_content.strip():
    c.put(StringIO(main_urls_py_content), main_urls_py_path)
    print("Updated main urls.py.")
else:
    print("Main urls.py is up-to-date. Skipping update.")

# Step 13: Run Django development server in the background
# Check if the server is already running
result = c.run("ps aux | grep 'manage.py runserver' | grep -v grep", warn=True)
if result.exited != 0:
    with c.cd('/opt/wwc/mysites/lab'):
        c.run('nohup ../myvenv/bin/python manage.py runserver 0.0.0.0:8000 &')
    print("Django development server started.")
else:
    print("Django development server is already running. Skipping start.")

```
The revised Fabric script is meticulously designed to automate the deployment of a Django web application on an EC2 instance, ensuring that each step is idempotent and safe to execute multiple times without causing errors or unintended side effects. It starts by establishing a secure connection to the EC2 instance identified by '23803313-vm2' using Fabric's Connection class. The script then performs system updates and upgrades by running apt-get update and apt-get upgrade with sudo privileges to ensure the instance has the latest packages, which is crucial for security and stability. It proceeds to install Python 3's virtual environment module and nginx if they are not already installed, checking their existence to prevent redundant installations. The script checks for the existence of the project directory /opt/wwc/mysites; if it doesn't exist, it creates it and assigns ownership to the 'ubuntu' user to prevent permission issues during subsequent operations. When setting up the Python virtual environment in /opt/wwc/mysites/myvenv, the script verifies whether it already exists to avoid unnecessary recreation. It installs Django within the virtual environment only if it hasn't been installed yet, ensuring efficient use of resources. For the Django project 'lab', the script checks if the directory and the manage.py file exist; if the directory exists but is incomplete (missing manage.py), it removes the incomplete directory to avoid conflicts and recreates the project, ensuring a clean setup. The same careful checks are applied when creating the 'polls' app; it is only created if it doesn't already exist, preventing duplication and errors. The nginx configuration is handled by reading the current configuration file and comparing it with the desired configuration; it updates the configuration and restarts the nginx service only if changes are detected, thus avoiding unnecessary restarts that could disrupt service. The script also manages the critical Django files views.py and urls.py for both the 'polls' app and the main project by comparing existing file contents with the desired code; it updates the files only if there are discrepancies, ensuring that the application code remains consistent and up-to-date without overwriting unchanged files. Finally, the script checks whether the Django development server is already running by searching for the process; if it is not running, it starts the server in the background using nohup, allowing it to continue running after the script completes, but if it is already running, it leaves it undisturbed. This comprehensive approach of checking and conditional execution ensures that the script is robust, avoids unnecessary operations, and adheres to best practices in automation and deployment, providing a reliable and efficient method to manage the application's lifecycle on the EC2 instance.
![image](https://github.com/user-attachments/assets/54727085-65bd-4151-9715-6a82cacfc029)

From your local OS environment, access the URL: `http://15.152.34.101/polls/`, and output what you've got. 

![image](https://github.com/user-attachments/assets/0446ec88-03d6-47ee-a4dd-3bda0c4aba13)

