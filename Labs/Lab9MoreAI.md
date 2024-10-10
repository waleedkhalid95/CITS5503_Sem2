# Lab 9


## Background

The aim of this lab is to write a series of scripts that will test the main features of AWS Comprehend and AWS Rekognition.

## AWS Comprehend

AWS Comprehend offers different services to analyse text using machine learning. With Comprehend API, you will be able to perform common NLP tasks such as sentiment analysis, or simply detecting the language from the text.

"Amazon Comprehend can discover the meaning and relationships in text from customer support incidents, product reviews, social media feeds, news articles, documents, and other sources. For example, you can identify the feature that's most often mentioned when customers are happy or unhappy about your product."

since comprehend is not availiable in ap-northeast-3, we will use ap-south-1

to test aws comprehend we will use the example program below
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
This means that the detected language is 'en' (English) and has a confidence in the prediction greater than 0.99. 

![image](https://github.com/user-attachments/assets/4a402681-05f6-46a1-9db4-8fb7f4e4c3e4)

**NOTE**: remember that often in machine learning the confidence score is expressed as a value in the range [0,1] where 0 indicates the lack of certainty and 1 means totally certain of the prediction.

### Detect Languages from text
first we create a Jupyter notebook, to keep a recort of our programs. 
```
python3 -m notebook
```

#### [1] Modify the code above
Based on the previous code, write a python script that can detect different languages. Besides, instead of language code (e.g., 'en' for English or 'it' for Italian), the script should be return the message "<predicted_language> detected with <xx> confidence" where <predicted_language> correspond to the name of the language in English and <xx> is given as a percentage. For the previous example, the result should look like this:

#### [2] Test your code with other languages

Test your code using the following texts in different languages:

**English:**
"The French Revolution was a period of social and political upheaval in France and its colonies beginning in 1789 and ending in 1799."


**Spanish:**
"El Quijote es la obra más conocida de Miguel de Cervantes Saavedra. Publicada su primera parte con el título de El ingenioso hidalgo don Quijote de la Mancha a comienzos de 1605, es una de las obras más destacadas de la literatura española y la literatura universal, y una de las más traducidas. En 1615 aparecería la segunda parte del Quijote de Cervantes con el título de El ingenioso caballero don Quijote de la Mancha."

**French:**
"Moi je n'étais rien Et voilà qu'aujourd'hui Je suis le gardien Du sommeil de ses nuits Je l'aime à mourir Vous pouvez détruire Tout ce qu'il vous plaira Elle n'a qu'à ouvrir L'espace de ses bras Pour tout reconstruire Pour tout reconstruire Je l'aime à mourir"
[From the Song: "Je l'Aime a Mourir" - Francis Cabrel ]

**Italian:**
"L'amor che move il sole e l'altre stelle."
[Quote from "Divine Comedy" - Dante Alighieri]

```python3
import boto3

def detect_language(text):
    client = boto3.client('comprehend')
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
# Test with different texts
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

### Analyze sentiment 

Sentiment analysis (or opinion mining) uses NLP to determine whether data is positive, negative or neutral. Sentiment analysis is often performed on textual data to help businesses monitor brand and product sentiment in customer feedback, and understand customer needs.

Use boto3 and AWS comprehend to create a python script for sentiment analysis and apply the previous 4 texts to test the script.

```python 3
def analyze_sentiment(text):
    client = boto3.client('comprehend')
    response = client.detect_sentiment(Text=text, LanguageCode='en')
    return f"Sentiment: {response['Sentiment']}"
for text in texts:
    print(analyze_sentiment(text))
```

![image](https://github.com/user-attachments/assets/9367425e-535d-4a4b-9f0a-e86f3e718c81)

### Detect entities

Use boto3 and AWS comprehend to create a python script for entities detection and apply the previous 4 texts to test the script.
```python3
def detect_entities(text):
    client = boto3.client('comprehend')
    response = client.detect_entities(Text=text, LanguageCode='en')
    return [(entity['Text'], entity['Type'], int(entity['Score'] * 100)) for entity in response['Entities']]
for text in texts:
    print(detect_entities(text))
```

![image](https://github.com/user-attachments/assets/1c0ef251-0962-4db3-aa05-ae12ba180759)

Answer this question: describe what entities are in your own words.

### Detect keyphrases

Use boto3 and AWS comprehend to create a python script for keyphrases detection and apply the previous 4 texts to test the script.
```python3
def detect_key_phrases(text):
    client = boto3.client('comprehend')
    response = client.detect_key_phrases(Text=text, LanguageCode='en')
    return [(phrase['Text'], int(phrase['Score'] * 100)) for phrase in response['KeyPhrases']]
for text in texts:
    print(detect_key_phrases(text))
```

![image](https://github.com/user-attachments/assets/fd5f7ad6-1526-48d3-92b6-59e2cef28b8f)


Answer this question: describe what keyphrases are in your own words.

### Detect syntaxes

Use boto3 and AWS Comprehend to create a python script for syntax detection and apply the previous 4 texts to test the script.
```pyhton3
def detect_syntax(text):
    client = boto3.client('comprehend')
    response = client.detect_syntax(Text=text, LanguageCode='en')
    return [(token['Text'], token['PartOfSpeech']['Tag']) for token in response['SyntaxTokens']]
for text in texts:
    print(detect_syntax(text))
```

![image](https://github.com/user-attachments/assets/d3b333c7-f824-4640-9f8e-b20ae1aff266)


Answer this question: describe what syntaxes are in your own words.

## AWS Rekognition

AWS Rekognition is the service of AWS that allows you to perform machine learning tasks on images.

Currently, given an image, AWS Rekognition allows:
1. **Label Recognition**: automatically label objects, concepts, scenes, and actions in your images, and provide a confidence score.
2. **Image Moderation**: automatically detect explicit or suggestive adult content, or violent content in your images, and provide confidence scores.
3. **Facial Analysis**: get a complete analysis of facial attributes, including confidence scores.
4. **Extract Text from an image**: automatically detect and extract text in your images.

### Add images

Create a python script: create an S3 bucket named as <studentid>-lab9 in the region you are mapped to. Add the 4 following images into the bucket:

1. Add an image of an urban setting (named as urban.jpg).

2. Add an image of a person on the beach (named as beach.jpg).

3. Add an image with people showing their faces (named as faces.jpg).

4. Add an image with texts (named as text.jpg).

### Test AWS rekognition

Update the python script above by using boto3 and AWS rekognition to test label recognition, image moderation, facial analysis and text extraction from images.

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
