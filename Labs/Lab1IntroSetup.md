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
