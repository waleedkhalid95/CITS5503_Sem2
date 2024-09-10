# Practical Worksheet 4

Version: 1.2 Date: 23/08/2018 Author: David Glance

Date: 25/07/2024 Updated by Zhi Zhang

## Learning Objectives

1. IAM policies applied to S3
2. KMS Key Management System – creating keys and using the key for symmetric encryption
3. Using AES Encryption for client and server side encryption

## Technologies Covered

* Ubuntu
* AWS
* AWS KMS
* AES Encryption
* Python/Boto scripts
* VirtualBox

**NOTE**: please use your Linux environment – if you do it from any other OS (e.g., Windows, Mac – some unknow issues might occur)

## Background

The aim of this lab is to write a program that will:

1. Apply a policy to your bucket to allow only you as a user to access it
2. Create a key in KMS and use it to encrypt files on the client before uploading to S3 and decrypt them after downloading from S3
3. Implement AES using python and test the difference in performance between the KMS solution and the local one.

## Apply a policy to restrict permissions on bucket

### [1] Write a Python script

Apply the following policy to the S3 bucket you created in the last lab to allow only your username to access the bucket. Make appropriate changes (e.g., `Resource`, `Condition`, etc) to the policy as necessary.

**NOTE**: in the policy below, you should replace `<your_s3_bucket>` with the S3 bucket you created and `<studentnumber>` with your own student number. You can use AWS console to create the S3 bucket in this lab that has the same contents as the bucket in the last lab.

```
{
  "Version": "2012-10-17",
  "Statement": [{
   "Sid": "AllowAllS3ActionsInUserFolderForUserOnly",
    "Effect": "DENY",
    "Principal": "*",
    "Action": "s3:*",
    "Resource": "arn:aws:s3:::<your_s3_bucket>/folder1/folder2/*",
    "Condition": {
      "StringNotLike": {
          "aws:username":"<studentnumber>@student.uwa.edu.au"
       }
    }
  }]
}
```

```python
import boto3
import json

# Initialize the S3 client
s3 = boto3.client('s3')

# Define the bucket name and policy
bucket_name = '23803313-cloudstorage'
policy = {
    "Version": "2012-10-17",
    "Statement": [{
        "Sid": "AllowAllS3ActionsInUserFolderForUserOnly",
        "Effect": "Deny",
        "Principal": "*",
        "Action": "s3:*",
        "Resource": f"arn:aws:s3:::{bucket_name}/*",
        "Condition": {
            "StringNotLike": {
                "aws:username": "23803313@student.uwa.edu.au"
            }
        }
    }]
}

# Convert the policy to a JSON string
policy_json = json.dumps(policy)

# Apply the policy to the bucket
try:
    s3.put_bucket_policy(Bucket=bucket_name, Policy=policy_json)
    print(f"Policy applied to bucket {bucket_name} successfully.")
except Exception as e:
    print(f"Failed to apply policy: {e}")

```

![image](https://github.com/user-attachments/assets/78fb279e-c24d-448a-bd14-fe188da2482f)

### [2] Check whether the script works

Use AWS CLI command and AWS S3 console to display the policy content applied to the S3 bucket. 

```bash
aws s3api get-bucket-policy --bucket 23803313-cloudstorage
```

![image](https://github.com/user-attachments/assets/aca1f3b9-6783-4d3d-a891-2dc265ceb2e9)

Access the Bucket Using Your Correct Username
To test access with your correct username (23803313@student.uwa.edu.au), you can use AWS CLI commands or the AWS S3 console. Since this username is supposed to have access, you should be able to perform actions like listing the bucket contents or downloading files.

```bash
aws s3 ls s3://23803313-cloudstorage/rootdir/
```

![image](https://github.com/user-attachments/assets/e5dd8734-4b4a-4719-94f3-ebe95915cc2e)

Test the policy by using a username that is not your to access the folder called `rootdir` and output what you've got. 

Ask friend to access 

![image](https://github.com/user-attachments/assets/a7b53aae-db90-4b5d-b32d-dd6d8bf4bc1a)


## AES Encryption using KMS

### [1] Create a KMS key

Write a Python script to create a KMS key, where your student number works as an alias for the key.

### [2] Attach a policy to the created KMS key

Update the script to attach the following policy to the key.

**NOTE**: in the policy below, you should replace `<your_username>` with your own username.


```
{
  "Version": "2012-10-17",
  "Id": "key-consolepolicy-3",
  "Statement": [
    {
      "Sid": "Enable IAM User Permissions",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::489389878001:root"
      },
      "Action": "kms:*",
      "Resource": "*"
    },
    {
      "Sid": "Allow access for Key Administrators",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::489389878001:user/<your_username>"
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
        "AWS": "arn:aws:iam::489389878001:user/<your_username>"
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
        "AWS": "arn:aws:iam::489389878001:user/<your_username>"
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
```

### [3] Check whether the script works

Use the AWS KMS console to test whether your username is the key administrator and key user.

 ![image](https://github.com/user-attachments/assets/ba9a2e51-d557-4f33-85c4-68659ea575fa)

**NOTE**: After you log into the console, you perform the test by showing the policy you create, i.e., which ARN is the key administrator and which ARN is the key user.

![image](https://github.com/user-attachments/assets/5d480602-0076-4f53-87f6-ffd95f6e21ae)


### [4] Use the created KMS key for encryption/decryption

Write a Python script where each file from the S3 bucket is encrypted and then decrypted via the created KMS key. Both encrypted and decrypted files will be in the same folder as the original file. 

![image](https://github.com/user-attachments/assets/34e8ff52-27df-44f3-83d6-a2cfae83cf47)

![image](https://github.com/user-attachments/assets/ab9587f5-12ae-47d7-853b-b8f555ae113d)


### [5] Apply `pycryptodome` for encryption/decryption

Write another Python script that uses the python library `pycryptodome` to encrypt and decrypt each file in the S3 bucket. Both encrypted and decrypted files will be in the same folder as the original file.

![image](https://github.com/user-attachments/assets/70d145c5-8096-4b2b-b24b-381128dd5ec7)


For encryption/decryption, refer to the example code from [fileencrypt.py](https://github.com/zhangzhics/CITS5503_Sem2/blob/master/Labs/src/fileencrypt.py)

**NOTE**: Delete the created S3 bucket and KMS key from AWS console after the lab is done.

## Answer the following question (Marked)

```
What is the performance difference between using KMS and using the custom solution?
```

Lab Assessment:

A structured presentation (15%). A clear step-by-step with detailed descriptions (85%). 
