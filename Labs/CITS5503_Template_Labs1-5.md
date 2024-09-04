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
- '--group-name': Specifies the name of the security group, making it easy to identify
- '--description': Describes the purpose of the security group. This security group acts as a virtual firewall to control inbound and outbound traffic for our EC2 instances. The output provides the security group ID, which we need for subsequent steps.

![image](https://github.com/user-attachments/assets/09a2b62f-df3c-47fa-8ea4-2f85e5ccc530)

Security groups are critical in AWS as they allow you to define which types of traffic can reach your EC2 instances. This step generated a security group ID, which I needed for the subsequent steps.

### [2] Authorize Inbound SSH Traffic
Next, I configured the security group to allow SSH access by modifying its inbound rules:

```bash
aws ec2 authorize-security-group-ingress --group-name 23803313-sg --protocol tcp --port 22 --cidr 0.0.0.0/0
```
- '--protocol tcp --port 22': Specifies that TCP traffic on port 22 (SSH) is allowed.
- '--cidr 0.0.0.0/0': Allows SSH access from any IP address. This setting, while useful for development, should be restricted in production environments to specific IP ranges for enhanced security.

By configuring SSH access, I ensured I could remotely manage and interact with the instance.

![image](https://github.com/user-attachments/assets/27d744f1-e297-4336-afbb-da10c11bb7e6)

### [3] Create a Key Pair and Set Permissions
To securely connect to our EC2 instance, we create a key pair using:

```bash
aws ec2 create-key-pair --key-name 23803313-key --query 'KeyMaterial' --output text > 23803313-key.pem
```

- '--key-name': Specifies the name of the key pair.
- '--query 'KeyMaterial' --output text > 23803313-key.pem': Extracts the key material and saves it to a .pem file. This private key is essential for SSH access.

I then secured the key by changing its permissions to read-only for the owner:

```bash
chmod 400 23803313-key.pem
```

This step is crucial because it prevents unauthorized access to the key, ensuring that only I can use it to connect to the instance.

![image](https://github.com/user-attachments/assets/f203ae30-0fc6-4ea0-ac07-72b9b908a1bc)

### [4] Launch the EC2 Instance
Using the AMI ID for the Osaka region, I launched the EC2 instance:

```bash
 aws ec2 run-instances --image-id ami-0a70c5266db4a6202 --security-group-ids 23803313-sg --count 1 --instance-type t2.micro --key-name 23803313-key --query 'Instances[0].InstanceId'

 ```
Instace created i-0dcfef96ec413ecca

![image](https://github.com/user-attachments/assets/3aec8350-8576-4ef9-b344-9f664f8fde70)

- '--image-id': Specifies the Amazon Machine Image (AMI) ID, which serves as a template for the instance.
- '--security-group-ids': Associates the instance with the security group created earlier.
- '--instance-type t2.micro': Specifies a cost-effective instance type suitable for development.
- '--key-name': Specifies the key pair for SSH access.
  
This command launched the instance and returned an instance ID, confirming the successful creation.

### [5] Tag the Instance
To make it easier to identify and manage the instance, I added a descriptive tag:

 ```bash
  aws ec2 create-tags --resources i-0dcfef96ec413ecca --tags Key=Name,Value=23803313-vm1
 ```
![image](https://github.com/user-attachments/assets/50613443-6ef9-4d86-a60f-36324e391364)

- '--resources': Specifies the instance ID to tag.
- '--tags Key=Name,Value=23803313-vm1': Adds a descriptive tag to the instance. This helps in managing and identifying instances in the AWS console.
Tags are helpful for organizing resources, especially when managing multiple instances.

### [6] Retrieve the Public IP Address
To connect to our instance, we need its public IP address, obtained via:

```bash
aws ec2 describe-instances --instance-ids i-0dcfef96ec413ecca --query 'Reservations[0].Instances[0].PublicIpAddress'
```
This command queries the instance details and extracts the public IP, 13.208.91.27.

![image](https://github.com/user-attachments/assets/f3ffee53-faed-44a1-a36d-326a7f9d6c29)

### [7] Connect to the Instance via SSH
Finally, we connect to the instance using SSH:
```bash
ssh -i 23803313-key.pem ubuntu@13.208.91.27"
```
![image](https://github.com/user-attachments/assets/42851d5a-8d3b-4e82-a78e-f5dbe1b79c42)

This command establishes a secure connection to the instance using the private key and public IP address, enabling remote management and interaction.

### [8] List the Instance in AWS Console
After completing these steps, the instance was visible and manageable through the AWS Console, where I could monitor and configure it as needed.

![image](https://github.com/user-attachments/assets/2d83568f-3fc4-47e6-9789-eb175386806d)

## EC2 Instance Setup Using Python Boto3
I automated the EC2 setup process using Python's Boto3 SDK, which allowed for more flexibility and integration into larger automation workflows. Here’s the Python script I used.

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
**Code Explanation:**
 - Initialize EC2 Client: boto3.client('ec2') initializes the EC2 client to interact with AWS services.
 - Create Security Group: The script creates a security group using ec2.create_security_group(), which includes a description and group name.
 - Authorize SSH Access: SSH access is enabled using ec2.authorize_security_group_ingress() with TCP protocol and port 22, allowing connections from all IP addresses (0.0.0.0/0).
 - Create Key Pair: A key pair is generated using ec2.create_key_pair(), and the private key material is saved to a .pem file. The file permissions are set to 400 to secure the key.
 - Launch EC2 Instance: The instance is launched with ec2.run_instances(), specifying the AMI ID, security group, instance type, and key name. It outputs the instance ID upon successful creation.
 - Tag Instance: The instance is tagged using ec2.create_tags() to make it identifiable in the AWS console.
 - Retrieve Public IP: The instance's public IP address is obtained with ec2.describe_instances(), which is necessary for connecting via SSH.
 - Connect via SSH: The script attempts to connect to the instance using SSH, automating the login process and enabling direct management of the instance from the terminal.

![image](https://github.com/user-attachments/assets/653b635c-203d-4166-af9c-633b7b47351a)

![image](https://github.com/user-attachments/assets/95724b9d-86ea-4326-897b-6f2e5805bf3b)

## Use Docker Inside a Linux OS
Docker allows for containerized applications, simplifying the deployment and management of applications in a consistent environment. To demonstrate Docker, I installed it on the EC2 instance and ran a simple HTTP server
### [1] Install Docker
I installed Docker using the following command
```bash
sudo apt install docker.io -y
```

![image](https://github.com/user-attachments/assets/e0813438-36c5-40ab-bc77-8e4c7dcba9d0)

This command installs Docker on the instance, enabling container management.

### [2] Start and Enable Docker
I started Docker and enabled it to run on boot with the following commands
```bash
sudo systemctl start docker
sudo systemctl enable docker
docker --version
```
Starting Docker ensures the service runs immediately, and enabling it makes Docker start automatically on boot. Then we verify the Docker installation by checking its version.

![image](https://github.com/user-attachments/assets/93a5c2f0-a8aa-468a-b638-b7f874f4a53f)

### [4] Build and Run an HTTPD Container

To demonstrate Docker’s utility, I built and ran a simple HTTP server container:
- Created a directory called html and added a file index.html with the content

```
  <html>
    <head> </head>
    <body>
      <p>Hello World!</p>
    </body>
  </html>
```
- Created a Dockerfile outside the html directory with:
```bash
FROM httpd:2.4
COPY ./html/ /usr/local/apache2/htdocs/
```
The Dockerfile uses the official HTTPD (Apache) image and copies the contents of the html directory into the container's web root.

- Build the docker image

```bash
docker build -t my-apache2 .
```

![image](https://github.com/user-attachments/assets/331a755a-35f8-40be-bde6-6cf97188b517)

- Run the container

```bash
docker run -p 80:80 -dit --name my-app my-apache2
```
This command runs the container, mapping port 80 on the instance to port 80 in the container, allowing me to access the server via the instance's IP address

![image](https://github.com/user-attachments/assets/47f797bc-af2b-41ae-bae0-539f87aef712)

- Visit http://localhost to confirm the "Hello World!" message displays.

![image](https://github.com/user-attachments/assets/8549d00a-5cb6-4ff4-a51c-4b4114f3902e)

### [5] Other docker commands

To check running containers

```bash
docker ps -a
```
To stop and remove the container

```bash
docker stop my-app
docker rm my-app
```
These commands allow for managing Docker containers, stopping them when they are no longer needed, and cleaning up resources

![image](https://github.com/user-attachments/assets/a9d48537-6705-465b-8cbe-f3f54ea79a98)

<div style="page-break-after: always;"></div>


# Lab 3: Cloud Storage with S3 and DynamoDB

## Summary

In this lab, I set up a personal cloud storage application using AWS services. The main objectives were to create and configure S3 buckets, work with DynamoDB for storing file metadata, and restore files from the cloud back to a local environment. By the end of this lab, I successfully scanned a directory, uploaded files to an S3 bucket, stored metadata in DynamoDB, and restored the files to a local directory.

## Program Step
### [1] Preparation

I started by preparing the environment:

1. Downloaded the Python code `cloudstorage.py` from the [src](https://github.com/zhangzhics/CITS5503_Sem2/blob/master/Labs/src/cloudstorage.py) directory.
2. Created a directory named `rootdir`.
3. Inside `rootdir`, I created a file named `rootfile.txt` and added the content `1\n2\n3\n4\n5\n`.
4. Next, I created a subdirectory named `subdir` within `rootdir` and added another file, `subfile.txt`, containing the same content as `rootfile.txt`.

This setup allowed me to create a nested directory structure, which would be replicated in the S3 bucket.

### [2] Save to S3 by updating `cloudstorage.py`
I modified the `cloudstorage.py` script to create an S3 bucket named `23803313-cloudstorage` and upload files to it. The script used Boto3 to interact with AWS services.

Here’s the modified code:
```python
import os
import boto3
import base64

ROOT_DIR = './'
ROOT_S3_DIR = '23803313-cloudstorage'

s3 = boto3.client("s3")
bucket_config = {'LocationConstraint': 'ap-northeast-3'} #Replace the region with your allocated region name.

def upload_file(folder_name, file, file_name):
	s3.upload_file(file, ROOT_S3_DIR, f"{folder_name}/{file_name}")
	print("Uploading %s" % file)

try:
	response=s3.create_bucket(Bucket=ROOT_S3_DIR, CreateBucketConfiguration=bucket_config)
	print(f"Bucket {ROOT_S3_DIR} created: {response}")
except Exception as error:
	print(f"Bucket creation failed: {error}")
	pass

# parse directory and upload files
for dir_name, subdir_list, file_list in os.walk(ROOT_DIR, topdown=True):
    if dir_name != ROOT_DIR:
        for fname in file_list:
            upload_file("%s/" % dir_name[2:], "%s/%s" % (dir_name, fname), fname)
print("done")
```
**Code Explanation:**
 - boto3.client("s3") initializes an S3 client, which allows the script to interact with AWS S3.
 - The script attempts to create an S3 bucket named 23803313-cloudstorage with the specified region (ap-northeast-3).
 - The create_bucket method includes a CreateBucketConfiguration parameter, which specifies the bucket's region using 'LocationConstraint'.
 - upload_file(folder_name, file, file_name): This function uploads files to the S3 bucket, preserving the directory structure.
 - It uses s3.upload_file(file, ROOT_S3_DIR, f"{folder_name}/{file_name}"), where file is the local file path. ROOT_S3_DIR is the bucket name. f"{folder_name}/{file_name}" specifies the target path in the bucket, ensuring the folder structure is maintained.
 - The script uses os.walk() to traverse ROOT_DIR, identifying files and directories. It uploads each file using the upload_file function.
 - This approach ensures all files are uploaded to the S3 bucket, replicating the local directory structure.

Upon running the script, the directory structure from rootdir was replicated in the S3 bucket, with files correctly uploaded.

![image](https://github.com/user-attachments/assets/77e64c70-11a5-4f27-b6f0-212278b5b2b8)

I verified the bucket and file creation through the AWS console.

![image](https://github.com/user-attachments/assets/154c624d-8d9b-4162-be8a-f6c199eab45a)

### [3] Restore from S3

Next, I created a new script restorefromcloud.py to restore the directory and files from S3 to a local directory named Restored.
Here’s the script:

```python
import boto3
import os

BUCKET_NAME = '23803313-cloudstorage' 
s3 = boto3.resource('s3')
try:
    # List objects in the specified S3 bucket
    response = s3.meta.client.list_objects_v2(Bucket=BUCKET_NAME)
    # Check if 'Contents' key exists in the response
    if 'Contents' not in response:
        print(f"No files found in bucket {BUCKET_NAME}.")
    else:
        for obj in response['Contents']:
            s3_key = obj['Key']
            print(f"Restoring {s3_key} from S3...")

            # Define the local path where the file will be saved
            local_path = os.path.join('./', s3_key)
            local_dir = os.path.dirname(local_path)
            
            # Create local directory if it does not exist
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
 - boto3.resource('s3') initializes an S3 resource, providing a higher-level interface for interacting with S3.
 - s3.meta.client.list_objects_v2(Bucket=BUCKET_NAME) retrieves the list of objects in the specified S3 bucket.
 - The script checks if the Contents key exists in the response to ensure files are available for restoration.
 - For each file (s3_key) in the bucket, the script constructs the local path where the file will be saved using os.path.join('./', s3_key).
 - If the necessary directories do not exist (local_dir), they are created using os.makedirs(local_dir).
 - Files are downloaded from S3 to the local path with s3.meta.client.download_file(BUCKET_NAME, s3_key, local_path), replicating the original directory structure.

After running this script, the Restored directory was populated with the files and structure from the S3 bucket, successfully restoring the original setup.

![image](https://github.com/user-attachments/assets/0a3f7e58-b258-4432-bc35-dfe6906fb90a)

### [4] Write Information About Files to DynamoDB

To store metadata about the files in DynamoDB, I first installed DynamoDB locally using:
```bash
mkdir dynamodb
cd dynamodb
sudo apt-get install default-jre
wget https://s3-ap-northeast-1.amazonaws.com/dynamodb-local-tokyo/dynamodb_local_latest.tar.gz
tar -zxvf dynamodb_local_latest.tar.gz
java -Djava.library.path=./DynamoDBLocal_lib -jar DynamoDBLocal.jar –sharedDb
```

Next, I wrote a Python script to create a table named CloudFiles and store file metadata:
```python
import boto3
import os
from datetime import datetime
BUCKET_NAME = '23803313-cloudstorage'  
REGION_NAME = 'ap-northeast-3' 
# Initialize AWS resources
s3 = boto3.client('s3')
dynamodb = boto3.resource('dynamodb', region_name=REGION_NAME)  # Modify if using AWS DynamoDB
# Define the table name
table_name = 'CloudFiles'

# Create DynamoDB table if it doesn't exist
existing_tables = dynamodb.meta.client.list_tables()['TableNames']
if table_name not in existing_tables:
    table = dynamodb.create_table(
        TableName=table_name,
        KeySchema=[
            {
                'AttributeName': 'userId',
                'KeyType': 'HASH'  # Partition key
            },
            {
                'AttributeName': 'fileName',
                'KeyType': 'RANGE'  # Sort key
            }
        ],
        AttributeDefinitions=[
            {
                'AttributeName': 'userId',
                'AttributeType': 'S'
            },
            {
                'AttributeName': 'fileName',
                'AttributeType': 'S'
            }
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

# List objects in the specified S3 bucket
response = s3.list_objects_v2(Bucket=BUCKET_NAME)

# Check if 'Contents' key exists in the response
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

        # Define item attributes
        item = {
            'userId': '23803313',
            'fileName': os.path.basename(s3_key),
            'path': os.path.dirname(s3_key),
            'lastUpdated': head_response['LastModified'].strftime('%Y-%m-%d %H:%M:%S'),
            'owner': owner,
            'permissions': ', '.join(permissions)  # Converting list to string
        }

        # Insert item into DynamoDB table
        try:
            table.put_item(Item=item)
            print(f"Inserted {s3_key} into DynamoDB.")
        except Exception as e:
            print(f"Failed to insert {s3_key} into DynamoDB: {e}")

print("Process complete.")
```
**Code Explanation:**
 - DynamoDB Resource Initialization: boto3.resource('dynamodb', region_name=REGION_NAME) initializes a DynamoDB resource pointing to the specified region.
 - Table Creation: The script checks if the table CloudFiles exists using list_tables().
 - If the table does not exist, it creates one with dynamodb.create_table() using userId as the partition key and fileName as the sort key. Both keys are of type string (S).
 - The ProvisionedThroughput is set with read and write capacity units.
 - A waiter is used to ensure the table is fully created before proceeding.
 - Fetching File Metadata:The script lists objects in the S3 bucket using list_objects_v2() and retrieves metadata using head_object() and access permissions with get_object_acl().
 - It extracts the owner's name or ID based on the region and compiles permissions into a string.
 - Inserting Metadata into DynamoDB: Metadata for each file is structured into an item dictionary and inserted into the CloudFiles table using put_item().

![image](https://github.com/user-attachments/assets/d87a04bc-d51b-42b8-879c-295635aaad25)


### [5] Scan the table

I used the AWS CLI to scan the CloudFiles table and output the data:
```bash
aws dynamodb delete-table --table-name CloudFiles --region ap-northeast-3
```

![image](https://github.com/user-attachments/assets/cc8ed6c6-b27c-458a-836b-8f147675c205)

### [6] Delete the table

After completing the tasks, I deleted the table using:

```bash
aws dynamodb delete-table --table-name CloudFiles --region ap-northeast-3
```

![image](https://github.com/user-attachments/assets/6c2f929e-8271-4a1c-a1e5-e3efa27dd285)

Finally, I removed the S3 bucket from the AWS console.


<div style="page-break-after: always;"></div>

# Lab 4

<div style="page-break-after: always;"></div>

# Lab 5

