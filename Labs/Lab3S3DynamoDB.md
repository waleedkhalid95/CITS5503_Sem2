
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
```
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
