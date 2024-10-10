# Lab 8: Hyperparameter Tuning with AWS SageMaker

## Summary
In this lab, we explored how to perform hyperparameter tuning using AWS SageMaker to optimize a machine learning model. We set up and ran hyperparameter tuning jobs on AWS SageMaker, utilizing the XGBoost algorithm on the bank marketing dataset from the UCI Machine Learning Repository. The process involved installing necessary libraries, preparing the dataset, configuring the hyperparameter tuning job, and analyzing the results.

## Setting Up the Environment
### Install and run jupyter notebooks
First, we need to install Jupyter Notebook, which provides an interactive environment to write and execute Python code.
```
pip install notebook
python3 -m notebook
```
This will start the Jupyter Notebook server, and you can access it through your web browser.

### Install ipykernel
To ensure that Jupyter Notebook can properly handle Python environments and kernels, we install the `ipykernel` package.
```
pip install ipykernel
```
This package allows us to manage different Python kernels within Jupyter Notebook.

### Install Necessary Libraries in Jupyter Notebook
Within the Jupyter Notebook, we need to install the essential libraries required for this lab: SageMaker, pandas, and numpy.

In a Jupyter Notebook cell, run:
```python3
# Install SageMaker via jupyter notebook
!pip install sagemaker
# Install pandas and numpy jupyter notebook
!pip install pandas
!pip install numpy
```
These libraries are crucial for interacting with AWS SageMaker and handling data processing tasks

## AWS IAM Role Setup

In order to interact with AWS SageMaker, we need an IAM role with the necessary permissions. In the AWS Console, navigate to **IAM > Roles**, and find the existing `SageMakerRole`. This role should have the required permissions to run SageMaker jobs.

