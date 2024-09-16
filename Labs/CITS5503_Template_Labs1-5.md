<div style="display: flex; flex-direction: column; justify-content: center; align-items: center; height: 100vh;">

  <h2>Labs 1-5</h2>
  
  <p>Student ID: 23803313</p>
  <p>Student Name: Waleed Khalid Siraj</p>


</div>

# Lab 1: AWS Setup and Environment Configuration

In this lab, I set up an AWS environment by configuring IAM access, installing necessary packages on a Linux OS, and verifying the setup with various tests. The goal was to ensure that my environment is fully prepared for interacting with AWS services via the command line and Python scripts.

## AWS Account and Log in

### [1] Log into an IAM User Account on AWS

First, I logged into my IAM user account by navigating to the [AWS Console](https://489389878001.signin.aws.amazon.com/console). Using my student email as the username and the provided password, I accessed the AWS Management Console, which serves as the primary interface for managing AWS services.

### [2] Search and Open Identity Access Management (IAM)

To configure access to AWS services, I followed these steps:

1. Clicked on my profile at the top right corner of the AWS Console.
2. Navigated to **Security Credentials**.
3. Scrolled down to the **Access Keys** section and clicked on **Create access key** to generate new access credentials.
4. Selected the **CLI** option, which configures the access key for use with command-line interfaces, essential for managing AWS resources programmatically.

   ![Access Key Best Practices](https://github.com/user-attachments/assets/a67ed185-d7b2-4970-997a-699c7127e113)

5. Set a description tag to help identify the purpose of this access key.
6. Clicked **Create access key**, which generated a confirmation screen showing the new access key ID and secret access key.

   ![Access Key Creation Confirmation](https://github.com/user-attachments/assets/765ca5d6-ddd1-416c-9348-e79a4750eeab)
   
7. I saved the access key and secret key securely, as they are crucial for authenticating CLI commands to AWS services.

## Setting Up a Linux OS

To establish a working environment compatible with AWS tools, I set up a virtual machine with the following steps:

1. **Downloaded and installed VMware for Windows** to run a virtual environment.
2. **Downloaded Kali Linux for VMware** and extracted the downloaded 7z file, which contains the necessary files to boot Kali Linux on VMware.
3. **Opened VMware**:
   - Clicked on **File** in the top menu and selected **Open**.
   - Located and opened the VMX file for Kali Linux from the extracted directory.

   ![Opening Kali Linux VMX File](https://github.com/user-attachments/assets/3fb96208-005a-461f-8940-8272ac592ff0)

4. **Edited Virtual Machine Settings**:
   - Adjusted the settings to allocate 8GB of memory, 4 processor cores, a 30GB hard disk, and set up a NAT network for internet connectivity.
5. **Powered on the Virtual Machine** and logged into Kali Linux using the default credentials provided.

This setup allowed me to create a dedicated Linux environment to work with AWS services and related tools effectively.

## Installing Linux Packages

### [1] Install Python 3.8.x

To ensure compatibility with the latest tools and libraries, I installed Python 3.8.x:

1. Opened the terminal and ran the following commands:
   - `"sudo apt update"`: This command updates the package lists to fetch the latest information about available packages and their dependencies.
   - `"sudo apt -y upgrade"`: This upgrades the installed packages to their latest versions, ensuring that the system is up-to-date.

   ![Updating and Upgrading Packages](https://github.com/user-attachments/assets/d27e790a-a68e-4c5e-9dfb-e74cbc5b3165)

2. Checked the installed Python version and installed pip (Python’s package installer):
   - `"python3.8 --version"`: Verified the Python version to ensure Python 3.8.x is installed.
   - `"sudo apt install python3-pip"`: Installed pip for Python 3, which is necessary for managing Python packages.

   ![Checking Python Version and Installing Pip](https://github.com/user-attachments/assets/bc9ac7be-8b8f-46d1-ad1b-c75edbce2f6a)

### [2] Install AWS CLI

To interact with AWS services from the command line, I installed the AWS CLI:

1. Ran `"sudo apt install awscli"` to install AWS CLI version 1, which provides a unified command line interface to manage AWS services.
2. Upgraded AWS CLI to the latest version using `"pip3 install awscli --upgrade"`, ensuring access to the latest features and improvements.

   ![Installing and Upgrading AWS CLI](https://github.com/user-attachments/assets/2a36e5ba-13ec-4b83-a50d-ad4a38bf6058)

### [3] Configure AWS CLI

Configured the AWS CLI to use my IAM credentials and region:

1. Ran `"aws configure"` to start the configuration process.
2. Entered the access key ID and secret access key that I had saved earlier.
3. Set the default region to `"ap-northeast-3"` based on my student ID range, which aligns with my geographic location and reduces latency.
4. Set the default output format to `"json"` to ensure data is returned in a readable format for automation scripts.

   ![Configuring AWS CLI](https://github.com/user-attachments/assets/2fac505e-644f-49f8-ae4f-e6616dc18837)

### [4] Install Boto3

Boto3 is the AWS SDK for Python, enabling Python developers to write software that makes use of Amazon services like S3 and EC2:

1. Installed Boto3 using `"pip3 install boto3"`, which allows me to manage AWS services directly from Python scripts.

## Testing the Installed Environment

### [1] Test the AWS Environment

To verify that AWS CLI was correctly configured, I tested it by listing available EC2 regions:

1. Ran `"aws ec2 describe-regions --output table"`, which lists all regions where EC2 services are available, formatted as a table for easy readability.

   ![Testing AWS Environment with EC2 Regions](https://github.com/user-attachments/assets/5871561f-d577-4389-942c-025cc694079e)

### [2] Test the Python Environment

To ensure the Python environment was set up correctly and could interact with AWS services, I wrote a short script to list EC2 regions:

1. Imported Boto3 and created an EC2 client:
   - `"import boto3"`: Imports the Boto3 library for AWS interaction.
   - `"ec2 = boto3.client('ec2')"`: Creates an EC2 client object for interacting with the EC2 service.
   
2. Retrieved the list of regions and printed it:
   - `"response = ec2.describe_regions()"`: Calls the `describe_regions` method on the EC2 client to fetch available regions.
   - `"print(response)"`: Outputs the response, confirming that Python can successfully interact with AWS services.

   ![Testing Python Environment](https://github.com/user-attachments/assets/9c8fa783-89fe-4e3e-a721-8f2cf731033a)


### [3] Write a Python Script

To solidify my environment setup, I created a Python script to display EC2 regions in a formatted table:

1. **Created a folder on the Desktop named `cloud-lab`.**
2. **Created an empty file named `lab1.py` and added the following Python script:**

   ```python
   import boto3
   import pandas as pd
   from tabulate import tabulate

   ec2 = boto3.client('ec2')
   response = ec2.describe_regions()
   regions = response['Regions']

   df = pd.DataFrame(regions, columns=['Endpoint', 'RegionName'])
   print(tabulate(df, headers='keys', tablefmt='psql'))
   ```
   - **`import boto3`**: Imports the boto3 library.
   - **`import pandas as pd`**: Imports the pandas library and aliases it as pd.
   - **`from tabulate import tabulate`**: Imports the tabulate function from the tabulate module.
   - **`boto3.client('ec2')`**: Creates an EC2 client to interact with the EC2 service.
   - **`response = ec2.describe_regions()`**: Calls the describe_regions method to get a list of regions.
   - **`regions = response['Regions']`**: Extracts the 'Regions' data from the response.
   - **`pd.DataFrame(regions, columns=['Endpoint', 'RegionName'])`**: Converts the data into a pandas DataFrame.
   - **`print(tabulate(df, headers='keys', tablefmt='psql'))`**: Prints the DataFrame in a table format using tabulate.
3. **Navigated to the folder using the terminal:**
   - Ran "cd /home/kali/Desktop/cloud-lab/" to navigate to the directory where the script is saved.
     - **`cd`**: Change directory command.
     - **`/home/kali/Desktop/cloud-lab/`**: Path to the cloud-lab folder.
4. **Made the script executable:**
   - Ran "chmod +x lab1.py" to change the file mode, making it executable.
     - **`chmod +x`**: Changes the file mode to make it executable.
     - **`lab1.py`**: The file to be made executable.
5. **Executed the Python script:**
   - Ran "python3 lab1.py" to execute the script and display the EC2 regions in a formatted table.
     - **`python3`**: Specifies the Python 3 interpreter.
     - **`lab1.py`**: The Python script to be executed.
       
    ![EC2 Regions](https://github.com/user-attachments/assets/d14a0ce4-bb70-4c8e-bba7-68a0ca759304)

<div style="page-break-after: always;"></div>


# Lab 2: Creating an EC2 Instance with AWS CLI and Boto3

## Summary

In this lab, I created an EC2 instance on AWS using both the AWS CLI and Python's Boto3 SDK. The objective was to automate the setup of a secure and accessible virtual machine for development purposes. The key tasks involved setting up security rules, generating secure access keys, launching the instance, and configuring Docker to run a simple web server. Each step ensured that the environment was secure, accessible, and functional for cloud-based development and testing.

## EC2 Instance Setup Using AWS CLI

### [1] Create a Security Group

To start, I created a security group, which acts as a virtual firewall controlling inbound and outbound traffic to the EC2 instance. This was done using the command:

```bash
aws ec2 create-security-group --group-name 23803313-sg --description "security group for development environment"
```
- `create-security-group` : This command is used to create a new security group within your specified AWS account. Security groups are essential in AWS as they define the allowed inbound and outbound traffic rules for instances.
- `--group-name` : This option specifies the name of the security group, making it easy to identify and manage within the AWS console. In this case, I named it `23803313-sg` to associate it with my specific environment.
- `--description` : Describes the purpose of the security group. This security group acts as a virtual firewall to control inbound and outbound traffic for our EC2 instances. The output provides the security group ID, which we need for subsequent steps.

The output of this command provides a security group ID, which is necessary for further configuration steps.

![image](https://github.com/user-attachments/assets/09a2b62f-df3c-47fa-8ea4-2f85e5ccc530)

Security groups are critical in AWS as they allow you to define which types of traffic can reach your EC2 instances. This step generated a security group ID, which I needed for the subsequent steps.

### [2] Authorize Inbound SSH Traffic
After creating the security group, I configured it to allow SSH access, which is required for remotely managing and configuring the EC2 instance. SSH (Secure Shell) is a protocol used to securely connect to Linux instances over the internet.

```bash
aws ec2 authorize-security-group-ingress --group-name 23803313-sg --protocol tcp --port 22 --cidr 0.0.0.0/0
```
- `authorize-security-group-ingress`: modifies the security group to allow specific inbound traffic rules.
- `--protocol tcp`: Specifies that the rule applies to TCP traffic, which is the protocol used for SSH connections.
- `--port 22`: Indicates the port number to open; port 22 is the standard port for SSH
- `--cidr 0.0.0.0/0`: This option allows access from any IP address, making the instance accessible globally. While this setting is convenient for testing and development, it poses a security risk for production environments and should be restricted to known IP addresses.

Configuring SSH access ensures that I can remotely access and manage the instance securely using the private key.

![image](https://github.com/user-attachments/assets/27d744f1-e297-4336-afbb-da10c11bb7e6)

### [3] Create a Key Pair and Set Permissions
To enable secure access to the EC2 instance, I needed a key pair. The key pair consists of a public key that AWS stores and a private key that I store. The private key is used to securely SSH into the EC2 instance

```bash
aws ec2 create-key-pair --key-name 23803313-key --query 'KeyMaterial' --output text > 23803313-key.pem
```

- `create-key-pair`: This command creates a new key pair with AWS, which is essential for securing SSH access to EC2 instances
- `--key-name`: Specifies the name of the key pair, making it identifiable. I named it `23803313-key` to align with my specific configuration
- `--query 'KeyMaterial' --output text > 23803313-key.pem`: Extracts the private key material and saves it to a `.pem` file named `23803313-key.pem`. This private key file is necessary for SSH access to the instance.


After generating the key pair, it’s crucial to secure the key by modifying its file permissions:

```bash
chmod 400 23803313-key.pem
```
- `chmod 400` : Sets the file permissions to read-only for the owner, which is a security best practice. This ensures that the private key is not accessible by others, safeguarding SSH access to the instance.

This step is crucial because it prevents unauthorized access to the key, ensuring that only I can use it to connect to the instance.

![image](https://github.com/user-attachments/assets/f203ae30-0fc6-4ea0-ac07-72b9b908a1bc)

### [4] Launch the EC2 Instance
Using the AMI ID for the Osaka region, I launched the EC2 instance:

```bash
 aws ec2 run-instances --image-id ami-0a70c5266db4a6202 --security-group-ids 23803313-sg --count 1 --instance-type t2.micro --key-name 23803313-key --query 'Instances[0].InstanceId'

 ```
Instace created `i-0dcfef96ec413ecca` 

![image](https://github.com/user-attachments/assets/3aec8350-8576-4ef9-b344-9f664f8fde70)

- `run-instances` : This command launches new EC2 instances based on the specified parameters.
- `--image-id ami-0a70c5266db4a6202`: Specifies the Amazon Machine Image (AMI) ID, which serves as the template for the instance, including the operating system and application software configurations.
- `--security-group-ids 23803313-sg`: Associates the instance with the security group created earlier, applying the inbound and outbound traffic rules defined for that group.
- `--count 1`: Indicates that only one instance should be launched.
- `--instance-type t2.micro` : Specifies the instance type, which determines the hardware configuration. The `t2.micro` instance type is cost-effective and suitable for low-traffic applications and development environments.
- `--key-name 23803313-key` : Associates the instance with the key pair created earlier, enabling SSH access using the private key.
  
This command launched the instance and returned an instance ID, confirming the successful creation.

### [5] Tag the Instance
To make it easier to identify and manage the instance, I added a descriptive tag:

 ```bash
  aws ec2 create-tags --resources i-0dcfef96ec413ecca --tags Key=Name,Value=23803313-vm1
 ```

![image](https://github.com/user-attachments/assets/50613443-6ef9-4d86-a60f-36324e391364)

- `create-tags`: This command is used to add metadata to AWS resources, making them easier to identify and manage.
- `--resources i-0dcfef96ec413ecca` : Specifies the instance ID to be tagged.
- `--tags Key=Name,Value=23803313-vm1`: Adds a tag with a key-value pair to the instance. This helps in organizing and managing instances, especially when dealing with multiple resources in the AWS console.

Tags are helpful for organizing resources, especially when managing multiple instances.

### [6] Retrieve the Public IP Address
To connect to the instance, I needed its public IP address, which was obtained with the following command:

```bash
aws ec2 describe-instances --instance-ids i-0dcfef96ec413ecca --query 'Reservations[0].Instances[0].PublicIpAddress'
```
This command queries the instance details and extracts the public IP, 13.208.91.27.

- `describe-instances`: This command retrieves detailed information about the specified EC2 instance.
- `--query 'Reservations[0].Instances[0].PublicIpAddress'`: Extracts the public IP address from the instance details, which is required for establishing a remote SSH connection.

![image](https://github.com/user-attachments/assets/f3ffee53-faed-44a1-a36d-326a7f9d6c29)

### [7] Connect to the Instance via SSH
With the public IP address and key pair, I connected to the instance using SSH:

```bash
ssh -i 23803313-key.pem ubuntu@13.208.91.27"
```
- `ssh -i 23803313-key.pem` : Specifies the private key file for authentication.
- `ubuntu@13.208.91.27` : Connects to the instance as the `ubuntu` user using the public IP address retrieved earlier.

![image](https://github.com/user-attachments/assets/42851d5a-8d3b-4e82-a78e-f5dbe1b79c42)

This command establishes a secure connection to the instance using the private key and public IP address, enabling remote management and interaction.

### [8] List the Instance in AWS Console
After completing these steps, the instance was visible and manageable through the AWS Console, where I could monitor its status, manage tags, adjust security settings, and perform other administrative tasks.

![image](https://github.com/user-attachments/assets/2d83568f-3fc4-47e6-9789-eb175386806d)

## EC2 Instance Setup Using Python Boto3
To further automate the setup process, I used Python's Boto3 SDK, which provides a programmatic way to interact with AWS services. Below is the Python script I used:

```python
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

# Change the file permission to chmod 400 to secure the key
os.chmod(key_file_path, 0o400)
print(f'Key pair created, saved to {key_file_path}, and permissions set to 400')

# Step 4: Create the instance
instance = ec2.run_instances(
    ImageId="ami-0a70c5266db4a6202", # AMI ID for the selected region
    SecurityGroupIds=[security_group['GroupId']],  # AMI ID for the selected region
    InstanceType='t2.micro', # Instance type suitable for development and testing
    KeyName=key_pair_name, # Key pair for SSH access
    MinCount=1, # Minimum number of instances to launch
    MaxCount=1 # Maximum number of instances to launch
)

instance_id = instance['Instances'][0]['InstanceId'] # Retrieve the instance ID
print(f'EC2 Instance Created: {instance_id}')

# Step 5: Add a tag to the instance for easier identification
ec2.create_tags(
    Resources=[instance_id],
    Tags=[{'Key': 'Name', 'Value': '23803313-vm2'}] # Assign a descriptive name
)
print(f'Tag added to instance {instance_id}')



# Step 6: Get the public IP address of the instance
response = ec2.describe_instances(InstanceIds=[instance_id])
public_ip = response['Reservations'][0]['Instances'][0]['PublicIpAddress']
print(f'Public IP Address of the instance: {public_ip}')

print('Waiting for the instance to initialize...')
time.sleep(240) # Wait time for the instance to fully initialize

# Step 7: Connect to the instance via SSH
ssh_command = f"ssh -i {key_file_path} ubuntu@{public_ip}"
print(f'Connecting to the instance via SSH: {ssh_command}')
try:
    subprocess.run(ssh_command, shell=True, check=True) # Attempt to connect using SSH
except subprocess.CalledProcessError as e:
    print(f"Failed to connect to the instance: {e}")
```

**Code Explanation:**

 - **Initialize EC2 Client**: `boto3.client('ec2')` initializes the EC2 client, providing a connection to AWS EC2 service through Boto3, the AWS SDK for Python.
 - **Create Security Group**: The script creates a security group using `ec2.create_security_group()`, including a description and group name. This security group acts as a virtual firewall that controls the traffic allowed to reach the EC2 instance.
 - **Authorize SSH Access**: SSH access is enabled using `ec2.authorize_security_group_ingress()` with TCP protocol on port 22, allowing connections from all IP addresses (`0.0.0.0/0`). For production environments, it is advisable to restrict this to specific IP addresses for better security.
 - **Create Key Pair**: A key pair is generated using `ec2.create_key_pair()`, specifying the AMI ID, security group, instance type, and key name. It outputs the instance ID upon successful creation, indicating that the instance is ready for use.
 - **Launch EC2 Instance**: The instance is launched with `ec2.run_instances()`, specifying the AMI ID, security group, instance type, and key name. It outputs the instance ID upon successful creation, indicating that the instance is ready for use.
 - **Tag Instance**: The instance is tagged using `ec2.create_tags()` to make it easily identifiable in the AWS console. Tags are used for organizing and managing resources within AWS, aiding in tracking and automation.
 - **Retrieve Public IP**: The instance's public IP address is obtained with `ec2.describe_instances()`, which is necessary for connecting via SSH and managing the instance remotely.
 - **Connect via SSH**: The script attempts to connect to the instance using SSH, automating the login process and enabling direct management of the instance from the terminal. This step provides a secure way to interact with the instance for any configuration or application setup tasks.

![image](https://github.com/user-attachments/assets/653b635c-203d-4166-af9c-633b7b47351a)

![image](https://github.com/user-attachments/assets/95724b9d-86ea-4326-897b-6f2e5805bf3b)

## Use Docker Inside a Linux OS
Docker allows for containerized applications, simplifying the deployment and management of applications in a consistent environment. In this lab, I demonstrated Docker's utility by installing it on the EC2 instance and running a simple HTTP server.
### [1] Install Docker
I installed Docker using the following command
```bash
sudo apt install docker.io -y
```

![image](https://github.com/user-attachments/assets/e0813438-36c5-40ab-bc77-8e4c7dcba9d0)

This command installs Docker on the instance, enabling container management, which is essential for deploying applications in isolated environments.

### [2] Start and Enable Docker
I started Docker and enabled it to run on boot with the following commands
```bash
sudo systemctl start docker
sudo systemctl enable docker
docker --version
```
- `sudo systemctl start docker`: Starts the Docker service immediately, allowing you to begin using Docker commands.
- `sudo systemctl enable docker`: Ensures that Docker will start automatically on system boot, maintaining consistency across restarts.
- `docker --version`: Verifies the Docker installation by displaying the installed version.

Starting Docker ensures the service runs immediately, and enabling it makes Docker start automatically on boot. Then we verify the Docker installation by checking its version.

![image](https://github.com/user-attachments/assets/93a5c2f0-a8aa-468a-b638-b7f874f4a53f)

### [4] Build and Run an HTTPD Container

To demonstrate Docker’s utility, I built and ran a simple HTTP server container:
1. Created a directory called `html` and added a file `index.html` with the content

```
  <html>
    <head> </head>
    <body>
      <p>Hello World!</p>
    </body>
  </html>
```
2. Created a `Dockerfile` outside the `html` directory with:
```bash
FROM httpd:2.4
COPY ./html/ /usr/local/apache2/htdocs/
```
- The `Dockerfile` uses the official HTTPD (Apache) image and copies the contents of the `html` directory into the container's web root. This setup is straightforward but demonstrates how Docker simplifies deploying web applications by encapsulating all dependencies within a container.
3. Build the docker image

```bash
docker build -t my-apache2 .
```

- `docker build -t my-apache2 .`: Builds the Docker image from the current directory (`.`) using the Dockerfile, tagging the image as `my-apache2`.

![image](https://github.com/user-attachments/assets/331a755a-35f8-40be-bde6-6cf97188b517)

4. Run the container

```bash
docker run -p 80:80 -dit --name my-app my-apache2
```
- `docker run -p 80:80 -dit --name my-app my-apache2`: Runs the container, mapping port 80 on the instance to port 80 in the container, and detaches the terminal (`-d`) while running in interactive mode (`-it`). The `--name` flag assigns a name to the running container, making it easier to manage.

This command runs the container, mapping port 80 on the instance to port 80 in the container, allowing me to access the server via the instance's IP address

![image](https://github.com/user-attachments/assets/47f797bc-af2b-41ae-bae0-539f87aef712)

5. Visited `http://localhost` or the instance’s public IP to confirm the "Hello World!" message displays, verifying that the HTTP server is running correctly inside the Docker container.

![image](https://github.com/user-attachments/assets/8549d00a-5cb6-4ff4-a51c-4b4114f3902e)

### [5] Other docker commands
To manage Docker containers, I used the following commands
- To check running containers
  ```bash
  docker ps -a
  ```
  - Lists all Docker containers, including those that are stopped, providing a full view of container statuses and allowing for management actions such as starting or stopping.
 - To stop and remove the container
   ```bash
   docker stop my-app
   docker rm my-app
   ```
   These commands allow for managing Docker containers, stopping them when they are no longer needed, and cleaning up resources

![image](https://github.com/user-attachments/assets/a9d48537-6705-465b-8cbe-f3f54ea79a98)

<div style="page-break-after: always;;"></div>


# Lab 3: Cloud Storage with S3 and DynamoDB

## Summary

In this lab, I set up a personal cloud storage application using AWS services. The main objectives were to create and configure S3 buckets, work with DynamoDB for storing file metadata, and restore files from the cloud back to a local environment. By the end of this lab, I successfully scanned a directory, uploaded files to an S3 bucket, stored metadata in DynamoDB, and restored the files to a local directory, achieving a robust understanding of AWS cloud storage and database services.

## Program Steps

### [1] Preparation

To begin, I prepared the environment as follows:

1. **Downloaded the Python code** `cloudstorage.py` from the [src](https://github.com/zhangzhics/CITS5503_Sem2/blob/master/Labs/src/cloudstorage.py) directory. This script serves as the base for interacting with AWS services using Boto3.
2. **Created a directory** named `rootdir` on my local machine, which will be used to simulate the source of the files to be uploaded to the S3 bucket.
3. **Inside `rootdir`, I created a file** named `rootfile.txt` and added some content: `1\n2\n3\n4\n5\n`. This file represents a typical file that might be stored in cloud storage.
4. **Created a subdirectory** within `rootdir` named `subdir`, and added another file named `subfile.txt`, containing the same content as `rootfile.txt`. This nested structure allowed me to test the ability to maintain directory structures when uploading to S3.

This setup of `rootdir` and its subdirectory `subdir` created a nested file structure that would be replicated in the S3 bucket, demonstrating S3's capability to maintain folder hierarchies.

### [2] Save to S3 by Updating `cloudstorage.py`

Next, I modified the `cloudstorage.py` script to create an S3 bucket and upload the files from `rootdir` while preserving their directory structure. Here’s the modified script:

```python
import os
import boto3
import base64

# Define the local root directory and the S3 bucket name
ROOT_DIR = './'  # The current working directory
ROOT_S3_DIR = '23803313-cloudstorage'  # Name of the S3 bucket

# Initialize an S3 client
s3 = boto3.client("s3")

# Specify the bucket configuration, including the region
bucket_config = {'LocationConstraint': 'ap-northeast-3'}  # Allocated region name

def upload_file(folder_name, file, file_name):
    """
    Uploads a file to the specified S3 bucket, preserving the directory structure.
    
    :param folder_name: The folder path within the S3 bucket
    :param file: The full local file path
    :param file_name: The name of the file to be uploaded
    """
    # Upload the file to S3, preserving the folder structure within the bucket
    s3.upload_file(file, ROOT_S3_DIR, f"{folder_name}/{file_name}")
    print(f"Uploading {file}")

# Attempt to create the S3 bucket
try:
    # Create the S3 bucket with the specified configuration
    response = s3.create_bucket(Bucket=ROOT_S3_DIR, CreateBucketConfiguration=bucket_config)
    print(f"Bucket {ROOT_S3_DIR} created: {response}")
except Exception as error:
    # Handle any errors during bucket creation, such as if the bucket already exists
    print(f"Bucket creation failed: {error}")
    pass

# Walk through the ROOT_DIR, recursively traversing all subdirectories and files
for dir_name, subdir_list, file_list in os.walk(ROOT_DIR, topdown=True):
    # Skip the root directory itself to avoid uploading it
    if dir_name != ROOT_DIR:  
        for fname in file_list:
            # Upload each file, preserving its directory structure in the S3 bucket
            upload_file("%s/" % dir_name[2:], "%s/%s" % (dir_name, fname), fname)

print("done")
```
**Code Explanation:**
- **Initialization** :
  - `boto3.client("s3")`: Initializes an S3 client, which provides a low-level interface to interact with AWS S3 services, enabling operations like creating buckets and uploading files.
- **Bucket Creation** :
  - The script attempts to create an S3 bucket named `23803313-cloudstorage` in the specified region (`ap-northeast-3`). The bucket configuration is specified using the `CreateBucketConfiguration` parameter, which includes the `LocationConstraint` to set the region.
  - The `try` block is used to handle exceptions that may occur during bucket creation, such as if the bucket already exists or if there are permission issues
- **File Upload Function** :
  - `upload_file(folder_name, file, file_name)`: This function uploads files to the S3 bucket while preserving the folder structure from the local directory. It constructs the S3 path using the folder name and file name, ensuring that the nested directory structure is maintained in the bucket.
  - `s3.upload_file()`: The method used to upload files to S3. It takes the local file path, bucket name, and S3 target path as arguments.
- **Directory Traversal** :
  - The script uses `os.walk()` to recursively traverse `ROOT_DIR`, listing all subdirectories and files.
  - For each file, it calls the `upload_file` function to upload it to the S3 bucket in the correct folder, replicating the local directory structure.

After running the script, I verified that the files and their directory structure from rootdir were correctly replicated in the S3 bucket:

![image](https://github.com/user-attachments/assets/77e64c70-11a5-4f27-b6f0-212278b5b2b8)

I confirmed the bucket and file creation through the AWS console, ensuring that the bucket contained the correct files in the expected directory structure.

![image](https://github.com/user-attachments/assets/154c624d-8d9b-4162-be8a-f6c199eab45a)


### [3] Restore from S3

I then created a new Python script named `restorefromcloud.py` to restore the files and directories from the S3 bucket back to a local environment. Here’s the script:

```python
import boto3
import os

BUCKET_NAME = '23803313-cloudstorage'  # Name of the S3 bucket to restore from
s3 = boto3.resource('s3')  # Initialize an S3 resource to interact with the bucket

try:
    # List all objects in the specified S3 bucket
    response = s3.meta.client.list_objects_v2(Bucket=BUCKET_NAME)
    
    # Check if 'Contents' key exists in the response to ensure files are present
    if 'Contents' not in response:
        print(f"No files found in bucket {BUCKET_NAME}.")
    else:
        for obj in response['Contents']:
            s3_key = obj['Key']  # Get the S3 object key (file path in the bucket)
            print(f"Restoring {s3_key} from S3...")

            # Define the local path where the file will be saved
            local_path = os.path.join('./', s3_key)  # Save in the current directory
            local_dir = os.path.dirname(local_path)  # Extract the directory part of the path
            
            # Create local directories if they do not exist
            if not os.path.exists(local_dir):
                os.makedirs(local_dir)
            
            # Download the file from S3 to the local path
            s3.meta.client.download_file(BUCKET_NAME, s3_key, local_path)
            print(f"Downloaded {s3_key} to {local_path}")
            
    print("Restoration complete.")

except botocore.exceptions.ClientError as error:
    print(f"An error occurred: {error}")
```

**Code Explanation**
- **Initialize S3 Resource**:
  - `boto3.resource('s3')`: Initializes an S3 resource, providing a higher-level interface for interacting with S3, such as managing objects and performing actions like download.
- **List and Restore Objects**:
  - `list_objects_v2(Bucket=BUCKET_NAME)`: Lists all objects in the specified S3 bucket. The response includes each object’s key, which indicates its path within the bucket.
  - The script checks for the `'Contents'` key in the response to ensure there are files to restore.
- **Restoring Files**:
  - For each object in the bucket, the script constructs the local path using `os.path.join('./', s3_key)`, preserving the directory structure as it downloads.
  - It creates necessary local directories with `os.makedirs(local_dir)` if they don’t already exist.
  - Files are downloaded using `s3.meta.client.download_file(BUCKET_NAME, s3_key, local_path)`, saving them to their respective paths on the local machine.

Upon running this script, the `Restored` directory on my local machine was populated with the files and structure from the S3 bucket, successfully replicating the original setup.

![image](https://github.com/user-attachments/assets/0a3f7e58-b258-4432-bc35-dfe6906fb90a)


### [4] Write Information About Files to DynamoDB

To further extend the cloud storage functionality, I stored metadata about the files in DynamoDB, allowing for efficient file management and retrieval. First, I set up DynamoDB locally:
```
mkdir dynamodb
cd dynamodb
sudo apt-get install default-jre
wget https://s3-ap-northeast-1.amazonaws.com/dynamodb-local-tokyo/dynamodb_local_latest.tar.gz
tar -zxvf dynamodb_local_latest.tar.gz
java -Djava.library.path=./DynamoDBLocal_lib -jar DynamoDBLocal.jar –sharedDb
```

This setup provides a local version of DynamoDB for development and testing purposes, simulating the AWS environment without incurring costs.

I then wrote a python script to create a table called `CloudFiles` on the local DynamoDB.

```python
import boto3
import os
from datetime import datetime

BUCKET_NAME = '23803313-cloudstorage'  
REGION_NAME = 'ap-northeast-3' 

# Initialize AWS resources: S3 client and DynamoDB resource
s3 = boto3.client('s3')
dynamodb = boto3.resource('dynamodb', region_name=REGION_NAME)

# Define the DynamoDB table name
table_name = 'CloudFiles'

# Check if the table exists, if not, create it
existing_tables = dynamodb.meta.client.list_tables()['TableNames']
if table_name not in existing_tables:
    # Create a new DynamoDB table with userId as partition key and fileName as sort key
    table = dynamodb.create_table(
        TableName=table_name,
        KeySchema=[
            {'AttributeName': 'userId', 'KeyType': 'HASH'},  # Partition key
            {'AttributeName': 'fileName', 'KeyType': 'RANGE'}  # Sort key
        ],
        AttributeDefinitions=[
            {'AttributeName': 'userId', 'AttributeType': 'S'},  # String type
            {'AttributeName': 'fileName', 'AttributeType': 'S'}  # String type
        ],
        ProvisionedThroughput={
            'ReadCapacityUnits': 6,
            'WriteCapacityUnits': 6
        }
    )
    
    # Wait until the table exists
    table.meta.client.get_waiter('table_exists').wait(TableName=table_name)
    print(f"Table {table_name} created successfully.")
else:
    table = dynamodb.Table(table_name)
    print(f"Table {table_name} already exists.")

# Retrieve the list of objects in the S3 bucket
response = s3.list_objects_v2(Bucket=BUCKET_NAME)

# Check for files in the S3 bucket
if 'Contents' not in response:
    print(f"No files found in bucket {BUCKET_NAME}.")
else:
    for obj in response['Contents']:
        s3_key = obj['Key']
        print(f"Processing {s3_key} from S3...")

        # Fetch file attributes
        head_response = s3.head_object(Bucket=BUCKET_NAME, Key=s3_key)
        acl_response = s3.get_object_acl(Bucket=BUCKET_NAME, Key=s3_key)

        # Extract owner information based on region
        owner_info = acl_response['Owner']
        owner = owner_info['DisplayName'] if REGION_NAME in ['us-east-1', 'ap-northeast-1', 'ap-southeast-1', 'ap-southeast-2'] else owner_info['ID']

        # Extract permissions
        permissions = [grant['Permission'] for grant in acl_response['Grants'] if 'Permission' in grant]

        # Define item attributes to be stored in DynamoDB
        item = {
            'userId': '23803313',
            'fileName': os.path.basename(s3_key),
            'path': os.path.dirname(s3_key),
            'lastUpdated': head_response['LastModified'].strftime('%Y-%m-%d %H:%M:%S'),
            'owner': owner,
            'permissions': ', '.join(permissions)  # Converting list to string
        }

        # Insert the item into DynamoDB table
        try:
            table.put_item(Item=item)
            print(f"Inserted {s3_key} into DynamoDB.")
        except Exception as e:
            print(f"Failed to insert {s3_key} into DynamoDB: {e}")

print("Process complete.")
```
**Code Explanation**:
- **DynamoDB Resource Initialization**:
  - `boto3.resource('dynamodb', region_name=REGION_NAME)` initializes a DynamoDB resource that points to the specified region (`ap-northeast-3`), allowing the script to interact with DynamoDB.
- **Table Creation**:
  - The script first checks if the table `CloudFiles` exists using `list_tables()`.
  - If the table does not exist, it is created using `dynamodb.create_table()`, with `userId` as the partition key and `fileName` as the sort key, both of type string.
  - Provisioned throughput is set to manage read and write capacity.
- **Metadata Extraction**:
  - The script retrieves the list of objects in the S3 bucket using `list_objects_v2()` and fetches metadata (e.g., last modified date) using `head_object()`.
  - It also retrieves the access control list of each object with `get_object_acl()` to determine ownership and permissions.
- **Data Insertion into DynamoDB**:
  - Metadata for each file is structured into an item dictionary and inserted into the `CloudFiles` table using `put_item()`.
  - This process enables efficient storage and retrieval of file metadata, facilitating management of the cloud storage.

![image](https://github.com/user-attachments/assets/d87a04bc-d51b-42b8-879c-295635aaad25)


### [5] Scan the table

I verified the contents of the `CloudFiles` table using AWS CLI:

```bash
aws dynamodb scan --table-name CloudFiles --region ap-northeast-3
```
This command scans the table and retrieves all stored items, allowing me to validate that the metadata was correctly inserted.

![image](https://github.com/user-attachments/assets/147950f0-1b8f-4a2b-8c4e-364f8d089650)

### [6] Delete the table

After completing the tasks, I deleted the table using the AWS CLI:

```bash
aws dynamodb delete-table --table-name CloudFiles --region ap-northeast-3
```
This command deletes the specified DynamoDB table, cleaning up resources after the lab.

![image](https://github.com/user-attachments/assets/6c2f929e-8271-4a1c-a1e5-e3efa27dd285)

Finally, I removed the S3 bucket from the AWS console to complete the cleanup process.


<div style="page-break-after: always;"></div>


# Lab 4: IAM Policies, KMS, and AES Encryption

## Summary

In this lab, I aimed to enhance the security and encryption of my AWS resources, focusing on managing access to S3 buckets and using encryption keys effectively. First, I applied a policy to restrict access to my S3 bucket, ensuring that only my specific user account could access it. Following that, I created a KMS key using my student number as an alias, and attached a policy that strictly controlled who could use and manage the key. I then used this key to encrypt and decrypt files in the S3 bucket, verifying that the permissions were correctly set and the encryption worked as intended. Finally, I implemented local encryption using the PyCryptodome library to explore an alternative to AWS KMS, comparing the performance and use cases of both methods.

## Applying a Policy to Restrict Permissions on S3 Bucket

### [1] Writing a Python Script to Apply S3 Bucket Policy

To start, I needed to ensure that my S3 bucket was secure by restricting access to only my user account. I achieved this by writing a Python script that applied a policy to the bucket. This policy specifically allowed actions only when the access request matched my username.

```python
import boto3
import json

# Initialize the S3 client
s3 = boto3.client('s3')

# Define the bucket name and the policy
bucket_name = '23803313-cloudstorage'
policy = {
    "Version": "2012-10-17",
    "Statement": [{
        "Sid": "AllowAllS3ActionsInUserFolderForUserOnly",
        "Effect": "Deny",  # Deny access by default
        "Principal": "*",  # Applies to all principals (users)
        "Action": "s3:*",  # All S3 actions
        "Resource": f"arn:aws:s3:::{bucket_name}/*",  # Apply to all objects in the bucket
        "Condition": {
            "StringNotLike": {
                "aws:username": "23803313@student.uwa.edu.au"  # Allow only the specific user
            }
        }
    }]
}

# Convert the policy to a JSON string
policy_json = json.dumps(policy)

# Apply the policy to the S3 bucket
try:
    s3.put_bucket_policy(Bucket=bucket_name, Policy=policy_json)
    print(f"Policy applied to bucket {bucket_name} successfully.")
except Exception as e:
    print(f"Failed to apply policy: {e}")
```
**Code Explanation**:
- **Initialization**:
  - `boto3.client('s3')`: Initializes an S3 client, allowing interaction with AWS S3 services programmatically.
- **Policy Definition**:
  - The policy is designed to deny all actions (`s3:*`) by default unless the request originates from the specified username (`23803313@student.uwa.edu.au`). This restrictive approach ensures that only authorized actions by the intended user are allowed.
- **Applying the Policy**:
  - The policy is converted into a JSON string using `json.dumps()`, which is then applied to the S3 bucket using the `put_bucket_policy` method.
  - The `try-except` block is used to handle any errors that may occur during policy application, such as permission issues or incorrect policy syntax.
 
**Policy Explanation**:
- **Policy Structure**:
  - **Version**: Specifies the policy language version. The date "2012-10-17" is the latest and most commonly used version.
  - **Sid**: A statement identifier that helps to distinguish the statement.
  - **Effect**: Set to "Deny", meaning the default action is to deny access unless specified conditions are met.
  - **Principal**: Set to "*", meaning the policy applies to all users.
  - **Action**: Specifies "s3:*", which means the policy applies to all S3 actions like `GetObject`, `PutObject`, etc.
  - **Resource**: Targets all objects within the specified bucket.
  - Condition: Uses `StringNotLike` to allow access only if the `aws:username` matches the specified username (`23803313@student.uwa.edu.au`).
- **Purpose**:
  - The policy enforces that only the specified user can perform actions on the bucket, effectively creating a whitelist. Any access attempts from other users are denied by default.

This approach was crucial to securing my data, as it explicitly restricted access to unauthorized users, thereby protecting the contents of my S3 bucket.

![image](https://github.com/user-attachments/assets/78fb279e-c24d-448a-bd14-fe188da2482f)

### [2] Verifying the S3 Bucket Policy

After applying the policy, I needed to confirm that it was correctly set and functioning as intended. I used the AWS CLI to retrieve and display the policy content from the S3 bucket:

```bash
aws s3api get-bucket-policy --bucket 23803313-cloudstorage
```
This command retrieves the current policy applied to the specified S3 bucket, allowing me to verify its correctness.

![image](https://github.com/user-attachments/assets/aca1f3b9-6783-4d3d-a891-2dc265ceb2e9)

I then tested the policy by accessing the bucket using my correct username. Since the policy was configured to allow my account, I was able to successfully list the bucket contents:

```bash
aws s3 ls s3://23803313-cloudstorage/rootdir/
```

![image](https://github.com/user-attachments/assets/e5dd8734-4b4a-4719-94f3-ebe95915cc2e)

To further validate the policy's effectiveness, I tried accessing the bucket with an incorrect username. As expected, the access was denied, proving that the policy was working as designed to restrict unauthorized access:

![image](https://github.com/user-attachments/assets/a7b53aae-db90-4b5d-b32d-dd6d8bf4bc1a)


## AES Encryption Using KMS

### [1] Creating a KMS Key and Attaching a Policy

Next, I moved on to setting up encryption using AWS KMS. I started by creating a KMS key with my student number as an alias. This key would be used to encrypt and decrypt files, providing an extra layer of security for the data stored in my S3 bucket.
```python3
import boto3
import json

# Initialize the KMS client
kms = boto3.client('kms')

# Define alias and student number
student_number = '23803313'
alias_name = f'alias/{student_number}'

# Create the KMS key with a description
try:
    response = kms.create_key(Description='KMS key for encryption tasks')
    key_id = response['KeyMetadata']['KeyId']
    print(f'KMS Key Created: {key_id}')
    
    # Create an alias for the KMS key for easy reference
    kms.create_alias(AliasName=alias_name, TargetKeyId=key_id)
    print(f'Alias {alias_name} created for KMS Key {key_id}')
except Exception as e:
    print(f'An error occurred: {e}')

# Define the username for the policy
username = '23803313@student.uwa.edu.au'

# Define the key policy with permissions for the specified user
key_policy = {
    "Version": "2012-10-17",
    "Id": "key-consolepolicy-3",
    "Statement": [
        {
            "Sid": "Enable IAM User Permissions",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::489389878001:root"  # Root account permissions
            },
            "Action": "kms:*",  # Full access to KMS
            "Resource": "*"
        },
        {
            "Sid": "Allow access for Key Administrators",
            "Effect": "Allow",
            "Principal": {
                "AWS": f"arn:aws:iam::489389878001:user/{username}"  # Specific user permissions
            },
            "Action": [
                "kms:Create*",
                "kms:Describe*",
                "kms:Enable*",
                "kms:List*",
                "kms:Put*",
                "kms:Update*",
                "kms:Revoke*",
                "kms:Disable*",
                "kms:Get*",
                "kms:Delete*",
                "kms:TagResource",
                "kms:UntagResource",
                "kms:ScheduleKeyDeletion",
                "kms:CancelKeyDeletion"
            ],
            "Resource": "*"
        },
        {
            "Sid": "Allow use of the key",
            "Effect": "Allow",
            "Principal": {
                "AWS": f"arn:aws:iam::489389878001:user/{username}"  # Specific permissions for encryption and decryption
            },
            "Action": [
                "kms:Encrypt",
                "kms:Decrypt",
                "kms:ReEncrypt*",
                "kms:GenerateDataKey*",
                "kms:DescribeKey"
            ],
            "Resource": "*"
        },
        {
            "Sid": "Allow attachment of persistent resources",
            "Effect": "Allow",
            "Principal": {
                "AWS": f"arn:aws:iam::489389878001:user/{username}"  # Permissions for creating and managing grants
            },
            "Action": [
                "kms:CreateGrant",
                "kms:ListGrants",
                "kms:RevokeGrant"
            ],
            "Resource": "*",
            "Condition": {
                "Bool": {
                    "kms:GrantIsForAWSResource": "true"
                }
            }
        }
    ]
}

# Attach the policy to the KMS key
try:
    key_policy_json = json.dumps(key_policy)
    kms.put_key_policy(
        KeyId=key_id,
        PolicyName='default',
        Policy=key_policy_json
    )
    print(f"Policy successfully attached to KMS Key {key_id}")
except Exception as e:
    print(f'An error occurred: {e}')

```
**Code Explanation**:
- Creating the KMS Key:
  - I initialized the KMS client and created a new key with a description specifying its use for encryption tasks. The key ID, which uniquely identifies the key, was stored for later use.
- Alias Creation:
  - An alias was created for the key (`alias/23803313`), simplifying the reference to the key in subsequent operations.
- Defining and Attaching the Policy:
  - A comprehensive policy was defined, granting the IAM user specified (`23803313@student.uwa.edu.au`) full permissions to manage and use the key. The policy restricted access to only this user, enforcing strict control over the key's usage.
  - The policy was attached to the key using the `put_key_policy` method.
 
**Policy Explanation**:
- **Policy Structure**:
  - **Version**: Defines the policy language version, which is "2012-10-17" for this policy.
  - **Id**: Identifies the policy.
  - **Statement**: Contains multiple statements specifying different permissions:
    - **Statement 1 ("Enable IAM User Permissions")**:
      - Effect: "Allow".
      - Principal: AWS account root user, which means the policy applies to all IAM users under this account.
      - Action: "kms:*" - grants permission to perform all KMS actions.
      - Resource: "*" - applies to all KMS keys within the account.
    - **Statement 2 ("Allow access for Key Administrators")**:
      - Allows the specified IAM user (`23803313@student.uwa.edu.au`) full administrative permissions over the key (e.g., create, describe, enable, disable).
    - **Statement 3 ("Allow use of the key")**:
      - Allows the specified IAM user to use the key for encryption and decryption tasks.
    - **Statement 4 ("Allow attachment of persistent resources")**:
      - Allows the specified IAM user to create grants for the key but restricts this action to AWS services by using a condition `kms:GrantIsForAWSResource`.
- **Purpose**:
  - This policy is designed to strictly control access and use of the KMS key. By defining different permissions for administrators and users, it ensures that only authorized actions can be performed by specific users, aligning with the principle of least privilege.

This setup was crucial for managing access to the encryption key, enforcing strict control over who could use and manage the key, and maintaining data security.

![image](https://github.com/user-attachments/assets/ba9a2e51-d557-4f33-85c4-68659ea575fa)

### [2] Testing KMS Key Usage and Permissions

To ensure that the KMS key and policy were set up correctly, I used the AWS KMS console to verify that my username had the correct permissions as both a key administrator and key user. This verification was essential to confirm that the policy effectively restricted access, adhering to the principle of least privilege.

![image](https://github.com/user-attachments/assets/5d480602-0076-4f53-87f6-ffd95f6e21ae)


### [3] Using the Created KMS Key for Encryption/Decryption
Next, I utilized the KMS key to encrypt and decrypt files stored in my S3 bucket. This process was important for understanding how to secure data using AWS-managed encryption keys.
```python3
import boto3
import os

# Initialize AWS clients for S3 and KMS
s3 = boto3.client('s3')
kms = boto3.client('kms')

# Define the bucket name and KMS key alias
bucket_name = '23803313-cloudstorage'
kms_key_alias = 'alias/23803313'

# Retrieve the list of objects in the S3 bucket
objects = s3.list_objects_v2(Bucket=bucket_name)

# Check if the 'Contents' key exists in the response, indicating files are present
if 'Contents' not in objects:
    print(f"No files found in bucket {bucket_name}.")
else:
    # Iterate through each file in the bucket
    for obj in objects['Contents']:
        s3_key = obj['Key']
        local_path = os.path.basename(s3_key)  # Set local path for downloaded file

        # Download the file from S3
        s3.download_file(bucket_name, s3_key, local_path)
        print(f"Downloaded {s3_key} to {local_path}")

        # Encrypt the file using KMS
        with open(local_path, 'rb') as file:
            plaintext = file.read()  # Read the file contents
            encrypt_response = kms.encrypt(
                KeyId=kms_key_alias,  # Use the alias to reference the key
                Plaintext=plaintext
            )
            ciphertext = encrypt_response['CiphertextBlob']  # Get the encrypted data

        # Save the encrypted file locally
        encrypted_path = f"{local_path}.encrypted"
        with open(encrypted_path, 'wb') as enc_file:
            enc_file.write(ciphertext)
        print(f"Encrypted file saved as {encrypted_path}")

        # Decrypt the file using KMS
        decrypt_response = kms.decrypt(
            CiphertextBlob=ciphertext  # Use the encrypted data
        )
        decrypted_text = decrypt_response['Plaintext']  # Get the decrypted data

        # Save the decrypted file locally
        decrypted_path = f"{local_path}.decrypted"
        with open(decrypted_path, 'wb') as dec_file:
            dec_file.write(decrypted_text)
        print(f"Decrypted file saved as {decrypted_path}")

```

**Code Explanation**:
- **Downloading and Encrypting Files**:
  - The script begins by listing the objects in the specified S3 bucket and downloading each file to the local system.
  - For each file, it reads the content and encrypts it using the KMS key alias (`alias/23803313`). The encrypted data (`CiphertextBlob`) is saved to a new file with the `.encrypted` extension.
- **Decrypting Files**:
  - The encrypted files are then decrypted using the KMS key, and the decrypted content is saved to a new file with the `.decrypted` extension.
- **Security Validation**:
  - This process validates that the KMS key was correctly configured and functional for securing data both in transit and at rest, ensuring that sensitive information remains protected.


![image](https://github.com/user-attachments/assets/34e8ff52-27df-44f3-83d6-a2cfae83cf47)

![image](https://github.com/user-attachments/assets/ab9587f5-12ae-47d7-853b-b8f555ae113d)


## [5] Applying  `pycryptodome` for encryption/decryption
### [1] Installing PyCryptodome
To explore an alternative to AWS KMS, I implemented AES encryption and decryption using the PyCryptodome library. This approach provided insights into client-side encryption methods, which can be more performant but lack the integrated key management features of KMS.

```bash
pip install pycryptodome
```
This command installs the PyCryptodome library, a powerful suite for encryption in Python.

![image](https://github.com/user-attachments/assets/70d145c5-8096-4b2b-b24b-381128dd5ec7)

### [2] AES Encryption and Decryption Using PyCryptodome

Using the PyCryptodome library, I developed the following script to encrypt and decrypt files using AES. This script uses a predefined password to generate a key for the encryption process.

```python3
import os
import boto3
import struct
from Crypto.Cipher import AES
from Crypto import Random
import hashlib

# Initialize AWS S3 client
s3 = boto3.client('s3')

# Define bucket name
bucket_name = '23803313-cloudstorage'

# AES encryption/decryption parameters
BLOCK_SIZE = 16
CHUNK_SIZE = 64 * 1024
password = 'kitty and the kat'  # Use a secure password

def encrypt_file(password, in_filename, out_filename):
    # Generate key from the password using SHA-256
    key = hashlib.sha256(password.encode("utf-8")).digest()
    iv = Random.new().read(AES.block_size)  # Generate a random initialization vector for AES
    encryptor = AES.new(key, AES.MODE_CBC, iv)  # Create AES encryptor object
    filesize = os.path.getsize(in_filename)  # Get the size of the input file

    with open(in_filename, 'rb') as infile:
        with open(out_filename, 'wb') as outfile:
            outfile.write(struct.pack('<Q', filesize))  # Write the original file size for use in decryption
            outfile.write(iv)  # Write the initialization vector

            while True:
                chunk = infile.read(CHUNK_SIZE)  # Read the file in chunks
                if len(chunk) == 0:
                    break
                elif len(chunk) % 16 != 0:
                    # Pad the last chunk to ensure it is a multiple of BLOCK_SIZE
                    chunk += ' '.encode("utf-8") * (16 - len(chunk) % 16)

                outfile.write(encryptor.encrypt(chunk))  # Encrypt the chunk and write to the output file

def decrypt_file(password, in_filename, out_filename):
    # Generate key from the password using SHA-256
    key = hashlib.sha256(password.encode("utf-8")).digest()

    with open(in_filename, 'rb') as infile:
        origsize = struct.unpack('<Q', infile.read(struct.calcsize('Q')))[0]  # Read the original file size
        iv = infile.read(16)  # Read the initialization vector
        decryptor = AES.new(key, AES.MODE_CBC, iv)  # Create AES decryptor object

        with open(out_filename, 'wb') as outfile:
            while True:
                chunk = infile.read(CHUNK_SIZE)  # Read the encrypted file in chunks
                if len(chunk) == 0:
                    break
                outfile.write(decryptor.decrypt(chunk))  # Decrypt and write the chunk

            outfile.truncate(origsize)  # Remove padding by truncating to the original file size

# Fetch files from S3 bucket
response = s3.list_objects_v2(Bucket=bucket_name)

if 'Contents' not in response:
    print(f"No files found in bucket {bucket_name}.")
else:
    for obj in response['Contents']:
        s3_key = obj['Key']
        local_path = os.path.basename(s3_key)
        # Download the file from S3
        s3.download_file(bucket_name, s3_key, local_path)
        print(f"Downloaded {s3_key} to {local_path}")

        # Encrypt the file
        encrypted_path = f"{local_path}.encrypted"
        encrypt_file(password, local_path, encrypted_path)
        print(f"Encrypted file saved as {encrypted_path}")

        # Decrypt the file
        decrypted_path = f"{local_path}.decrypted"
        decrypt_file(password, encrypted_path, decrypted_path)
        print(f"Decrypted file saved as {decrypted_path}")
```
**Code Explanation**:
- **AES Encryption and Decryption Functions**:
  - `encrypt_file()`: This function encrypts the input file using AES in CBC mode. It generates a key from the provided password using SHA-256, creates an initialization vector (IV), and encrypts the file in chunks, adding padding if necessary to ensure chunk sizes are multiples of the block size.
  - `decrypt_file()`: This function decrypts the file encrypted by `encrypt_file()`. It reads the original file size and IV from the encrypted file, then decrypts each chunk, removing any padding at the end to restore the original file.
- **Fetching and Processing Files**:
  - The script lists files in the specified S3 bucket, downloads each file, encrypts it, and then decrypts it using the defined AES functions.
- **Security Considerations**:
  - This method demonstrates how client-side encryption can be performed outside of AWS services, offering flexibility and control over the encryption process.

![image](https://github.com/user-attachments/assets/b8798627-2006-476a-aa0c-f710a746f787)

By examining the local directory, I confirmed that all encrypted and decrypted files were correctly processed, verifying the success of the encryption workflow.

![image](https://github.com/user-attachments/assets/8dc4886d-bbb0-40f0-b51a-db9f2b4081f0)


## Answer the following question (Marked)

```
What is the performance difference between using KMS and using the custom solution?
```


<div style="page-break-after: always;"></div>


# Lab 5: Application Load Balancer with EC2 Instances

## Summary

In this lab, I configured an Application Load Balancer (ALB) to distribute HTTP requests between two EC2 instances located in different availability zones within the `ap-northeast-3` region. Using Python and Boto3, I automated the creation of EC2 instances, set up a security group allowing HTTP and SSH access, and configured a load balancer with a target group containing the instances. After installing Apache2 on both instances and customizing the default web pages to display the instance names, I verified the load balancer's functionality by accessing its DNS name, which successfully routed traffic between the instances. This demonstrated effective load balancing and improved application availability.

## Application Load Balancer

### [1] Creating EC2 Instances and Configuring the Load Balancer

The goal of this task was to set up a scalable and resilient web application environment by distributing incoming traffic across multiple EC2 instances using an Application Load Balancer. The steps involved included creating EC2 instances, configuring a security group, setting up an Application Load Balancer, creating a target group, and registering the instances in the target group.

Here’s the complete Python script:

```python
import boto3  # Boto3 is the AWS SDK for Python, which allows Python developers to write software that makes use of Amazon services
import time   # The time module is used to add delays in the script for waiting on resource provisioning

# Initialize EC2 and ELBv2 clients in the specified region
ec2 = boto3.client('ec2', region_name='ap-northeast-3')
elbv2 = boto3.client('elbv2', region_name='ap-northeast-3')

# Define parameters for EC2 instances and security group
ami_id = 'ami-0a70c5266db4a6202'  # AMI ID for a standard Linux instance in the ap-northeast-3 region
instance_type = 't3.micro'        # Instance type chosen for a balance of cost and performance
key_name = '23803313-key'         # Name of the SSH key pair for accessing the instances
security_group_name = '23803313-sg'  # Security group name to manage access rules

# Step 1: Check if the Security Group exists or create it if it doesn't
try:
    # Attempt to describe the security group by name to see if it already exists
    response = ec2.describe_security_groups(GroupNames=[security_group_name])
    security_group_id = response['SecurityGroups'][0]['GroupId']
    print(f"Existing Security Group found: {security_group_id}\n")
except ec2.exceptions.ClientError as e:
    # If the security group is not found, an exception is raised; check if it's due to a missing group
    if 'InvalidGroup.NotFound' in str(e):
        # Create a new security group for managing access to the instances
        try:
            security_group = ec2.create_security_group(
                Description='Security group for web and SSH access',
                GroupName=security_group_name
            )
            security_group_id = security_group['GroupId']
            print(f"Security Group Created: {security_group_id}\n")

            # Authorize inbound traffic rules for SSH (port 22) and HTTP (port 80)
            ec2.authorize_security_group_ingress(
                GroupId=security_group_id,
                IpPermissions=[
                    {
                        'IpProtocol': 'tcp',
                        'FromPort': 22,
                        'ToPort': 22,
                        'IpRanges': [{'CidrIp': '0.0.0.0/0'}]  # Allow SSH access from any IP address
                    },
                    {
                        'IpProtocol': 'tcp',
                        'FromPort': 80,
                        'ToPort': 80,
                        'IpRanges': [{'CidrIp': '0.0.0.0/0'}]  # Allow HTTP access from any IP address
                    }
                ]
            )
            print("Inbound rules added to the security group.\n")
        except Exception as create_error:
            # Handle any errors that occur during the creation of the security group
            print(f"An error occurred while creating security group: {create_error}\n")
    else:
        # Handle any unexpected errors
        print(f"Unexpected error: {e}\n")

# Step 2: Launch EC2 Instances in Different Availability Zones
instance_ids = []  # List to store instance IDs
subnet_ids = []    # List to store subnet IDs

try:
    # Launch the first EC2 instance in the specified availability zone (ap-northeast-3a)
    instance_1 = ec2.run_instances(
        ImageId=ami_id,
        InstanceType=instance_type,
        KeyName=key_name,
        MinCount=1,  # Minimum number of instances to launch
        MaxCount=1,  # Maximum number of instances to launch
        SecurityGroupIds=[security_group_id],  # Attach the security group to the instance
        Placement={'AvailabilityZone': 'ap-northeast-3a'},  # Specify the availability zone
        TagSpecifications=[
            {
                'ResourceType': 'instance',  # Tag the resource type as an instance
                'Tags': [{'Key': 'Name', 'Value': '23803313-vm1'}]  # Assign a name tag to the instance
            }
        ]
    )
    # Extract the instance ID and subnet ID for further configuration
    instance_1_id = instance_1['Instances'][0]['InstanceId']
    instance_ids.append(instance_1_id)
    subnet_ids.append(instance_1['Instances'][0]['SubnetId'])
    print(f"Instance 1 created: {instance_1_id} in ap-northeast-3a\n")

    # Launch the second EC2 instance in a different availability zone (ap-northeast-3b)
    instance_2 = ec2.run_instances(
        ImageId=ami_id,
        InstanceType=instance_type,
        KeyName=key_name,
        MinCount=1,
        MaxCount=1,
        SecurityGroupIds=[security_group_id],
        Placement={'AvailabilityZone': 'ap-northeast-3b'},  # Specify a different availability zone for redundancy
        TagSpecifications=[
            {
                'ResourceType': 'instance',
                'Tags': [{'Key': 'Name', 'Value': '23803313-vm2'}]
            }
        ]
    )
    # Extract the instance ID and subnet ID
    instance_2_id = instance_2['Instances'][0]['InstanceId']
    instance_ids.append(instance_2_id)
    subnet_ids.append(instance_2['Instances'][0]['SubnetId'])
    print(f"Instance 2 created: {instance_2_id} in ap-northeast-3b\n")

except Exception as e:
    # Handle any errors that occur during instance launch
    print(f"An error occurred while launching instances: {e}\n")

# Wait for instances to initialize by adding a delay
print("Waiting for instances to initialize...\n")
time.sleep(180)  # Pause for 3 minutes to allow instances to initialize fully

# Check the status of each instance to ensure they are running
for instance_id in instance_ids:
    instance_status = ec2.describe_instance_status(InstanceIds=[instance_id])
    print(f"Instance {instance_id} status: {instance_status}\n")

# Step 3: Create an Application Load Balancer
try:
    # Create an internet-facing Application Load Balancer to distribute incoming HTTP requests
    load_balancer = elbv2.create_load_balancer(
        Name='23803313-alb',        # Name of the load balancer
        Subnets=subnet_ids,         # Use the subnets of the created instances to attach to the load balancer
        SecurityGroups=[security_group_id],  # Attach the security group to allow HTTP access
        Scheme='internet-facing',   # Make the load balancer accessible from the internet
        Tags=[
            {'Key': 'Name', 'Value': '23803313-alb'}  # Tagging the load balancer with a name
        ],
        Type='application',         # Specify that this is an application load balancer
        IpAddressType='ipv4'        # Use IPv4 for addressing
    )
    # Extract the ARN (Amazon Resource Name) for the load balancer
    lb_arn = load_balancer['LoadBalancers'][0]['LoadBalancerArn']
    print(f"Load Balancer created: {lb_arn}\n")

except Exception as e:
    # Handle any errors that occur during the creation of the load balancer
    print(f"An error occurred while creating the load balancer: {e}\n")

# Step 4: Create a Target Group for the Load Balancer
try:
    # Use the VPC (Virtual Private Cloud) ID from the first instance to ensure compatibility
    vpc_id = instance_1['Instances'][0]['VpcId']
    # Create a target group for the load balancer to direct traffic to EC2 instances
    target_group = elbv2.create_target_group(
        Name='23803313-tg',         # Name of the target group
        Protocol='HTTP',            # Protocol for communication with targets
        Port=80,                    # Port on which the targets are listening
        VpcId=vpc_id,               # VPC ID for the target group
        HealthCheckProtocol='HTTP', # Protocol for health checks
        HealthCheckPort='80',       # Port for health checks
        HealthCheckPath='/',        # Path for health checks to determine if the instance is healthy
        TargetType='instance'       # Specify that the targets are EC2 instances
    )
    # Extract the ARN of the target group
    target_group_arn = target_group['TargetGroups'][0]['TargetGroupArn']
    print(f"Target Group created: {target_group_arn}\n")

except Exception as e:
    # Handle any errors that occur during the creation of the target group
    print(f"An error occurred while creating the target group: {e}\n")

# Step 5: Register the EC2 Instances as Targets in the Target Group
try:
    # Register the EC2 instances with the target group so that they receive traffic from the load balancer
    elbv2.register_targets(
        TargetGroupArn=target_group_arn,
        Targets=[{'Id': instance_id} for instance_id in instance_ids]  # Register each instance by its ID
    )
    print(f"Instances registered to target group: {target_group_arn}\n")

except Exception as e:
    # Handle any errors that occur during the registration of targets
    print(f"An error occurred while registering targets: {e}\n")

# Step 6: Create a Listener for the Load Balancer
try:
    # Create a listener that will forward incoming HTTP requests on port 80 to the target group
    listener = elbv2.create_listener(
        LoadBalancerArn=lb_arn,  # ARN of the load balancer to attach the listener to
        Protocol='HTTP',         # Protocol the listener will use to communicate with clients
        Port=80,                 # Port for the listener
        DefaultActions=[
            {
                'Type': 'forward',
                'TargetGroupArn': target_group_arn  # Forward requests to the target group
            }
        ]
    )
    # Extract the ARN of the listener
    listener_arn = listener['Listeners'][0]['ListenerArn']
    print(f"Listener created: {listener_arn}\n")

except Exception as e:
    # Handle any errors that occur during the creation of the listener
    print(f"An error occurred while creating the listener: {e}\n")

```
**Code Explanation**:
- **Security Group Check and Creation**: The script first checks if a security group named `23803313-sg` already exists. If not, it creates one with rules to allow inbound SSH (port 22) and HTTP (port 80) traffic from any IP address. This is essential for managing access to the EC2 instances.
- **EC2 Instances Launch**: Two EC2 instances are launched in separate availability zones (`ap-northeast-3a` and `ap-northeast-3b`). This improves availability and fault tolerance by distributing the instances across different zones.
- **Load Balancer Creation**: An Application Load Balancer named `23803313-alb` is created using the subnets of the EC2 instances, making it accessible over the internet (`internet-facing`).
- **Target Group Creation**: A target group named `23803313-tg` is created within the same VPC as the EC2 instances. The target group is configured to route HTTP traffic on port 80 and includes health checks to ensure that the instances are functioning correctly.
- **Register Targets**: The EC2 instances are registered to the target group, enabling the load balancer to distribute incoming traffic to the instances.
- **Listener Setup**: A listener is created on the load balancer to listen for HTTP requests on port 80 and forward them to the target group.

![image](https://github.com/user-attachments/assets/d8ed2b3e-1ff5-449d-95f8-158aa6da665e)

We can check our AWS console and see that the instance is created

![image](https://github.com/user-attachments/assets/9620cf6e-99da-403e-8765-a03b4758a389)

The load balancer is also created sucessfully

![image](https://github.com/user-attachments/assets/874d2d5a-df39-4309-b0c2-f8cd71cf8749)

### [2] Test the Application Load Balancer

After setting up the load balancer, I tested its functionality by accessing each EC2 instance using their public IP addresses.
- Accessing Instances Directly: Checked public IP addresses for each instance:
  - Instance 1: `13.208.252.206`
  - Instance 2: `15.152.119.216`

![image](https://github.com/user-attachments/assets/c3ee221e-4018-446f-97d6-c7d2fd87ad49)

![image](https://github.com/user-attachments/assets/35fd70e9-9826-44cc-9ad0-46675f9e3d53)

Initially, the load balancer did not work because Apache2 was not installed on the instances.

![image](https://github.com/user-attachments/assets/64253bb9-6f19-4c10-a0a6-248cd38a1e89)

![image](https://github.com/user-attachments/assets/ff0ef7e6-9328-4f50-8896-50014283bff0)

To resolve this, I SSH'd into each instance and installed Apache2 to serve a basic web page.

### [3] Steps to Install Apache2 on EC2 Instances
1. **SSH into Each Instance**:
   Initially, I attempted to SSH into the instances using their IP addresses directly, but I received a permission denied error:
   ```bash
   ssh -i "23803313-key.pem" ubuntu@13.208.252.206
   ssh -i "23803313-key.pem" ubuntu@15.152.119.216
   ```
   ![image](https://github.com/user-attachments/assets/13b24fb3-b026-4bc9-b2f0-7fe65b889031)
   
   To resolve this, I used the private key created earlier and the public DNS of the instances to connect via SSH:

   ```bash
   ssh -i "23803313-key.pem" ubuntu@ec2-13-208-252-206.ap-northeast-3.compute.amazonaws.com
   ssh -i "23803313-key.pem" ubuntu@ec2-15-152-119-216.ap-northeast-3.compute.amazonaws.com
   ```
   ![image](https://github.com/user-attachments/assets/ad76fc5e-2314-4577-bfc6-0861bd08023c)


2. **Update and Install Apache2**:
   Once connected, I updated the package lists and installed Apache2 to serve HTTP traffic:
   ```
   sudo apt-get update
   sudo apt install apache2
   ```

   ![image](https://github.com/user-attachments/assets/87380f27-3d28-450a-80d8-c3c29e2e4800)

   ![image](https://github.com/user-attachments/assets/3ee76eac-0ce1-45a3-bf63-21e12300c6ba)

3. **Customize Web Page**:
   I edited the `/var/www/html/index.html` file on each instance to display the instance name, confirming correct setup when accessed via the load balancer.

   ![image](https://github.com/user-attachments/assets/25fc2b4d-5707-4b14-99e6-7a199e6c3447)

### Final Verification
- Accessed the load balancer’s DNS name in a browser, and it successfully routed requests between the two instances, demonstrating load balancing functionality.

![image](https://github.com/user-attachments/assets/30c7b93c-0a5b-4f40-9045-4c401d68ce19)

![image](https://github.com/user-attachments/assets/5c16f4df-c605-4fad-904d-22a36922a758)

### Clean-Up
After verifying the load balancer's functionality, I deleted the EC2 instances and load balancer from the AWS console to avoid unnecessary costs.
