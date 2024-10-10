# Lab 9: Working with AWS Comprehend and Rekognition
## Summary

In this lab, we explored the capabilities of **AWS Comprehend** and **AWS Rekognition** services using Boto3 in a Jupyter Notebook environment.
- **AWS Comprehend:** We performed various natural language processing (NLP) tasks such as detecting the dominant language of a text, analyzing sentiment, detecting entities, extracting key phrases, and analyzing syntax using example texts in different languages like English, Spanish, French, and Italian.
- **AWS Rekognition:** We created an S3 bucket to store images and utilized image processing tasks such as label detection, facial analysis, and text extraction. We encountered some image format compatibility issues, which we resolved by converting images to JPEG format, and we also handled exceptions to troubleshoot errors during image processing tasks.

## AWS Comprehend

**AWS Comprehend** is a natural language processing service that uses machine learning to discover insights from text. It can perform tasks such as detecting the dominant language, sentiment analysis, entity recognition, key phrase extraction, and syntax analysis.

AWS Comprehend is not available in the `ap-northeast-3` region, so we used the `ap-south-1` region for this lab.

### Setting Up AWS Comprehend
First, we need to import the Boto3 library and set up the AWS Comprehend client.
```python3
import boto3

# Create a Comprehend client in the 'ap-south-1' region
client = boto3.client('comprehend', region_name='ap-south-1')
```
**Explanation**
- We import the `boto3` library, which is AWS's SDK for Python.
- We create a client for AWS Comprehend, specifying the `region_name` as `ap-south-1`.
  
### Detecting Dominant Language
To get started with AWS Comprehend, we wrote a script to detect the dominant language of a given text. We used an example text about the French Revolution and checked the predicted language using the `detect_dominant_language` function.

```python
# Detect the dominant language of the text
response = client.detect_dominant_language(
    Text="The French Revolution was a period of social and political upheaval in France and its colonies beginning in 1789 and ending in 1799.",
)

# Print the detected languages and their confidence scores
print(response['Languages'])
```
**Explanation**
- We call the `detect_dominant_language` method with our sample text.
- The method returns a list of detected languages with their language codes and confidence scores.

By executing the code above, AWS Comprehend returns a response indicating that the text is in English (`'en'`), with a confidence score of over 99%. 

