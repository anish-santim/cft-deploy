# ETL Pipeline with AWS Step Functions

When a user uploads CSV file is uploaded to the AWS S3 (Simple Storage Service) Bucket source folder, Eventbridge is triggered.
This Eventbridge initiates the execution of a Step Function. 
Initially, it will validate the unprocessed file using lambda function and then generate a schema using Glue Crawler. 
Finally, it will execute a Glue Job to convert the CSV file into JSON format and store it in the Destination source bucket for furthur analysis.

## Prerequisites 

* An active AWS account with programmatic access
* AWS CLI with AWS account configuration, so that you can create AWS resources by deploying CloudFormation stack
* Amazon S3 bucket console access
* AWS Glue console access
* AWS Step Functions console access

## Product Versions
* Python 3 for AWS Lambda
* AWS Glue version 3.0

## High level work flow

1. User uploads a csv file. AWS S3 Bucket will send notifications to Amazon EventBridge.
2. Amazon EventBridge will starts the step function state machine execution.
3. AWS Lambda function validates the data type of the raw file uploaded.
4. AWS Glue Crawler create the schema of the raw file and store in glue database table.
5. AWS Glue job transform the raw file into csv format.
6. AWS Glue job also move the transformed file to the destination folder.
7. AWS SNS sends error notification for any error inside workflow.

## Repository Structure
- template.yaml - CloudFormation template file
- script.py - ETL job transformation script
- data.csv - File to be transformed using ETL Job

## Deploy
This pattern can be deployed through AWS CloudFormation template.

Follow the below step to deploy this pattern using CloudFormation template file [template.yml](template.yml) included in this repository.

1.	Clone the Repo.
2.	Navigate to the Directory.
3.  Update the following parameters in template.yaml file - 
    - SourceBucket - Unique bucket name. This bucket will be created to store the users dataset.
    - DestinationBucket - Unique bucket name. This bucket will be created to store the transformed result dataset.
    - ScriptBucket - Unique bucket name. This bucket will be created to store the etl job transformation script.
    - SNSTopicEmail - A valid email address to receive error notification.
    - GlueCrawlerName - Glue crawler name which will crawl raw file.
    - GlueDatabaseName - Glue database which will store the schema.
4.  Update the following parameters in script.py file - 
    - database-name - Glue database name which will contain glue table.
    - table-name - Glue table name which will store the schema.
    - destination-bucket-name - Destination bucket name which will store the transformed result dataset.
3.  Execute the following AWS CLI command with pre-configured AWS CLI profile

    *aws s3 mb s3://<etl-script-bucket>

    *aws s3 cp script.py s3://<etl-script-bucket>/script.py 

    *aws cloudformation create-stack --stack-name etlstack --template-body file://template.yaml --capabilities CAPABILITY_NAMED_IAM

4. Check the progress of CloudFormation stack deployment in console and wait for it to finish.


## Test

1. Enable the event bridge notification for source bucket using aws console.
2. Once the notification is enabled, navigate to source S3 bucket.
3. Upload the script.py file to script bucket. Or you can use AWS CLI commands as well.

    *aws s3 cp script.py s3://<etl-script-bucket>/script.py

3. Upload a sample csv file to source bucket. Or you can use AWS CLI commands as well.

    *aws s3 cp data.csv s3://<etl-source-bucket>/data.csv

4. Check the ETL pipeline status in the AWS Step Functions console.
5. Once ETL pipeline completes, check for transformed file in destination folder.

## Sample Workflow Execution and Notification
### Successful Execution
<img src="images/Successful_Execution.png">


### Failed Execution 
<img src="images/Failed_Execution.png">



