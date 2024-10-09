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
