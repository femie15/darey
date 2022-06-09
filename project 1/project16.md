# AUTOMATE INFRASTRUCTURE WITH IAC USING TERRAFORM PART 1

1. Create an IAM user named "terraform" and give it only "programatic access" to AWS account and grant "AdministratorAccess permissions" to it.
2. Configure programmatic access from the workstation to connect to AWS using "secret access key and access key ID" of the new user, and a <a href="https://boto3.amazonaws.com/v1/documentation/api/latest/guide/quickstart.html">Python SDK (boto3)</a>. (Python 3.6 or higher is recommended on your workstation), Use "AWS CLI with AWS configure" for easier authentication.
3. Create a S3 bucket and name it "femie13-dev-terraform-bucket"

To confirm the boto3 set up, on the terminal, run `python` , this will cause the terminal to expect python code, then enter the script below to fetch the list of S3 buckets.

```
import boto3
s3 = boto3.resource('s3')
for bucket in s3.buckets.all():
    print(bucket.name)
```




