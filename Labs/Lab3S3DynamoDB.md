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

![image](https://github.com/user-attachments/assets/f8eb1b5f-6d45-4e2f-b338-6902580ea569)

### [6] Delete the table

After completing the tasks, I deleted the table using the AWS CLI:

```bash
aws dynamodb delete-table --table-name CloudFiles --region ap-northeast-3
```
This command deletes the specified DynamoDB table, cleaning up resources after the lab.

![image](https://github.com/user-attachments/assets/6c2f929e-8271-4a1c-a1e5-e3efa27dd285)

Finally, I removed the S3 bucket from the AWS console to complete the cleanup process.


<div style="page-break-after: always;"></div>
