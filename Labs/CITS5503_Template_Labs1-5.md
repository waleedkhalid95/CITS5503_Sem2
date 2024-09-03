<div style="display: flex; flex-direction: column; justify-content: center; align-items: center; height: 100vh;">

  <h2>Labs 1-5</h2>
  
  <p>Student ID: 23803313</p>
  <p>Student Name: Waleed Khalid Siraj</p>

</div>

# Lab 1

## AWS Account and Log in

### [1] Log into an IAM user account created for you on AWS

First, I went to [AWS Console](https://489389878001.signin.aws.amazon.com/console) and logged in with my student email as username and the password provided.

### [2] Search and open Identity Access Management

1. Clicked on my profile on the top right.
2. Went to **Security Credentials**.
3. Scrolled down to find the **Access Keys** section and clicked on **Create access key**.
4. Selected **CLI**.
   ![Screenshot 2024-08-02 203545](https://github.com/user-attachments/assets/a67ed185-d7b2-4970-997a-699c7127e113)

6. Set the description tag.
7. Clicked on **Create access key**.
   - A confirmation screen popped up confirming that the access key is created.
     ![Screenshot 2024-08-02 203925](https://github.com/user-attachments/assets/765ca5d6-ddd1-416c-9348-e79a4750eeab)

8. Made a note of the ID and password or downloaded the CSV file containing the ID and password.

## Set up recent Linux OSes

1. **Download and install VMware for Windows.**
2. **Download Kali Linux for VMware and extract the 7z file.**
3. **Open VMware:**
   - Click on **File** on the top right and select **Open**.
   - Find the VMX file for Kali Linux in the extracted directory.
     ![Screenshot 2024-08-02 204513](https://github.com/user-attachments/assets/3fb96208-005a-461f-8940-8272ac592ff0)

4. **Edit Virtual Machine Settings:**
   - Set memory to 8GB, 4 processor cores, 30GB hard disk, and NAT network.
5. **Power on the virtual machine.**
6. **Log into Kali Linux with the default ID and password.**

## Install Linux Packages

### [1] Install Python 3.8.x

1. Open terminal and update packages:
   - "sudo apt update"
     - **`sudo`**: Executes the command as a superuser.
     - **`apt`**: Advanced Package Tool, used for handling package management in Debian-based distributions.
     - **`update`**: Fetches the package lists from the repositories and updates them to get information on the newest versions of packages and their dependencies.
   - "sudo apt -y upgrade"
     - **`-y`**: Assumes "yes" as the answer to all prompts and runs non-interactively.
    ![image](https://github.com/user-attachments/assets/d27e790a-a68e-4c5e-9dfb-e74cbc5b3165)

2. Check Python version and install pip:
   - "python3.8 --version"
     - **`python3.8`**: Specifies the Python 3.8 version.
     - **`--version`**: Displays the version of the installed Python.
   - "sudo apt install python3-pip"
     - **`install`**: Installs the specified package.
     - **`python3-pip`**: Pip is the package installer for Python.
       ![Screenshot 2024-08-02 215157](https://github.com/user-attachments/assets/bc9ac7be-8b8f-46d1-ad1b-c75edbce2f6a)


### [2] Install awscli

1. Install AWS CLI:
   - "sudo apt install awscli"
     - **`awscli`**: AWS Command Line Interface, a unified tool to manage AWS services.
2. Upgrade AWS CLI:
   - "pip3 install awscli --upgrade"
     - **`pip3`**: Pip for Python 3, used to install Python packages.
     - **`install`**: Installs the specified package.
     - **`awscli`**: AWS Command Line Interface package.
     - **`--upgrade`**: Upgrades the package to the latest version.
       ![image](https://github.com/user-attachments/assets/2a36e5ba-13ec-4b83-a50d-ad4a38bf6058)


### [3] Configure AWS

1. Configure AWS CLI:
   - "aws configure"
     - **`configure`**: Starts the AWS configuration process.
   - Enter Access Key ID: "AKIAXD4PI5LY6VWGSI4M"
   - Enter Secret Access Key.
   - Default region name: "ap-northeast-3" (as per student ID range).
   - Default output format: "json".
     ![image](https://github.com/user-attachments/assets/2fac505e-644f-49f8-ae4f-e6616dc18837)


### [4] Install boto3

1. Install boto3:
   - "pip3 install boto3"
     - **`boto3`**: The Amazon Web Services (AWS) SDK for Python, which allows Python developers to write software that makes use of Amazon services like S3 and EC2.

## Test the Installed Environment

### [1] Test the AWS environment

1. Test the AWS environment by listing regions:
   - "aws ec2 describe-regions --output table"
     - **`aws ec2`**: Command to interact with the EC2 service.
     - **`describe-regions`**: Describes the regions that are available to you.
     - **`--output table`**: Specifies that the output should be formatted as a table.
   - The output is a table.
     ![image](https://github.com/user-attachments/assets/5871561f-d577-4389-942c-025cc694079e)


### [2] Test the Python environment

1. Test if Python works by extracting the same table in JSON format:
   - "import boto3"
     - **`import boto3`**: Imports the boto3 library.
   - "ec2 = boto3.client('ec2')"
     - **`boto3.client('ec2')`**: Creates a low-level client representing Amazon EC2.
   - "response = ec2.describe_regions()"
     - **`ec2.describe_regions()`**: Calls the describe_regions method to get a list of regions.
   - "print(response)"
     - **`print(response)`**: Prints the response.
       ![image](https://github.com/user-attachments/assets/9c8fa783-89fe-4e3e-a721-8f2cf731033a)


### [3] Write a Python script

1. **Create a folder on the Desktop named `cloud-lab`.**
2. **Create an empty file and name it `lab1.py`.**
3. **Open the file and add the following Python script, then save:**
      - ![image](https://github.com/user-attachments/assets/be11cd02-2b57-4e64-8f82-b2b65c9a1e4f)

     - **`import boto3`**: Imports the boto3 library.
     - **`import pandas as pd`**: Imports the pandas library and aliases it as pd.
     - **`from tabulate import tabulate`**: Imports the tabulate function from the tabulate module.
     - **`boto3.client('ec2')`**: Creates a low-level client representing Amazon EC2.
     - **`response = ec2.describe_regions()`**: Calls the describe_regions method to get a list of regions.
     - **`regions = response['Regions']`**: Extracts the 'Regions' data from the response.
     - **`pd.DataFrame(regions, columns=['Endpoint', 'RegionName'])`**: Converts the data into a pandas DataFrame.
     - **`print(tabulate(df, headers='keys', tablefmt='psql'))`**: Prints the DataFrame in a table format using tabulate.
4. **Navigate to the folder using the terminal:**
   - "cd /home/kali/Desktop/cloud-lab/"
     - **`cd`**: Change directory command.
     - **`/home/kali/Desktop/cloud-lab/`**: Path to the cloud-lab folder.
5. **Make the file executable:**
   - "chmod +x lab1.py"
     - **`chmod +x`**: Changes the file mode to make it executable.
     - **`lab1.py`**: The file to be made executable.
6. **Execute the Python script:**
   - "python3 lab1.py"
     - **`python3`**: Specifies the Python 3 interpreter.
     - **`lab1.py`**: The Python script to be executed.
    ![image](https://github.com/user-attachments/assets/d14a0ce4-bb70-4c8e-bba7-68a0ca759304)


<div style="page-break-after: always;"></div>

# Lab 2: Creating an EC2 Instance with AWS CLI and Boto3
### Summary
In this lab, we created an EC2 instance on AWS using both the AWS CLI and Python's Boto3 SDK. The objective was to automate the setup of a secure and accessible virtual machine for development purposes. Key tasks included setting up security rules, generating secure access keys, launching the instance, and configuring Docker to run a simple web server. Each step ensures that the environment is secure, accessible, and functional for cloud-based development and testing.

## EC2 Instance Setup Using AWS CLI

### [1] Create a Security Group
We create a security group using:
```
aws ec2 create-security-group --group-name 23803313-sg --description "security group for development environment"
```
- '--group-name': Specifies the name of the security group
- '--description': Describes the purpose of the security group. This security group acts as a virtual firewall to control inbound and outbound traffic for our EC2 instances. The output provides the security group ID, which we need for subsequent steps.

![image](https://github.com/user-attachments/assets/09a2b62f-df3c-47fa-8ea4-2f85e5ccc530)
Security groups act as virtual firewalls that control traffic to instances. This is essential for defining which types of connections are allowed.

### [2] Authorize Inbound SSH Traffic
Next, we authorize SSH access to the EC2 instance by modifying the security group with:

```
aws ec2 authorize-security-group-ingress --group-name 23803313-sg --protocol tcp --port 22 --cidr 0.0.0.0/0
```
- '--protocol tcp --port 22': Specifies TCP as the protocol and opens port 22 for SSH access.
- '--cidr 0.0.0.0/0': Allows access from any IP address. This configuration allows secure remote SSH connections to the EC2 instance.

![image](https://github.com/user-attachments/assets/27d744f1-e297-4336-afbb-da10c11bb7e6)

### [3] Create a Key Pair and Set Permissions
To securely connect to our EC2 instance, we create a key pair using:

```
aws ec2 create-key-pair --key-name 23803313-key --query 'KeyMaterial' --output text > 23803313-key.pem
```

- '--key-name': Specifies the name of the key pair.
- '--query 'KeyMaterial' --output text > 23803313-key.pem': Extracts the key material and saves it to a .pem file. The private key is essential for SSH access, and we secure the key with:

```
chmod 400 23803313-key.pem
```
This restricts the file permissions to read-only for the owner, enhancing security.

![image](https://github.com/user-attachments/assets/f203ae30-0fc6-4ea0-ac07-72b9b908a1bc)

### [4] Launch the EC2 Instance
Using the AMI ID corresponding to our region, we launch an instance with:

```
 aws ec2 run-instances --image-id ami-0a70c5266db4a6202 --security-group-ids 23803313-sg --count 1 --instance-type t2.micro --key-name 23803313-key --query 'Instances[0].InstanceId'

 ```
Instace created i-0dcfef96ec413ecca
![image](https://github.com/user-attachments/assets/3aec8350-8576-4ef9-b344-9f664f8fde70)

- '--image-id': Specifies the AMI ID for the Osaka region (ami-0a70c5266db4a6202).
- '--security-group-ids': Associates the instance with the previously created security group.
- '--instance-type t2.micro': Uses a cost-effective instance type suitable for development environments. The command outputs the instance ID upon successful creation.

### [5] Tag the Instance
To identify the instance easily, we add a tag:
 ```
  aws ec2 create-tags --resources i-0dcfef96ec413ecca --tags Key=Name,Value=23803313-vm1
 ```
![image](https://github.com/user-attachments/assets/50613443-6ef9-4d86-a60f-36324e391364)
- '--resources': Specifies the instance ID.
- '--tags Key=Name,Value=23803313-vm1': Adds a descriptive tag to the instance. This helps in managing and identifying instances in the AWS console.

### [6] Retrieve the Public IP Address
To connect to our instance, we need its public IP address, obtained via:

```
aws ec2 describe-instances --instance-ids i-0dcfef96ec413ecca --query 'Reservations[0].Instances[0].PublicIpAddress'
```
This command queries the instance details and extracts the public IP, 13.208.91.27.

![image](https://github.com/user-attachments/assets/f3ffee53-faed-44a1-a36d-326a7f9d6c29)

### [7] Connect to the Instance via SSH
Finally, we connect to the instance using SSH:
```
ssh -i 23803313-key.pem ubuntu@13.208.91.27"
```
![image](https://github.com/user-attachments/assets/42851d5a-8d3b-4e82-a78e-f5dbe1b79c42)
The SSH command uses the private key and the instance’s public IP address to establish a secure shell session.

### [8] List the Instance in AWS Console
After completing the above steps, the instance can be managed via the AWS console.
![image](https://github.com/user-attachments/assets/2d83568f-3fc4-47e6-9789-eb175386806d)

## EC2 Instance Setup Using Python Boto3
Using Python's Boto3, we automated the same steps programmatically, allowing for greater flexibility and integration into larger automation workflows. The Python script covered creating security groups, setting up key pairs, launching instances, and configuring access, similar to the AWS CLI approach but in a more scriptable format.

```
import boto3
import os
import subprocess
import time

# Initialize the EC2 client
ec2 = boto3.client('ec2')

# Step 1: Create a security group
security_group = ec2.create_security_group(
    Description='security group for development environment',
    GroupName='23803313-sg-boto3',
)
print(f"Security Group Created: {security_group['GroupId']}")

# Step 2: Authorize inbound traffic for SSH
ec2.authorize_security_group_ingress(
    GroupName='23803313-sg-boto3',
    IpPermissions=[
        {
            'IpProtocol': 'tcp',
            'FromPort': 22,
            'ToPort': 22,
            'IpRanges': [{'CidrIp': '0.0.0.0/0'}]
        }
    ]
)
print(f"Inbound SSH traffic authorized for {security_group['GroupId']}")

# Step 3: Create a key pair
key_pair_name = '23803313-boto3-key'
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
time.sleep(240)

# Step 7: Connect to the instance via SSH
ssh_command = f"ssh -i {key_file_path} ubuntu@{public_ip}"
print(f'Connecting to the instance via SSH: {ssh_command}')
try:
    subprocess.run(ssh_command, shell=True, check=True)
except subprocess.CalledProcessError as e:
    print(f"Failed to connect to the instance: {e}")
```
![image](https://github.com/user-attachments/assets/653b635c-203d-4166-af9c-633b7b47351a)
![image](https://github.com/user-attachments/assets/95724b9d-86ea-4326-897b-6f2e5805bf3b)

## Use Docker Inside a Linux OS
Docker allows for containerized applications, making it easier to deploy and manage applications consistently.
To demonstrate Docker, we installed it on the EC2 instance and ran a simple HTTP server
### [1] Install Docker
```
sudo apt install docker.io -y
```
![image](https://github.com/user-attachments/assets/e0813438-36c5-40ab-bc77-8e4c7dcba9d0)
This command installs Docker on the instance, enabling container management.

### [2] Start and Enable Docker
```
sudo systemctl start docker
sudo systemctl enable docker
docker --version
```
Starting Docker ensures the service runs immediately, and enabling it makes Docker start automatically on boot. Then we verify the Docker installation by checking its version.

![image](https://github.com/user-attachments/assets/93a5c2f0-a8aa-468a-b638-b7f874f4a53f)

### [4] Build and Run an HTTPD Container

Create a directory called html, then create an index.html file inside with:

```
  <html>
    <head> </head>
    <body>
      <p>Hello World!</p>
    </body>
  </html>
```
Create a Dockerfile outside the html directory:
```
FROM httpd:2.4
COPY ./html/ /usr/local/apache2/htdocs/
```
The server was then accessed via the instance’s IP, demonstrating how Docker simplifies the deployment of applications on cloud instances.

Build a docker image

```
docker build -t my-apache2 .
```
![image](https://github.com/user-attachments/assets/331a755a-35f8-40be-bde6-6cf97188b517)

Run the container

```
docker run -p 80:80 -dit --name my-app my-apache2
```
![image](https://github.com/user-attachments/assets/47f797bc-af2b-41ae-bae0-539f87aef712)

Visit http://localhost to confirm the "Hello World!" message displays.

![image](https://github.com/user-attachments/assets/8549d00a-5cb6-4ff4-a51c-4b4114f3902e)

### [5] Other docker commands

To check running containers

```
docker ps -a
```
To stop and remove the container

```
docker stop my-app
docker rm my-app
```
![image](https://github.com/user-attachments/assets/a9d48537-6705-465b-8cbe-f3f54ea79a98)


<div style="page-break-after: always;"></div>

# Lab 3

<div style="page-break-after: always;"></div>

# Lab 4

<div style="page-break-after: always;"></div>

# Lab 5

