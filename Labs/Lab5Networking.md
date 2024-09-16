# lab 5


## Application Load Balancer

The aim of this part of the lab is to create an application load balancer and load balance requests to 2 EC2 instances. 

### [1] Create 2 EC2 instances

Write a Python Boto3 script to create 2 EC2 instances (the instance type can be `t3.micro`) in two different availability zones (name the instances following the format: 23803313-vm1 and 23803313-vm2) in the region mapped to your student number. In this script, a security group should be created to authorise inbound traffic for HTTP and SSH, which will be used by the following steps. 

**NOTE**: Regarding your region name, find it in the table below based on your student number (If you cannot find your region name, it means you enrolled late and you should send an email to `cits5503-pmc@uwa.edu.au` requesting your region name.).

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

### [2] Create an Application Load Balancer

Update the script above to create an application load balancer and load balance HTTP requests to the created 2 instances. Note that the v2 of the ELB interface below should be used:

```
client = boto3.client('elbv2')
```

The script updates include:

First, create a load balancer, during which specify the two created region subnets and the
security group created in the previous step.

Second, create a target group using the same VPC that was used to create
the instances.

Third, register targets in the target group.

Last, create a listener with a default rule Protocol: HTTP and Port 80
forwarding on to the target group.

![image](https://github.com/user-attachments/assets/d8ed2b3e-1ff5-449d-95f8-158aa6da665e)

checking console and we can see that the instance is created

![image](https://github.com/user-attachments/assets/9620cf6e-99da-403e-8765-a03b4758a389)

load balancer is also created
![image](https://github.com/user-attachments/assets/874d2d5a-df39-4309-b0c2-f8cd71cf8749)

### [3] Test the Application Load Balancer

Try and access each EC2 instance using its public IP address in a browser. The load balancer is expected not to work at the moment, because Apache 2 is not installed in the instance. 

i checked the console for the public ip of the instances 

![image](https://github.com/user-attachments/assets/c3ee221e-4018-446f-97d6-c7d2fd87ad49)

![image](https://github.com/user-attachments/assets/35fd70e9-9826-44cc-9ad0-46675f9e3d53)

13.208.252.206 for 23803313-vm1
15.152.119.216 for 23803313-vm2

hewever we cannot access them

![image](https://github.com/user-attachments/assets/64253bb9-6f19-4c10-a0a6-248cd38a1e89)

![image](https://github.com/user-attachments/assets/ff0ef7e6-9328-4f50-8896-50014283bff0)

To make it work, follow the steps below:

First, ssh to each of the two instances. 
```bash
ssh -i "23803313-key.pem" ubuntu@13.208.252.206
ssh -i "23803313-key.pem" ubuntu@15.152.119.216
```
however we get an permission denied error

then we use the private key we created earlier and the public dns to create the ssh
```bash
ssh -i "23803313-key.pem" ubuntu@ec2-13-208-252-206.ap-northeast-3.compute.amazonaws.com
ssh -i "23803313-key.pem" ubuntu@ec2-15-152-119-216.ap-northeast-3.compute.amazonaws.com
```

![image](https://github.com/user-attachments/assets/13b24fb3-b026-4bc9-b2f0-7fe65b889031)

![image](https://github.com/user-attachments/assets/ad76fc5e-2314-4577-bfc6-0861bd08023c)


If you can't make it, try [here](https://bobbyhadz.com/blog/aws-ssh-permission-denied-publickey).

Second, update each instance:

```
sudo apt-get update
```

![image](https://github.com/user-attachments/assets/87380f27-3d28-450a-80d8-c3c29e2e4800)

Third, install apache2 in each instance:

```
sudo apt install apache2
```
![image](https://github.com/user-attachments/assets/3ee76eac-0ce1-45a3-bf63-21e12300c6ba)

Fourth, edit the `<title>` and `</title>` tags inside the `/var/www/html/index.html` file to show the instance name.
![image](https://github.com/user-attachments/assets/25fc2b4d-5707-4b14-99e6-7a199e6c3447)

Last, use a browser from your host OS to access each instance by their respective IP address and see if you can get an Apache web page that shows your instance name. Output what you've got. If you are using the University network, it is likely that you cannot access the installed apache2. To address this issue, you may switch to a non-university network.

![image](https://github.com/user-attachments/assets/30c7b93c-0a5b-4f40-9045-4c401d68ce19)

![image](https://github.com/user-attachments/assets/5c16f4df-c605-4fad-904d-22a36922a758)


**NOTE**: Delete all the created AWS resources from AWS console after the lab is done.

Lab Assessment:

A structured presentation (15%). A clear step-by-step with detailed descriptions (85%). 
