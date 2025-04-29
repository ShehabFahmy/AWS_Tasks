1) Create a Policy for:
- S3 with permissions:
	- `GetObject` and `DeleteObject` for source bucket
	- `PutObject` for destination bucket  
- DynamoDB with permissions:
	- `PutItem` for the required DynamoDB
```json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "VisualEditor0",
			"Effect": "Allow",
			"Action": [
				"s3:GetObject",
				"dynamodb:PutItem",
				"s3:DeleteObject"
			],
			"Resource": [
				"arn:aws:dynamodb:*:119557588013:table/yat-task2-dynamoDB",
				"arn:aws:s3:::yat-task2-main/*"
			]
		},
		{
			"Sid": "VisualEditor1",
			"Effect": "Allow",
			"Action": "s3:PutObject",
			"Resource": "arn:aws:s3:::yat-task2-copy/*"
		}
	]
}
```
2) Create a role for lambda service with the created policy.
3) Create a lambda function and don't forget to choose `use an existing role` at `Change default execution role`.
4) At the lambda function overview, add S3 trigger with the source bucket.
5) Add Code:
```python
import json
import boto3

main_s3_client = boto3.client('s3')
dynamodb = boto3.resource('dynamodb')  # Use resource instead of client for DynamoDB Table

def lambda_handler(event, context):
    key = event['Records'][0]['s3']['object']['key']
    
    # Define your S3 bucket names
    main_bucket = 'yat-task2-main'
    copy_bucket = 'yat-task2-copy'
    table = dynamodb.Table('yat-task2-dynamoDB')
    
    try:
        # Copy object to destination bucket
        copy_source = {'Bucket': main_bucket, 'Key': key}
        main_s3_client.copy_object(CopySource=copy_source, Bucket=copy_bucket, Key=key)
        
        # Add copy bucket object URL to DynamoDB
        table.put_item(
            Item={
                'Name': key,
                'S3Url': f'https://{copy_bucket}.s3.amazonaws.com/{key}'
            }
        )

        return {
            'statusCode': 200,
            'body': json.dumps({
                'Success': 'Files are copied!'
            })
        }
    except Exception as e:
        return {
            'statusCode': 500,
            'body': json.dumps({
                'error': str(e)
            })
        }
```
6) Deploy the function.
7) Test by adding files to the main bucket.
