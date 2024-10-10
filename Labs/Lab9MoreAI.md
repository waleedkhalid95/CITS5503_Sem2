# Lab 9: Working with AWS Comprehend and Rekognition
## Summary

I worked with AWS Comprehend and Rekognition services using boto3 in a Jupyter Notebook environment. For AWS Comprehend, I explored natural language processing tasks such as detecting the language of a text, analyzing sentiment, detecting entities, key phrases, and syntax using various example texts in different languages like English, Spanish, French, and Italian. For AWS Rekognition, I created an S3 bucket to store images and tested image processing tasks such as label recognition, facial analysis, and text extraction. I encountered some format compatibility issues with images but resolved them by converting them to JPEG, and I also handled exceptions to troubleshoot errors in image processing tasks.

## AWS Comprehend

AWS Comprehend is a natural language processing (NLP) service that uses machine learning to analyze text. With Comprehend, you can perform tasks such as detecting the dominant language in a text, analyzing sentiment, detecting named entities, and extracting key phrases.

Since AWS Comprehend is not available in the `ap-northeast-3` region, I used the `ap-south-1` region for this lab.

### Detecting Dominant Language
To get started with AWS Comprehend, I wrote a script that detects the language of a given text. I used an example text about the French Revolution and checked the predicted language using the `detect_dominant_language` function.
```python
import boto3
client = boto3.client('comprehend')

# Detect Entities
response = client.detect_dominant_language(
    Text="The French Revolution was a period of social and political upheaval in France and its colonies beginning in 1789 and ending in 1799.",
)

print(response['Languages'])
```

By executing the code above, we will get something like this:
```
[{'LanguageCode': 'en', 'Score': 0.9961233139038086}]
```
This returned a response indicating that the text was in English (`'en'`), with a confidence score of over 99%. 

