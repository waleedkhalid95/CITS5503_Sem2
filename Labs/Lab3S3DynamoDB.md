# Practical Worksheet 3

Version: 1.0 Date: 12/04/2018 Author: David Glance

Date: 24/07/2024 Updated by Zhi Zhang

## Learning Objectives

1. Learn how to create and configure S3 buckets and read and write objects to them
2. Learn how to use operations on DynamoDB: Create table, put items, get items
3. Start an application as your own personal Cloud Storage

## Technologies Covered

* Ubuntu
* AWS
* AWS S3
* AWS DynamoDB
* Python/Boto scripts
* VirtualBox

**NOTE**: please use your Linux environment – if you do it from any other OS (e.g., Windows, Mac – some unknow issues might occur)

## Background

The aim of this lab is to write a program that will:

[1] Scan a directory and upload all of the files found in a directory to an S3 bucket, preserving the path information

[2] Store information about each file uploaded to S3 in a DynamoDB

[3] Restore the directory on a local drive using the files in S3 and the information in DynamoDB

## Program

### [1] Preparation

Download the python code `cloudstorage.py` from the directory of [src](https://github.com/zhangzhics/CITS5503_Sem2/blob/master/Labs/src/cloudstorage.py) \
First we create a directory `rootdir` \
then create a file in `rootdir` called `rootfile.txt` and write some content in it `1\n2\n3\n4\n5\n` \
then we create a second directory in rootdir called `subdir`, and in the `subdir` directory create another file `subfile.txt` with the same content as `rootfile.txt`. This is done to create a nested directory and subdirectory that we can creplicate in our s3 bucket

### [2] Save to S3 by updating `cloudstorage.py`

we downloaded Python script, `cloudstorage.py`, which contains python boto3 code to traverse current directory and uplode files, but we must modify it to create an S3 bucket named `23803313-cloudstorage`.

When the program traverses the directory starting at the root directory `rootdir`, upload each file onto the S3 bucket using the fucktion upload_file() 

```
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


# Main program
# Insert code to create bucket if not there

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
![image](https://github.com/user-attachments/assets/77e64c70-11a5-4f27-b6f0-212278b5b2b8)

We can verify that the bucket and file has been created in the AWS console

![image](https://github.com/user-attachments/assets/154c624d-8d9b-4162-be8a-f6c199eab45a)


### [3] Restore from S3

we Create a new program called `restorefromcloud.py` that reads the S3 bucket and writes the contents of the bucket within the appropriate directories. 
we create a new subdirectry called Restored and place this restorefrom cloud.py file in it.
```
import boto3
import os

BUCKET_NAME = '23803313-cloudstorage'  # Replace with your bucket name
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
 the code first creates a boto3 connection then we use list_objects_v2 to get the contents from our bucket.
then we check for object keys in the content returned byt he function and use that info to create the appropriate directory and download the files. 
we can confirm by checking the newly created restore directory in has the  copy of the files and the directories from the S3 bucket.

![image](https://github.com/user-attachments/assets/0a3f7e58-b258-4432-bc35-dfe6906fb90a)


### [4] Write information about files to DynamoDB

Install DynamoDB on your Linux environment

```
mkdir dynamodb
cd dynamodb
```

Install jre if not done

```
sudo apt-get install default-jre
wget https://s3-ap-northeast-1.amazonaws.com/dynamodb-local-tokyo/dynamodb_local_latest.tar.gz
```

You can use the following command to extract files from dynamodb_local_latest.tar.gz

```
tar -zxvf dynamodb_local_latest.tar.gz
```

After the extraction, run the command below

```
java -Djava.library.path=./DynamoDBLocal_lib -jar DynamoDBLocal.jar –sharedDb
```

Alternatively, you can use docker:
```
docker run -p 8000:8000 amazon/dynamodb-local -jar DynamoDBLocal.jar -inMemory -sharedDb
```
**Note**: Do not close the current window, open a new window to run the following Python script.

Write a Python script to create a table called `CloudFiles` on your local DynamoDB and the attributes for the table are:

```
        CloudFiles = {
            'userId',
            'fileName',
            'path',
            'lastUpdated',
	    'owner',
            'permissions'
            }
        )
```
`userId` is the partition key and `fileName` is the sort key. Regarding the creation, refer to this [page](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/dynamodb.html)

Then, you need to get the attributes above for each file of the S3 bucket and then write the attributes of each file into the created DynamoDB table. Regarding how to get the attributes for a file, refer to this [page](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/s3/client/get_object_acl.html)

**NOTE**: 

1) The table should have 2 items. One item corresponds to one file in the bucket and consists of the attributes above and their values.

2) Regarding the attribute `owner`, if you use a region in the table below, its value should be **owner's name**. Otherwise, its value should be **owner's ID**.

| Region | Region Name |
| --- | --- |
| US East (N. Virginia) | us-east-1 |
| Asia Pacific (Tokyo)	| ap-northeast-1 |
| Asia Pacific (Singapore) | ap-southeast-1 |
| Asia Pacific (Sydney)	| ap-southeast-2 |

```
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

![image](https://github.com/user-attachments/assets/d87a04bc-d51b-42b8-879c-295635aaad25)


### [5] Scan the table

Use AWS CLI command to scan the created DynamoDB table, and output what you've got. 
```
aws dynamodb delete-table --table-name CloudFiles --region ap-northeast-3
```
![image](https://github.com/user-attachments/assets/cc8ed6c6-b27c-458a-836b-8f147675c205)


### [6] Delete the table

Use AWS CLI command to delete the table.

**NOTE**: Delete the created S3 bucket from AWS console after the lab is done.
```
aws dynamodb delete-table --table-name CloudFiles --region ap-northeast-1
```
![image](https://github.com/user-attachments/assets/6c2f929e-8271-4a1c-a1e5-e3efa27dd285)

Lab Assessment:

A structured presentation (15%). A clear step-by-step with detailed descriptions (85%). 
