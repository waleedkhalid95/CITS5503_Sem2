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
   - <Insert screenshot of Access Key Best Practices & Alternatives, hovering on CLI option>
5. Set the description tag.
   - <Insert screenshot of Description Tag page>
6. Clicked on **Create access key**.
   - A confirmation screen popped up confirming that the access key is created.
   - <Insert screenshot of confirmation screen>
7. Made a note of the ID and password or downloaded the CSV file containing the ID and password.

## Set up recent Linux OSes

1. **Download and install VMware for Windows.**
2. **Download Kali Linux for VMware and extract the 7z file.**
3. **Open VMware:**
   - Click on **File** on the top right and select **Open**.
   - Find the VMX file for Kali Linux in the extracted directory.
   - <Insert screenshot>
4. **Edit Virtual Machine Settings:**
   - Set memory to 8GB, 4 processor cores, 30GB hard disk, and NAT network.
   - <Insert screenshot>
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
   - <Insert screenshot>
2. Check Python version and install pip:
   - "python3.8 --version"
     - **`python3.8`**: Specifies the Python 3.8 version.
     - **`--version`**: Displays the version of the installed Python.
   - "sudo apt install python3-pip"
     - **`install`**: Installs the specified package.
     - **`python3-pip`**: Pip is the package installer for Python.
   - <Insert screenshot>

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

### [3] Configure AWS

1. Configure AWS CLI:
   - "aws configure"
     - **`configure`**: Starts the AWS configuration process.
   - Enter Access Key ID: "AKIAXD4PI5LY5HOWG256"
   - Enter Secret Access Key.
   - Default region name: "ap-northeast-3" (as per student ID range).
   - Default output format: "json".

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
   - <Insert screenshot>

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
   - <Insert screenshot>

### [3] Write a Python script

1. **Create a folder on the Desktop named `cloud-lab`.**
2. **Create an empty file and name it `lab1.py`.**
3. **Open the file and add the following Python script, then save:**
   - "import boto3"
   - "import pandas as pd"
   - "from tabulate import tabulate"
   - ""
   - "ec2 = boto3.client('ec2')"
   - "response = ec2.describe_regions()"
   - "regions = response['Regions']"
   - "df = pd.DataFrame(regions, columns=['Endpoint', 'RegionName'])"
   - "print(tabulate(df, headers='keys', tablefmt='psql'))"
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
   - <Insert screenshot>

<div style="page-break-after: always;"></div>

# Lab 2

## Create an EC2 instance using awscli
### [1] Create a security group

```
aws ec2 create-security-group --group-name 23803313-sg --description "security group for development environment"
```

This will use the default VPC (if you want to specify a VPC, use --vpc-id vpc-xxxxxxxx). Take a note of the security group id that is created. 

### [2] Authorise inbound traffic for ssh

```
aws ec2 authorize-security-group-ingress --group-name 23803313-sg --protocol tcp --port 22 --cidr 0.0.0.0/0
```

### [3] Create a key pair

```
aws ec2 create-key-pair --key-name 23803313-key --query 'KeyMaterial' --output text > 23803313-key.pem
```

To use this key on Linux, copy the file to a directory ~/.ssh and change the permissions to:

```
chmod 400 23803313-key.pem
```

![image](https://github.com/user-attachments/assets/f6a91887-c276-465f-a00c-b4757f686976)

### [4] Create the instance 

| Student Number | Region | Region Name | ami id |
| --- | --- | --- | --- |
| 20666666 – 22980000 | US East (N. Virginia) |	us-east-1 |	ami-0a0e5d9c7acc336f1 |
| 22984000 – 23370000 | Asia Pacific (Tokyo)	| ap-northeast-1	| ami-0162fe8bfebb6ea16 |
| 23400000 – 23798000 | Asia Pacific (Seoul)	| ap-northeast-2	| ami-056a29f2eddc40520 |
| 23799000 – 23863700 | Asia Pacific (Osaka)	| ap-northeast-3	| ami-0a70c5266db4a6202 |
| 23864000 – 23902200 | Asia Pacific (Mumbai)	| ap-south-1	| ami-0c2af51e265bd5e0e |
| 23904000 – 23946000 | Asia Pacific (Singapore)	| ap-southeast-1	| ami-0497a974f8d5dcef8 |
| 23946100 – 24024000 | Asia Pacific (Sydney)	| ap-southeast-2	| ami-0375ab65ee943a2a6 |
| 24025000 – 24071000 | Canada (Central)	| ca-central-1	| ami-048ddca51ab3229ab |
| 24071100 – 24141000 | Europe (Frankfurt)	| eu-central-1	| ami-07652eda1fbad7432 |
| 24143000 – 24700000 | Europe (Stockholm)	| eu-north-1	| ami-07a0715df72e58928 |


Based on your region code, find the corresponding ami id in the table above and fill it in the command below:

```
 aws ec2 run-instances --image-id ami-0a70c5266db4a6202 --security-group-ids 23803313-sg --count 1 --instance-type t2.micro --key-name 23803313-key --query 'Instances[0].InstanceId'

 ```

If you are allocated to Europe (Stockholm), eu-north-1, please use `t3.micro` to replace `t2.micro` in the command above.

### [5] Add a tag to your Instance

 ```
  aws ec2 create-tags --resources i-0b3193d1eefc9086f --tags Key=Name,Value=23803313-vm1
 ```
**NOTE**: If you need to create a single instance, follow the naming format of `<student number>-vm` (e.g., 24242424-vm). If you need to create multiple ones, follow the naming format of `<student number>-vm1` and `<student number>-vm2` (e.g., 24242424-vm1, 24242424-vm2).

### [6] Get the public IP address

```
aws ec2 describe-instances --instance-ids i-0b3193d1eefc9086f --query 'Reservations[0].Instances[0].PublicIpAddress'
```

### [7] Connect to the instance via ssh
```
ssh -i 23803313-key.pem ubuntu@13.208.193.75"
```
![image](https://github.com/user-attachments/assets/78313900-62ae-4a51-8f67-db49ef95f508)


### [8] List the created instance using the AWS console

![image](https://github.com/user-attachments/assets/614b9593-1e17-4fa6-8d58-cfab1b030a10)

## Create an EC2 instance with Python Boto3

Use a Python script to implement the steps above (steps 1-6 and 8 are required, step 7 is optional). Refer to [page](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/ec2.html) for details.

**NOTE**: If you are allocated to Europe (Stockholm), eu-north-1, the type of the instance in your script should be `t3.micro` rather than `t2.micro`. When you are done, log into the EC2 console and terminate the instances you created.

## Use Docker inside a Linux OS

### [1] Install Docker
```
sudo apt install docker.io -y
```

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

If you run into permission errors, you may need add your user to the docker group:

```
sudo usermod -a -G docker <username>
```

Be sure to log out and log back in for this change to take effect.

Run the image

```
docker run -p 80:80 -dit --name my-app my-apache2
```

Open a browser and access address: http://localhost or http://127.0.0.1. 

Confirm you get "Hello World!"

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


<div style="page-break-after: always;"></div>

# Lab 3

<div style="page-break-after: always;"></div>

# Lab 4

<div style="page-break-after: always;"></div>

# Lab 5

