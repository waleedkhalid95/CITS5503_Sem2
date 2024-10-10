# Lab 8: Hyperparameter Tuning with AWS SageMaker

## Summary
In this lab, I set up and ran hyperparameter tuning jobs on AWS SageMaker. I started by installing Jupyter Notebook and setting up necessary libraries like SageMaker, pandas, and numpy. Then, I prepared an AWS SageMaker session, using an existing SageMaker role, and created an S3 bucket to store training and validation data. After downloading the bank marketing dataset from UCI's ML Repository, I processed the data, converted categorical variables into dummy variables, and split it into training, validation, and test datasets. I ensured all data was numeric by converting non-numeric values like booleans into integers. The data was then uploaded to S3 for use in the tuning job. I configured a hyperparameter tuning job for the XGBoost algorithm, set up training job definitions, and launched the tuning job. Finally, I verified the job's completion by checking the SageMaker console and reviewing the job output.

## Install and run jupyter notebooks
To run this lab, I first installed Jupyter Notebook to create an environment where I could execute my Python scripts for hyperparameter tuning.
```
pip install notebook
python3 -m notebook
```

## Install ipykernel
This is needed to ensure that Jupyter Notebooks can properly handle Python environments.
```
pip install ipykernel
```

## AWS IAM Role Setup

In the AWS Console, I navigated to IAM -> Role and found the existing `SageMakerRole` which has the necessary permissions for this lab. Since I didn’t have permission to create a new role, I proceeded with this existing role.

