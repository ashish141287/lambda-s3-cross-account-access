# lambda-s3-cross-account-access
AWS Lambda S3 cross-account access


An AWS account—for example, Account A—can grant another AWS account, Account B, permission to access its resources such as buckets and objects.

We will be performing following steps to carry out this activity.

1.) Create a Lambda role providing following permission to access bucket A in the AWS account A and bucket B in AWS account B. 
This lambda role will be used by lambda when the lambda function is triggered. Please add cloud watch logs access to the role.

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:PutObjectAcl"
            ],
            "Resource": [
                "arn:aws:s3:::bucket-A/*",
                "arn:aws:s3:::bucket-B/*"
            ]
        }
    ]
}

2.) Create a lambda function and add S3 trigger to it.

import boto3
import json

s3client = boto3.client("s3")
def lambda_handler(event, context):
    record = event["Records"][0]
    source_bucket = record['s3']['bucket']['name']
    getKey = record['s3']['object']['key']
    print(getKey)
    dest_key_path= "test/"
    filename = str(getKey)
    dest_key = dest_key_path + filename
    try:
        destbucket = 'bucket-B'
        copy_source = {'Bucket': source_bucket, 'Key': getKey}
        copyObject= s3client.copy_object(CopySource = copy_source, Bucket = destbucket, Key = dest_key)
        print(copyObject)
    except Exception as e:
        print(e)
        raise e

3.) Add the following bucket policy to bucket B in destination account B

{
    "Version": "2008-10-17",
    "Statement": [
        {
            "Sid": "",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::xxxxxxxxxxxxxxx:role/lambda-role-in-account-A"
            },
            "Action": [
                "s3:PutObject",
                "s3:PutObjectAcl"
            ],
            "Resource": "arn:aws:s3:::bucket-b/*"
        }
    ]
}


