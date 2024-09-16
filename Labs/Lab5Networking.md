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
