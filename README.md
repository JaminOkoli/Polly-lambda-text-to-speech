# Polly-lambda-text-to-speech Project

## Overview
This project is an AWS Lambda-based solution that converts text files stored in an S3 bucket into speech using Amazon Polly. The synthesized speech is then saved as an MP3 file in a designated S3 bucket. This automation enables seamless text-to-speech (TTS) processing with minimal manual intervention.

## Features
- **Automated Text-to-Speech Conversion**: Converts text files into high-quality speech using Amazon Polly.
- **AWS Lambda & S3 Integration**: Retrieves text files from an S3 source bucket and stores the generated MP3 files in a destination bucket.
- **Customizable Voice Output**: Supports multiple Polly voices to match different use cases.
- **Event-Driven Processing**: Uses S3 event triggers to automatically process new text files.

## Architecture
1. A text file is uploaded to the **source S3 bucket**.
2. An **S3 event trigger** invokes the AWS Lambda function.
3. The function:
   - Retrieves the text file.
   - Uses **Amazon Polly** to convert text to speech.
   - Saves the generated MP3 file to the **destination S3 bucket**.
4. The MP3 file is now available for download or playback.

![image](https://github.com/user-attachments/assets/31ccd556-6736-466b-a87e-828205bd14ee)


## AWS Services Used
- **Amazon S3**: Storage for input text files and output MP3 files.
- **AWS Lambda**: Serverless compute function to process text and invoke Polly.
- **Amazon Polly**: Text-to-Speech engine for generating natural-sounding speech.
- **IAM**: Manages permissions for Lambda to access S3 and Polly.

## Prerequisites
- An **AWS account** with necessary permissions.
- Two **S3 buckets**:
  - One for text file uploads (source bucket).
  - One for storing MP3 outputs (destination bucket).
- An **IAM Role** with permissions to access S3 and Polly.

## Deployment Instructions
### 1. Set Up S3 Buckets
Create two S3 buckets:
- `enzo-polly-source-bucket`
- `enzo-polly-destination-bucket`

### 2. Configure IAM Role
Create an IAM role with the following policy:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject"],
      "Resource": [
        "arn:aws:s3:::your-source-bucket-name/*",
        "arn:aws:s3:::your-destination-bucket-name/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": ["polly:SynthesizeSpeech"],
      "Resource": "*"
    }
  ]
}
```

### 3. Deploy Lambda Function
1. Create an AWS Lambda function using Python.
2. Upload the `lambda_function.py` script.
3. Set up environment variables:
   - `SOURCE_BUCKET`: `enzo-polly-source-bucket`
   - `DESTINATION_BUCKET`: `enzo-polly-destination-bucket`
4. Assign the IAM role to Lambda.

### 4. Configure S3 Event Trigger
- In the **source S3 bucket**, enable event notifications for `ObjectCreated` events.
- Configure it to trigger the Lambda function when new `.txt` files are uploaded.

## Usage
1. Upload a `.txt` file to the **source S3 bucket**.
2. Lambda will process the file and generate an MP3 output.
3. The generated audio file will be available in the **destination S3 bucket**.

## Example Lambda Function (Python)
```python
import boto3
import os
import logging

def lambda_handler(event, context):
    s3 = boto3.client('s3')
    polly = boto3.client('polly')
    
    source_bucket = os.environ['SOURCE_BUCKET']
    destination_bucket = os.environ['DESTINATION_BUCKET']
    text_file_key = event['Records'][0]['s3']['object']['key']
    audio_key = text_file_key.replace('.txt', '.mp3')
    
    text_file = s3.get_object(Bucket=source_bucket, Key=text_file_key)
    text = text_file['Body'].read().decode('utf-8')
    
    response = polly.synthesize_speech(Text=text, OutputFormat='mp3', VoiceId='Joanna')
    
    if 'AudioStream' in response:
        with open('/tmp/audio.mp3', 'wb') as file:
            file.write(response['AudioStream'].read())
        s3.upload_file('/tmp/audio.mp3', destination_bucket, audio_key)
    
    return {"statusCode": 200, "body": "Text-to-Speech conversion completed!"}
```

## Future Enhancements
- Add support for different languages and voices.
- Implement an API Gateway to trigger TTS conversion via HTTP requests.
- Store metadata for generated MP3 files in a DynamoDB table.

## License
This project is open-source and available under the MIT License.

## Contributing
Feel free to fork the repository, create issues, or submit pull requests for improvements!

---

This README provides a clear, structured overview of your project while making it more personal and detailed. Let me know if you'd like any modifications!