![image](https://github.com/user-attachments/assets/c6f3c855-0ab3-4a4a-8cec-39e313f33c14)

## Install Necessary Libraries in Jupyter Notebook
I installed the essential libraries: SageMaker, pandas, and numpy to work with the datasets and interface with AWS.
```python3
# Install SageMaker via jupyter notebook
!pip install sagemaker
# Install pandas and numpy jupyter notebook
!pip install pandas
!pip install numpy
```
## Prepare AWS SageMaker Session
I set up an AWS SageMaker session to communicate with the SageMaker service. This involved initializing the SageMaker role, region, and bucket name for storing the datasets.
```python3
import sagemaker
import boto3

import numpy as np  # For matrix operations and numerical processing
import pandas as pd  # For munging tabular data
from time import gmtime, strftime
import os

smclient = boto3.Session().client("sagemaker")
iam = boto3.client('iam')
sagemaker_role = iam.get_role(RoleName='SageMakerRole')['Role']['Arn']
region = 'ap-northeast-3' # use the region you are mapped to 
student_id = "23803313" # use your student id 
bucket = '23803313-lab8' # use <studentid-lab8> as your bucket name
prefix = f"sagemaker/{student_id}-hpo-xgboost-dm"
```
## Create S3 Bucket
Next, I created an S3 bucket to store training and validation data. If the bucket already existed, the script handled that gracefully
```python3
# Create an S3 bucket using the bucket variable above. The bucket creation is done using the region variable above.
# Create an object into the bucket. The object is a folder and its name is the prefix variable above. 
# Create S3 Bucket
s3 = boto3.resource('s3')
try:
    s3.create_bucket(Bucket=bucket, CreateBucketConfiguration={'LocationConstraint': region})
    print(f"S3 bucket '{bucket}' created.")
except s3.meta.client.exceptions.BucketAlreadyOwnedByYou:
    print(f"S3 bucket '{bucket}' already exists and is owned by you.")

# Create a folder (prefix) in the S3 bucket
s3.Bucket(bucket).put_object(Key=(prefix + '/'))
print(f"S3 prefix '{prefix}/' created.")
```

![image](https://github.com/user-attachments/assets/3603c86b-bfab-4e19-a5d1-fab41410958e)

## Download and Load Dataset
I downloaded the bank marketing dataset from UCI's ML Repository, then loaded it into a Pandas DataFrame for further processing.
```python3
!wget -N https://archive.ics.uci.edu/ml/machine-learning-databases/00222/bank-additional.zip
!unzip -o bank-additional.zip
data = pd.read_csv("./bank-additional/bank-additional-full.csv", sep=";")
pd.set_option("display.max_columns", 500)  # Make sure we can see all of the columns
pd.set_option("display.max_rows", 50)  # Keep the output on one page
data.head()
```

![image](https://github.com/user-attachments/assets/df8a9261-17be-42aa-813b-8a15fc0304ee)

## Data Preprocessing
I identified categorical and numeric variables and processed the data by adding new indicator columns and converting categorical variables into dummy variables for modeling.
```python3
# Identify categorical variables
categorical_vars = data.select_dtypes(include=['object']).columns.tolist()
print("Categorical Variables:", categorical_vars[:4])  # Display first four

# Identify numerical variables
numerical_vars = data.select_dtypes(include=[np.number]).columns.tolist()
print("Numerical Variables:", numerical_vars[:4])  # Display first four
```

![image](https://github.com/user-attachments/assets/7956c9f3-ea82-4aa2-b291-61b6d529ca2d)

Process the data by adding two new indicator columns and then expands categorical columns into binary dummy columns for modelling purposes
```python3
# Add indicator columns
data["no_previous_contact"] = np.where(data["pdays"] == 999, 1, 0)
data["not_working"] = np.where(data["job"].isin(["student", "retired", "unemployed"]), 1, 0)

# Convert categorical variables to dummy/indicator variables
model_data = pd.get_dummies(data)

# Display the first few rows of the processed data
model_data.head()
```

![image](https://github.com/user-attachments/assets/406317e7-6758-43ef-b499-fd2fdce0ceb7)

I also removed economic variables and checked for any non-numeric values, converting boolean columns to integers as necessary.

```python3
model_data = model_data.drop(
    ["duration", "emp.var.rate", "cons.price.idx", "cons.conf.idx", "euribor3m", "nr.employed"],
    axis=1,
)
model_data.head()
```

![image](https://github.com/user-attachments/assets/f99d9941-3acd-480b-9d33-4aae5414262b)

then we want to chech for non numeric columns and convert boolean columns to integer
```python3
# Check for non-numeric columns
non_numeric_columns = model_data.select_dtypes(include=['object', 'bool']).columns.tolist()
print("Non-numeric columns:", non_numeric_columns)

# Convert boolean columns to integers if any
for col in non_numeric_columns:
    if model_data[col].dtype == 'bool':
        model_data[col] = model_data[col].astype(int)
    elif model_data[col].dtype == 'object':
        # Convert 'True'/'False' strings to 1/0
        model_data[col] = model_data[col].map({'True': 1, 'False': 0})

# Verify all columns are now numeric
non_numeric_columns = model_data.select_dtypes(include=['object', 'bool']).columns.tolist()
print("Non-numeric columns after conversion:", non_numeric_columns)
```

![image](https://github.com/user-attachments/assets/19786a9d-adae-491e-a205-8f3765e656c7)

## Split Data into Training, Validation, and Test Sets
We split the dataset into training (70%), validation (20%), and test (10%) datasets and convert the datasets to an appropriate format. We will use the training and validation datasets during training. Test dataset will be used to evaluate model performance after it is deployed to an endpoint.

Amazon SageMaker's XGBoost algorithm expects data in the libSVM or CSV data format. In this lab, we use the CSV format. Note that the first column must be the target variable and the CSV should not include headers. Also, notice that although repetitive it’s easier to do this after the train|validation|test split rather than before. This avoids any misalignment issues due to random reordering.
```python3
train_data, validation_data, test_data = np.split(
    model_data.sample(frac=1, random_state=1729),
    [int(0.7 * len(model_data)), int(0.9 * len(model_data))],
)

pd.concat([train_data["y_yes"], train_data.drop(["y_no", "y_yes"], axis=1)], axis=1).to_csv(
    "train.csv", index=False, header=False
)
pd.concat(
    [validation_data["y_yes"], validation_data.drop(["y_no", "y_yes"], axis=1)], axis=1
).to_csv("validation.csv", index=False, header=False)
pd.concat([test_data["y_yes"], test_data.drop(["y_no", "y_yes"], axis=1)], axis=1).to_csv(
    "test.csv", index=False, header=False
)
```

## Upload Data to S3
I uploaded the processed training and validation datasets to the S3 bucket for use in SageMaker training jobs.

```python3
boto3.Session().resource("s3").Bucket(bucket).Object(
    os.path.join(prefix, "train/train.csv")
).upload_file("train.csv")
boto3.Session().resource("s3").Bucket(bucket).Object(
    os.path.join(prefix, "validation/validation.csv")
).upload_file("validation.csv")
```
## Hyperparameter Tuning Configuration
I defined the configuration for the hyperparameter tuning job, including parameter ranges, resource limits, and tuning strategy.
```python3
from time import gmtime, strftime, sleep

# Names have to be unique. You will get an error if you reuse the same name
tuning_job_name = f"{student_id}-xgboost-tuningjob-02"

print(tuning_job_name)

tuning_job_config = {
    "ParameterRanges": {
        "CategoricalParameterRanges": [],
        "ContinuousParameterRanges": [
            {
                "MaxValue": "1",
                "MinValue": "0",
                "Name": "eta",
            },
            {
                "MaxValue": "10",
                "MinValue": "1",
                "Name": "min_child_weight",
            },
            {
                "MaxValue": "2",
                "MinValue": "0",
                "Name": "alpha",
            },
        ],
        "IntegerParameterRanges": [
            {
                "MaxValue": "10",
                "MinValue": "1",
                "Name": "max_depth",
            }
        ],
    },
    "ResourceLimits": {"MaxNumberOfTrainingJobs": 2, "MaxParallelTrainingJobs": 2},
    "Strategy": "Bayesian",
    "HyperParameterTuningJobObjective": {"MetricName": "validation:auc", "Type": "Maximize"},
}
```
## Specify the XGBoost algorithm 
I retrieved the latest XGBoost image from SageMaker and set up the paths for the training and validation data.
```python3
from sagemaker.image_uris import retrieve
# Use XGBoost algorithm for training
training_image = retrieve(framework="xgboost", region=region, version="latest")

s3_input_train = "s3://{}/{}/train".format(bucket, prefix)
s3_input_validation = "s3://{}/{}/validation/".format(bucket, prefix)

training_job_definition = {
    "AlgorithmSpecification": {"TrainingImage": training_image, "TrainingInputMode": "File"},
    "InputDataConfig": [
        {
            "ChannelName": "train",
            "CompressionType": "None",
            "ContentType": "csv",
            "DataSource": {
                "S3DataSource": {
                    "S3DataDistributionType": "FullyReplicated",
                    "S3DataType": "S3Prefix",
                    "S3Uri": s3_input_train,
                }
            },
        },
        {
            "ChannelName": "validation",
            "CompressionType": "None",
            "ContentType": "csv",
            "DataSource": {
                "S3DataSource": {
                    "S3DataDistributionType": "FullyReplicated",
                    "S3DataType": "S3Prefix",
                    "S3Uri": s3_input_validation,
                }
            },
        },
    ],
    "OutputDataConfig": {"S3OutputPath": "s3://{}/{}/output".format(bucket, prefix)},
    "ResourceConfig": {"InstanceCount": 1, "InstanceType": "ml.m5.xlarge", "VolumeSizeInGB": 10},
    "RoleArn": sagemaker_role,
    "StaticHyperParameters": {
        "eval_metric": "auc",
        "num_round": "1",
        "objective": "binary:logistic",
        "rate_drop": "0.3",
        "tweedie_variance_power": "1.4",
    },
    "StoppingCondition": {"MaxRuntimeInSeconds": 43200},
}
```
## Launch Hyperparameter Tuning Job
I launched the hyperparameter tuning job using the configurations set up in the previous step.
```pyhton3
#Launch Hyperparameter Tuning Job
smclient.create_hyper_parameter_tuning_job(
    HyperParameterTuningJobName=tuning_job_name,
    HyperParameterTuningJobConfig=tuning_job_config,
    TrainingJobDefinition=training_job_definition,
)
```

![image](https://github.com/user-attachments/assets/9b1298d8-c420-48c6-ad91-2180f160188d)

the we head over to aws console and see the tuning job in sagemaker
![image](https://github.com/user-attachments/assets/f0f1d6a5-2d90-4fee-98f4-f8e0db2705cf)

## Verify Job Completion
I verified the successful completion of the hyperparameter tuning job in the SageMaker console.

![image](https://github.com/user-attachments/assets/16e04331-f559-43c4-b898-1911b96c1cbd)

## Check Tuning Job Output
I reviewed the output of the tuning job to evaluate its performance and ensure the optimal hyperparameters were identified.

![image](https://github.com/user-attachments/assets/16a98776-ea69-4802-8611-1787a51396f5)