![image](https://github.com/user-attachments/assets/4a402681-05f6-46a1-9db4-8fb7f4e4c3e4)


### Testing Language Detection in Different Languages

I tested AWS Comprehend with texts in different languages (English, Spanish, French, and Italian) to detect the dominant language. Here’s how I implemented this:

```python3
import boto3

def detect_language(text):
    client = boto3.client('comprehend', region_name='ap-south-1')
    response = client.detect_dominant_language(Text=text)
    lang_code = response['Languages'][0]['LanguageCode']
    confidence = response['Languages'][0]['Score'] * 100
    languages = {
        'en': 'English',
        'es': 'Spanish',
        'fr': 'French',
        'it': 'Italian'
    }
    predicted_language = languages.get(lang_code, lang_code)
    print(f"{predicted_language} detected with {int(confidence)}% confidence")

# Test with different languages
texts = [
    "The French Revolution was a period of social and political upheaval in France.",
    "El Quijote es la obra más conocida de Miguel de Cervantes.",
    "Moi je n'étais rien Et voilà qu'aujourd'hui Je suis le gardien.",
    "L'amor che move il sole e l'altre stelle."
]
for text in texts:
    detect_language(text)
```

![image](https://github.com/user-attachments/assets/b00a2547-c150-4af2-8508-17af96cee384)

This correctly identified the languages as English, Spanish, French, and Italian, respectively.

### Sentiment Analysis 

Next, I used AWS Comprehend’s sentiment analysis capability to determine whether a text’s sentiment was positive, negative, neutral, or mixed.

```python 3
def analyze_sentiment(text):
    client = boto3.client('comprehend')
    response = client.detect_sentiment(Text=text, LanguageCode='en')
    return f"Sentiment: {response['Sentiment']}"
for text in texts:
    print(analyze_sentiment(text))
```

This provided sentiment classifications for each of the English texts.

![image](https://github.com/user-attachments/assets/9367425e-535d-4a4b-9f0a-e86f3e718c81)

### Entity Detection

Entity detection identifies named entities like people, places, organizations, and dates within the text. I applied entity detection to the texts using AWS Comprehend
```python3
def detect_entities(text):
    client = boto3.client('comprehend')
    response = client.detect_entities(Text=text, LanguageCode='en')
    return [(entity['Text'], entity['Type'], int(entity['Score'] * 100)) for entity in response['Entities']]
for text in texts:
    print(detect_entities(text))
```

This function identified entities such as names, dates, and places in the English text.

![image](https://github.com/user-attachments/assets/1c0ef251-0962-4db3-aa05-ae12ba180759)

`Entities` are elements such as people, places, organizations, and dates found within a text.

### Key Phrase Detection

I used AWS Comprehend to detect key phrases within a text. Key phrases help identify the most important information in a sentence.
```python3
def detect_key_phrases(text):
    client = boto3.client('comprehend')
    response = client.detect_key_phrases(Text=text, LanguageCode='en')
    return [(phrase['Text'], int(phrase['Score'] * 100)) for phrase in response['KeyPhrases']]
for text in texts:
    print(detect_key_phrases(text))
```

![image](https://github.com/user-attachments/assets/fd5f7ad6-1526-48d3-92b6-59e2cef28b8f)

`Key phrases` are the important words or phrases in a text that convey the main ideas.

### Syntax Analysis

Syntax analysis identifies the grammatical structure of a sentence, tagging each word with its part of speech (noun, verb, adjective, etc.).

```pyhton3
def detect_syntax(text):
    client = boto3.client('comprehend')
    response = client.detect_syntax(Text=text, LanguageCode='en')
    return [(token['Text'], token['PartOfSpeech']['Tag']) for token in response['SyntaxTokens']]
for text in texts:
    print(detect_syntax(text))
```

![image](https://github.com/user-attachments/assets/d3b333c7-f824-4640-9f8e-b20ae1aff266)

`Syntax` refers to the arrangement of words in a sentence and the grammatical structure used.

## AWS Rekognition

AWS Rekognition is a service that analyzes images using machine learning. It can detect objects, people, text, and even facial features in images.

### Add images

Create a python script: create an S3 bucket named as 23803313-lab9 in the region you are mapped to. Add the 4 following images into the bucket:

1. Add an image of an urban setting (named as urban.jpg).

2. Add an image of a person on the beach (named as beach.jpg).

3. Add an image with people showing their faces (named as faces.jpg).

4. Add an image with texts (named as text.jpg). we will have to change the format to jpeg due to compatibility issues.

### Set Up S3 Bucket and Upload Images

I created an S3 bucket to store images and uploaded four images to test various Rekognition features: label recognition, facial analysis, and text detection.

```python3
import boto3

# Initialize S3 resource
s3 = boto3.resource('s3')
bucket_name = '23803313-lab9'
region = 'ap-south-1'  # Replace with your AWS region

# Create S3 bucket
try:
    s3.create_bucket(Bucket=bucket_name, CreateBucketConfiguration={'LocationConstraint': region})
    print(f"S3 bucket '{bucket_name}' created.")
except s3.meta.client.exceptions.BucketAlreadyOwnedByYou:
    print(f"S3 bucket '{bucket_name}' already exists.")

# Upload images to S3
images = ['urban.jpg', 'beach.jpg', 'faces.jpg', 'text.jpeg']
for image in images:
    s3.Bucket(bucket_name).upload_file(image, image)
    print(f"Uploaded {image} to {bucket_name}")
```

![image](https://github.com/user-attachments/assets/9ddff8f5-aca7-41db-92ef-b49c829e3a21)
### Label Detection, Facial Analysis, Text Detection    
```python3
# Initialize Rekognition client
rekognition = boto3.client('rekognition')

# Function to detect labels
def detect_labels(image_name, bucket_name):
    response = rekognition.detect_labels(Image={'S3Object': {'Bucket': bucket_name, 'Name': image_name}}, MaxLabels=10)
    return [(label['Name'], int(label['Confidence'])) for label in response['Labels']]

# Function to detect faces
def detect_faces(image_name, bucket_name):
    response = rekognition.detect_faces(Image={'S3Object': {'Bucket': bucket_name, 'Name': image_name}}, Attributes=['ALL'])
    return [(int(face['Confidence']), [emotion['Type'] for emotion in face['Emotions']]) for face in response['FaceDetails']]

# Function to detect text
def detect_text(image_name, bucket_name):
    response = rekognition.detect_text(Image={'S3Object': {'Bucket': bucket_name, 'Name': image_name}})
    return [(text['DetectedText'], int(text['Confidence'])) for text in response['TextDetections']]
images = ['urban.jpg', 'beach.jpg', 'faces.jpg', 'text.jpeg']

for image in images:
    print(f"Labels in {image}: {detect_labels(image, bucket_name)}")
    if image == 'faces.jpg':
        print(f"Facial analysis for {image}: {detect_faces(image, bucket_name)}")
    if image == 'text.jpeg':
        print(f"Text detected in {image}: {detect_text(image, bucket_name)}")
```

![image](https://github.com/user-attachments/assets/14c534c7-1f24-43b6-aed3-04ad37bff392)
