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
