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

# Lab 2

## Create an EC2 instance using awscli
### [1] Create a security group

```
aws ec2 create-security-group --group-name 23803313-sg --description "security group for development environment"
```
{
    "GroupId": "sg-05f6535931263b31f"
}

![image](https://github.com/user-attachments/assets/09a2b62f-df3c-47fa-8ea4-2f85e5ccc530)

### [2] Authorise inbound traffic for ssh

```
aws ec2 authorize-security-group-ingress --group-name 23803313-sg --protocol tcp --port 22 --cidr 0.0.0.0/0
```
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-01bfacc882ad92e77",
            "GroupId": "sg-05f6535931263b31f",
            "GroupOwnerId": "489389878001",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 22,
            "ToPort": 22,
            "CidrIpv4": "0.0.0.0/0"
        }
    ]
}

![image](https://github.com/user-attachments/assets/27d744f1-e297-4336-afbb-da10c11bb7e6)

### [3] Create a key pair

```
aws ec2 create-key-pair --key-name 23803313-key --query 'KeyMaterial' --output text > 23803313-key.pem
```

To use this key on Linux, copy the file to a directory ~/.ssh and change the permissions to:

```
chmod 400 23803313-key.pem
```

![image](https://github.com/user-attachments/assets/f203ae30-0fc6-4ea0-ac07-72b9b908a1bc)

### [4] Create the instance 

| Student Number | Region | Region Name | ami id |
| --- | --- | --- | --- |

| 23799000 – 23863700 | Asia Pacific (Osaka)	| ap-northeast-3	| ami-0a70c5266db4a6202 |



Based on your region code, find the corresponding ami id in the table above and fill it in the command below:

```
 aws ec2 run-instances --image-id ami-0a70c5266db4a6202 --security-group-ids 23803313-sg --count 1 --instance-type t2.micro --key-name 23803313-key --query 'Instances[0].InstanceId'

 ```
Instace created i-0dcfef96ec413ecca
![image](https://github.com/user-attachments/assets/3aec8350-8576-4ef9-b344-9f664f8fde70)

### [5] Add a tag to your Instance

 ```
  aws ec2 create-tags --resources i-0dcfef96ec413ecca --tags Key=Name,Value=23803313-vm1
 ```
![image](https://github.com/user-attachments/assets/50613443-6ef9-4d86-a60f-36324e391364)

**NOTE**: If you need to create a single instance, follow the naming format of `<student number>-vm` (e.g., 24242424-vm). If you need to create multiple ones, follow the naming format of `<student number>-vm1` and `<student number>-vm2` (e.g., 24242424-vm1, 24242424-vm2).

### [6] Get the public IP address

```
aws ec2 describe-instances --instance-ids i-0dcfef96ec413ecca --query 'Reservations[0].Instances[0].PublicIpAddress'
```
13.208.91.27
![image](https://github.com/user-attachments/assets/f3ffee53-faed-44a1-a36d-326a7f9d6c29)

### [7] Connect to the instance via ssh
```
ssh -i 23803313-key.pem ubuntu@13.208.91.27"
```
![image](https://github.com/user-attachments/assets/42851d5a-8d3b-4e82-a78e-f5dbe1b79c42)


### [8] List the created instance using the AWS console
![image](https://github.com/user-attachments/assets/2d83568f-3fc4-47e6-9789-eb175386806d)

## Create an EC2 instance with Python Boto3

Use a Python script to implement the steps above (steps 1-6 and 8 are required, step 7 is optional). Refer to [page](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/ec2.html) for details.
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

**NOTE**: If you are allocated to Europe (Stockholm), eu-north-1, the type of the instance in your script should be `t3.micro` rather than `t2.micro`. When you are done, log into the EC2 console and terminate the instances you created.
![image](https://github.com/user-attachments/assets/653b635c-203d-4166-af9c-633b7b47351a)
![image](https://github.com/user-attachments/assets/95724b9d-86ea-4326-897b-6f2e5805bf3b)

## Use Docker inside a Linux OS

### [1] Install Docker
```
sudo apt install docker.io -y
```
![image](https://github.com/user-attachments/assets/e0813438-36c5-40ab-bc77-8e4c7dcba9d0)


### [2] Start Docker
```
sudo systemctl start docker
```

### [3] Enable Docker
```
sudo systemctl enable docker
```

### [4] Check the version

```
docker --version
```
![image](https://github.com/user-attachments/assets/93a5c2f0-a8aa-468a-b638-b7f874f4a53f)

### [5] Build and run an httpd container

Create a directory called html

Edit a file index.html inside the html directory and add the following content

```
  <html>
    <head> </head>
    <body>
      <p>Hello World!</p>
    </body>
  </html>
```

Create a file called Dockerfile outside the html directory with the following content:

```
FROM httpd:2.4
COPY ./html/ /usr/local/apache2/htdocs/
```

Build a docker image

```
docker build -t my-apache2 .
```
![image](https://github.com/user-attachments/assets/331a755a-35f8-40be-bde6-6cf97188b517)

If you run into permission errors, you may need add your user to the docker group:

```
sudo usermod -a -G docker <username>
```

Be sure to log out and log back in for this change to take effect.

Run the image

```
docker run -p 80:80 -dit --name my-app my-apache2
```
![image](https://github.com/user-attachments/assets/47f797bc-af2b-41ae-bae0-539f87aef712)

Open a browser and access address: http://localhost or http://127.0.0.1. 

Confirm you get "Hello World!"
![image](https://github.com/user-attachments/assets/8549d00a-5cb6-4ff4-a51c-4b4114f3902e)

### [6] Other docker commands

To check what is running

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