![image](https://github.com/user-attachments/assets/c6f3c855-0ab3-4a4a-8cec-39e313f33c14)

Note: If you don't have permissions to create a new role, you can proceed with an existing one.

## Preparing the AWS SageMaker Session
We set up a SageMaker session to communicate with the SageMaker service and define key variables like the AWS region, IAM role, and S3 bucket
```python3
import sagemaker
import boto3
import numpy as np
import pandas as pd
from time import gmtime, strftime
import os

# Initialize SageMaker client
smclient = boto3.Session().client("sagemaker")

# Initialize IAM client
iam = boto3.client('iam')

# Get SageMaker role ARN
sagemaker_role = iam.get_role(RoleName='SageMakerRole')['Role']['Arn']

# Specify AWS region
region = 'ap-northeast-3'  # Replace with your AWS region

# Define your student ID
student_id = "23803313"  # Replace with your student ID

# Define S3 bucket name
bucket = f'{student_id}-lab8'  # Format: <studentid>-lab8

# Define prefix for S3 paths
prefix = f"sagemaker/{student_id}-hpo-xgboost-dm"
```
**Explanation**
- We import the necessary libraries.
- `smclient` and `iam` are clients to interact with SageMaker and IAM services
- `sagemaker_role` retrieves the ARN of the SageMaker role.
- `region`, `student_id`, `bucket`, and `prefix` are variables we'll use throughout the lab.

## Create S3 Bucket
We need an S3 bucket to store the training and validation data for our SageMaker jobs.
```python3
# Initialize S3 resource
s3 = boto3.resource('s3')

# Attempt to create the S3 bucket
try:
    s3.create_bucket(Bucket=bucket, CreateBucketConfiguration={'LocationConstraint': region})
    print(f"S3 bucket '{bucket}' created.")
except s3.meta.client.exceptions.BucketAlreadyOwnedByYou:
    print(f"S3 bucket '{bucket}' already exists and is owned by you.")

# Create a folder (prefix) in the S3 bucket
s3.Bucket(bucket).put_object(Key=(prefix + '/'))
print(f"S3 prefix '{prefix}/' created.")
```
**Explanation**
- We initialize an S3 resource.
- We attempt to create a new S3 bucket with the specified name and region.
- If the bucket already exists and is owned by us, we catch the exception and proceed.
- We create a folder in the bucket using the specified prefix.
- 
![image](https://github.com/user-attachments/assets/3603c86b-bfab-4e19-a5d1-fab41410958e)

## Download and Load Dataset
We use the bank marketing dataset from the UCI Machine Learning Repository.
```python3
# Download the dataset
!wget -N https://archive.ics.uci.edu/ml/machine-learning-databases/00222/bank-additional.zip

# Unzip the dataset
!unzip -o bank-additional.zip

# Load the dataset into a Pandas DataFrame
data = pd.read_csv("./bank-additional/bank-additional-full.csv", sep=";")

# Display all columns and rows
pd.set_option("display.max_columns", None)
pd.set_option("display.max_rows", None)

# Preview the first few rows
data.head()
```
**Explanation**
- We use `wget` to download the dataset zip file
- We unzip the file to extract the CSV data.
- We read the CSV file into a Pandas DataFrame.
- We adjust Pandas settings to display all columns and rows for better visibility.
- We display the first few rows to get an initial look at the data.
  
![image](https://github.com/user-attachments/assets/df8a9261-17be-42aa-813b-8a15fc0304ee)

## Data Preprocessing
### Handling Categorical Variables
We identify categorical and numeric variables and process the data by adding new indicator columns and converting categorical variables into dummy variables for modeling.
```python3
# Identify categorical variables
categorical_vars = data.select_dtypes(include=['object']).columns.tolist()
print("Categorical Variables:", categorical_vars[:4])  # Display first four

# Identify numerical variables
numerical_vars = data.select_dtypes(include=[np.number]).columns.tolist()
print("Numerical Variables:", numerical_vars[:4])  # Display first four
```
**Explanation**
- We use `select_dtypes` to select columns of specific data types
- `include=['object']` selects columns with string data types, which are our categorical variables.
- `include=[np.number]` selects columns with numerical data types.
  
![image](https://github.com/user-attachments/assets/7956c9f3-ea82-4aa2-b291-61b6d529ca2d)

### Adding Indicator Variables
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
**Explanation**
- `pdays` equal to 999 indicates that the client was not previously contacted; we create a binary variable `no_previous_contact`.
- We identify clients who are students, retired, or unemployed and create a binary variable `not_working`.
- `pd.get_dummies(data)` automatically converts all categorical variables into one-hot encoded variables
- This process creates new binary columns for each category in the categorical variables
  
![image](https://github.com/user-attachments/assets/406317e7-6758-43ef-b499-fd2fdce0ceb7)

### Removing Unnecessary Variables
We also removed economic variables and checked for any non-numeric values, converting boolean columns to integers as necessary.

```python3
model_data = model_data.drop(
    ["duration", "emp.var.rate", "cons.price.idx", "cons.conf.idx", "euribor3m", "nr.employed"],
    axis=1,
)
model_data.head()
```
**Explanation**
- These variables are removed because they may not be available in a real-world scenario when making predictions, or they might not be relevant for our model.

![image](https://github.com/user-attachments/assets/f99d9941-3acd-480b-9d33-4aae5414262b)

### Ensuring All Data is Numeric

We ensure that all variables are numeric, converting any non-numeric columns.
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
**Explanation**
- We identify any non-numeric columns.
- We convert boolean columns to integers (True -> 1, False -> 0).
- For object columns, we map 'yes' and 'no' to 1 and 0, respectively.
- We verify that all columns are now numeric.
  
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
**Explanation**
- We use `np.split` to split the data into 70% training, 20% validation, and 10% test sets.
- `model_data.sample(frac=1, random_state=1729)` shuffles the data before splitting.
- We specify indices to split the data appropriately.
- We place the target variable y_yes as the first column
- We drop the y_no column since y_yes and y_no are complements
- We save the datasets to CSV files without headers and indices, as required by XGBoost in CSV format
  
## Upload Data to S3
We upload the processed training and validation datasets to the S3 bucket for use in SageMaker training jobs.

```python3
# Upload training data to S3
boto3.Session().resource("s3").Bucket(bucket).Object(
    os.path.join(prefix, "train/train.csv")
).upload_file("train.csv")

# Upload validation data to S3
boto3.Session().resource("s3").Bucket(bucket).Object(
    os.path.join(prefix, "validation/validation.csv")
).upload_file("validation.csv")
```
**Explanation**
- We use the `upload_file` method to upload the local CSV files to the specified S3 paths.
- The data is stored in the `train` and `validation` folders within the S3 bucket.

## Configuring the Hyperparameter Tuning Job
We define the configuration for the hyperparameter tuning job, including parameter ranges, resource limits, and tuning strategy.
```python3
from time import gmtime, strftime, sleep

# Define a unique tuning job name
tuning_job_name = f"{student_id}-xgboost-tuningjob-02"

print(tuning_job_name)

# Define the hyperparameter ranges
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
**Explanation**
- We create a unique tuning job name using the current time.
- We define the hyperparameters to tune:
  - `eta`: Learning rate.
  - `min_child_weight`: Minimum sum of instance weight needed in a child.
  - `alpha`: L1 regularization term on weights.
  - `max_depth`: Maximum depth of a tree.
- We set the resource limits:
  - `MaxNumberOfTrainingJobs`: Total number of training jobs to run.
  - `MaxParallelTrainingJobs`: Number of training jobs to run in parallel.
- We specify the optimization strategy and objective metric.
  
## Specifying the XGBoost Algorithm 
We specify the details of the training job definition, including the algorithm and data sources
```python3
from sagemaker.image_uris import retrieve

# Retrieve the XGBoost image URI
training_image = retrieve(framework="xgboost", region=region, version="latest")

# Define S3 input paths for training and validation data
s3_input_train = f"s3://{bucket}/{prefix}/train"
s3_input_validation = f"s3://{bucket}/{prefix}/validation"

# Define the training job parameters
training_job_definition = {
    "AlgorithmSpecification": {
        "TrainingImage": training_image,
        "TrainingInputMode": "File"
    },
    "InputDataConfig": [
        {
            "ChannelName": "train",
            "DataSource": {
                "S3DataSource": {
                    "S3Uri": s3_input_train,
                    "S3DataType": "S3Prefix",
                    "S3DataDistributionType": "FullyReplicated"
                }
            },
            "ContentType": "csv",
            "CompressionType": "None"
        },
        {
            "ChannelName": "validation",
            "DataSource": {
                "S3DataSource": {
                    "S3Uri": s3_input_validation,
                    "S3DataType": "S3Prefix",
                    "S3DataDistributionType": "FullyReplicated"
                }
            },
            "ContentType": "csv",
            "CompressionType": "None"
        }
    ],
    "OutputDataConfig": {
        "S3OutputPath": f"s3://{bucket}/{prefix}/output"
    },
    "ResourceConfig": {
        "InstanceType": "ml.m5.xlarge",
        "InstanceCount": 1,
        "VolumeSizeInGB": 10
    },
    "RoleArn": sagemaker_role,
    "StaticHyperParameters": {
        "objective": "binary:logistic",
        "num_round": "100",
        "eval_metric": "auc",
        "verbosity": "1"
    },
    "StoppingCondition": {
        "MaxRuntimeInSeconds": 3600
    },
}
```
**Explanation**
- We retrieve the XGBoost image URI for the specified region and version.
- We define the input data configurations for the training and validation datasets.
- We specify the output data configuration, resource configuration, and static hyperparameters.
- The static hyperparameters include:
  - `objective`: The learning task and the corresponding learning objective.
  - `num_round`: Number of boosting rounds.
  - `eval_metric`: Evaluation metric.
- We set the stopping condition to limit the maximum runtime.
  
## Launching the Hyperparameter Tuning Job
We launch the hyperparameter tuning job using the configurations defined.
```pyhton3
# Launch the hyperparameter tuning job
smclient.create_hyper_parameter_tuning_job(
    HyperParameterTuningJobName=tuning_job_name,
    HyperParameterTuningJobConfig=tuning_job_config,
    TrainingJobDefinition=training_job_definition,
)

print(f"Hyperparameter tuning job '{tuning_job_name}' launched successfully.")
```
**Explanation**
- We call `create_hyper_parameter_tuning_job` with the tuning job name, configuration, and training job definition.

![image](https://github.com/user-attachments/assets/9b1298d8-c420-48c6-ad91-2180f160188d)

## Verifying Job Completion
I verified the successful completion of the hyperparameter tuning job in the SageMaker console.

![image](https://github.com/user-attachments/assets/16e04331-f559-43c4-b898-1911b96c1cbd)

## Check Tuning Job Output
I reviewed the output of the tuning job to evaluate its performance and ensure the optimal hyperparameters were identified.

![image](https://github.com/user-attachments/assets/16a98776-ea69-4802-8611-1787a51396f5)