![image](https://github.com/user-attachments/assets/4a402681-05f6-46a1-9db4-8fb7f4e4c3e4)

### Testing Language Detection in Different Languages

We tested AWS Comprehend with texts in different languages (English, Spanish, French, and Italian) to detect the dominant language. Here's how we implemented this:

```python3
import boto3

def detect_language(text):
    # Create a Comprehend client
    client = boto3.client('comprehend', region_name='ap-south-1')
    # Detect the dominant language
    response = client.detect_dominant_language(Text=text)
    # Extract the language code and confidence score
    lang_code = response['Languages'][0]['LanguageCode']
    confidence = response['Languages'][0]['Score'] * 100
    # Map language codes to language names
    languages = {
        'en': 'English',
        'es': 'Spanish',
        'fr': 'French',
        'it': 'Italian'
    }
    predicted_language = languages.get(lang_code, lang_code)
    # Print the detected language and confidence
    print(f"{predicted_language} detected with {int(confidence)}% confidence")

# Test texts in different languages
texts = [
    "The French Revolution was a period of social and political upheaval in France.",
    "El Quijote es la obra más conocida de Miguel de Cervantes.",
    "Moi je n'étais rien Et voilà qu'aujourd'hui Je suis le gardien.",
    "L'amor che move il sole e l'altre stelle."
]

# Detect language for each text
for text in texts:
    detect_language(text)
```
**Explanation**
- We define a function `detect_language` that takes a text input.
- Within the function, we create a Comprehend client.
- We call `detect_dominant_language` with the input text.
- We use `client.detect_dominant_language` to get the language code and confidence score.
- We map the language codes to human-readable language names.
- We print the text, detected language, language code, and confidence percentage.
- The function is tested on four sample texts in different languages.

![image](https://github.com/user-attachments/assets/b00a2547-c150-4af2-8508-17af96cee384)

This correctly identified the languages as English, Spanish, French, and Italian, respectively.

### Sentiment Analysis 

Next, we used AWS Comprehend's sentiment analysis capability to determine whether a text's sentiment is positive, negative, neutral, or mixed.

```python 3
def analyze_sentiment(text):
    # Create a Comprehend client
    client = boto3.client('comprehend')
    # Analyze the sentiment of the text
    response = client.detect_sentiment(Text=text, LanguageCode='en')
    # Return the detected sentiment
    return f"Sentiment: {response['Sentiment']}"
for text in texts:
    print(analyze_sentiment(text))
```
**Explanation:**
- Define a function `analyze_sentiment` that takes a text string as input.
- Create a Comprehend client.
- Call `detect_sentiment` with the input text and specify the language code as `'en'` (English).
- We extract the sentiment from the response.
- We return the sentiment as a string.
- We use a list of English texts to test sentiment analysis.
- We print the sentiment analysis results.

AWS Comprehend's sentiment analysis currently supports English, Spanish, French, German, Italian, Portuguese, and Japanese. Therefore, we ensure that the texts used for sentiment analysis are in supported languages.

![image](https://github.com/user-attachments/assets/9367425e-535d-4a4b-9f0a-e86f3e718c81)

This provided sentiment classifications for each of the English texts.

### Entity Detection
`Entities` are specific items of information in a text that represent real-world objects or concepts like people, places, organizations, dates, quantities, and other tangible things mentioned in a sentence.
Entity detection identifies named entities like people, places, organizations, dates, and quantities within the text. We applied entity detection to the texts using AWS Comprehend.
```python3
def detect_entities(text):
    client = boto3.client('comprehend')
    response = client.detect_entities(Text=text, LanguageCode='en')
    return [(entity['Text'], entity['Type'], int(entity['Score'] * 100)) for entity in response['Entities']]
for text in texts:
    print(detect_entities(text))
```
**Explanation**
- Define a function detect_entities that takes a text string as input.
- Create a Comprehend client.
- Call `detect_entities` with the input text and specify the language code as `'en'`.
- We extract the list of entities from the response.
- We return a list of tuples containing the entity text, type, and confidence score.
- We use an example text about Amazon Web Services for entity detection.
- We print each detected entity with its type and confidence score.

This function identified entities such as names, dates, and places in the English text.

![image](https://github.com/user-attachments/assets/1c0ef251-0962-4db3-aa05-ae12ba180759)

### Key Phrase Detection
`Key phrases` are the main ideas or significant expressions in a text that capture the meaning of what is being communicated. They are words or combinations of words that are meaningful or relevant.
We used AWS Comprehend to detect key phrases within a text. Key phrases help identify the most important information in a sentence.
```python3
def detect_key_phrases(text):
    client = boto3.client('comprehend')
    response = client.detect_key_phrases(Text=text, LanguageCode='en')
    return [(phrase['Text'], int(phrase['Score'] * 100)) for phrase in response['KeyPhrases']]
for text in texts:
    print(detect_key_phrases(text))
```
**Explanation:**
- Define a function `detect_key_phrases` that takes a text string as input.
- Create a Comprehend client.
- Call `detect_key_phrases` with the input text and specify the language code as `'en'`.
- Extract the list of key phrases from the response.
- Return a list of tuples containing the key phrase text and confidence score.
- We use an example text about machine learning for key phrase detection.
- We print each detected key phrase with its confidence score.
  
![image](https://github.com/user-attachments/assets/fd5f7ad6-1526-48d3-92b6-59e2cef28b8f)

### Syntax Analysis
`Syntax` refer to the grammatical structure of sentences—the way words are arranged and connected to convey meaning. Syntax analysis involves identifying the part of speech for each word in a sentence, such as nouns, verbs, adjectives, and so on.

```pyhton3
def detect_syntax(text):
    client = boto3.client('comprehend')
    response = client.detect_syntax(Text=text, LanguageCode='en')
    return [(token['Text'], token['PartOfSpeech']['Tag']) for token in response['SyntaxTokens']]
for text in texts:
    print(detect_syntax(text))
```
**Explanation:**
- Define a function `detect_syntax` that takes a text string as input.
- Create a Comprehend client.
- Call `detect_syntax` with the input text and specify the language code as `'en'`.
- Extract the list of syntax tokens from the response.
- We return a list of tuples containing the token text and its part of speech tag.
- We use an example text about natural language processing for syntax analysis.
- We print each token with its part of speech.

![image](https://github.com/user-attachments/assets/d3b333c7-f824-4640-9f8e-b20ae1aff266)

AWS Comprehend identified the parts of speech for each word in the sentence.

## AWS Rekognition

**AWS Rekognition** is a service that analyzes images and videos using machine learning. It can detect objects, scenes, activities, text, faces, and facial expressions in images.

### Creating an S3 Bucket and Uploading Images
Before we can use AWS Rekognition, we need to store our images in an S3 bucket.

#### Set Up S3 Bucket and Upload Images
We created an S3 bucket to store images and uploaded four images to test various Rekognition features: label detection, facial analysis, and text detection
```python3
import boto3

# Initialize the S3 resource
s3 = boto3.resource('s3')

# Define the bucket name and region
bucket_name = '23803313-lab9'  # Use your student ID to name the bucket
region = 'ap-south-1'  # Replace with your AWS region

# Create the S3 bucket
try:
    s3.create_bucket(Bucket=bucket_name, CreateBucketConfiguration={'LocationConstraint': region})
    print(f"S3 bucket '{bucket_name}' created.")
except s3.meta.client.exceptions.BucketAlreadyOwnedByYou:
    print(f"S3 bucket '{bucket_name}' already exists.")
except Exception as e:
    print(f"Error creating bucket: {e}")

# List of images to upload
images = ['urban.jpg', 'beach.jpg', 'faces.jpg', 'text.jpeg']

# Upload images to S3 bucket
for image in images:
    try:
        s3.Bucket(bucket_name).upload_file(image, image)
        print(f"Uploaded {image} to {bucket_name}")
    except Exception as e:
        print(f"Error uploading {image}: {e}")
```
**Explanation:**
- Import the `boto3` library and initialize the S3 resource.
- Define the S3 bucket name and region.
- We attempt to create the S3 bucket. If the bucket already exists and is owned by us, we handle the exception.
- Define a list of image filenames that we want to upload to the S3 bucket.
- Loop over the images and upload each one to the S3 bucket.
- Handle any exceptions that may occur during the upload process.
  
Note: We had to change the format of one of the images to JPEG due to compatibility issues.

![image](https://github.com/user-attachments/assets/9ddff8f5-aca7-41db-92ef-b49c829e3a21)

### Label Detection, Facial Analysis, Text Detection    
We utilized AWS Rekognition to perform label detection, facial analysis, and text detection.

```python3
import boto3

# Initialize Rekognition client
rekognition = boto3.client('rekognition')

# Function to detect labels
def detect_labels(image_name, bucket_name):
    # Call Rekognition to detect labels in the image
    response = rekognition.detect_labels(
        Image={'S3Object': {'Bucket': bucket_name, 'Name': image_name}},
        MaxLabels=10
    )
    # Extract label names and confidence scores
    labels = [(label['Name'], int(label['Confidence'])) for label in response['Labels']]
    return labels

# Function to detect faces
def detect_faces(image_name, bucket_name):
    # Call Rekognition to detect faces in the image
    response = rekognition.detect_faces(
        Image={'S3Object': {'Bucket': bucket_name, 'Name': image_name}},
        Attributes=['ALL']
    )
    # Extract confidence and emotions for each face
    faces = [(int(face['Confidence']), [emotion['Type'] for emotion in face['Emotions']]) for face in response['FaceDetails']]
    return faces

# Function to detect text
def detect_text(image_name, bucket_name):
    # Call Rekognition to detect text in the image
    response = rekognition.detect_text(
        Image={'S3Object': {'Bucket': bucket_name, 'Name': image_name}}
    )
    # Extract detected text and confidence scores
    texts = [(text['DetectedText'], int(text['Confidence'])) for text in response['TextDetections']]
    return texts

# List of images to analyze
images = ['urban.jpg', 'beach.jpg', 'faces.jpg', 'text.jpeg']

# Analyze each image
for image in images:
    # Detect labels in the image
    labels = detect_labels(image, bucket_name)
    print(f"Labels in {image}: {labels}")
    
    # If the image is 'faces.jpg', perform facial analysis
    if image == 'faces.jpg':
        faces = detect_faces(image, bucket_name)
        print(f"Facial analysis for {image}: {faces}")
    
    # If the image is 'text.jpeg', perform text detection
    if image == 'text.jpeg':
        texts = detect_text(image, bucket_name)
        print(f"Text detected in {image}: {texts}")
```
**Explanation**
- Initialized the Rekognition client.
- Defined three functions:
  - `detect_labels`: Detects labels in an image and returns a list of label names and confidence scores.
    - Calls `rekognition.detect_labels` with the specified image from S3.
    - Specifies `MaxLabels=10` to limit the number of labels returned.
    - Extracts the `Name` and `Confidence` for each label detected
    - Returns List of tuples containing label names and confidence scores
  - `detect_faces`: Detects faces in an image and returns confidence scores and detected emotions for each face.
    - Calls `rekognition.detect_faces` with the specified image from S3.
    - Specifies `Attributes=['ALL']` to get detailed facial attributes.
    - Extracts the `Confidence` and Emotions for each face detected.
    - Returns List of tuples containing confidence scores and list of emotions.
  - `detect_text`: Detects text in an image and returns detected text and confidence scores.
    - Calls `rekognition.detect_text` with the specified image from S3.
    - Extracts the `DetectedText` and `Confidence` for each text detected.
    - Returns List of tuples containing detected text and confidence scores.
- Defined a list of images to analyze.
- Iterated over each image:
  - Detected labels in the image and printed the results.
  - If the image is `'faces.jpg'`, performed facial analysis and printed the results.
  - If the image is `'text.jpeg'`, performed text detection and printed the results.
    
![image](https://github.com/user-attachments/assets/14c534c7-1f24-43b6-aed3-04ad37bff392)

We obtained the labels, facial analysis, and text detection results for the respective images.
