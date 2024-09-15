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
