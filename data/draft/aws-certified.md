Development with AWS service
Use AWS services for different design patterns

- Event-driven
- Orchestration
- Fanout
- Sync vs Async

AWS services

- Step Functions
- Amazon DynamoDB
- AWS Lambda
- CodeBuild
- AWS SAM
- AWS CloudFormation
- API Gateway
- Amazon Cognito

Security

- IAM

Data classification
You should know:

- General data security patterns with clear mapping to security controls
- How to classify data
- How your data is stored
- Who has access to your data
- How to apply the two categories of security controls and categories

Preventive and detective controls
Preventive

- IAM
- Infrastructure security
- Data protection
  Detective
- Respond
- Configuration drift

Identifying the data

- AWS Macie
- Amazon Sagemaker
- Amazon S3
- AWS Glue Data Catalog?

A developer wants to deploy an AWS Serverless Application Model (AWS SAM) application with the AWS SAM CLI. The
developer is using the AWS CLI in an AWS Cloud9 environment for deployment and has run the sam build command to prepare
the application for deployment. The next day, the developer connects to the same AWS Cloud9 environment and attempts to
deploy the application. The sam deploy command returns the following error:
Invalid (or missing) template file (path must be workspace-relative, or absolute)
How can the developer resolve this error?

A developer has written several custom applications that read and write to the same Amazon DynamoDB table. Each time the
data in the DynamoDB table is modified, this change should be sent to an external API.
Which combination of steps should the developer perform to accomplish this task? (Select TWO.)

A company is working on a project to enhance its serverless application development process. The company hosts
applications on AWS Lambda. The development team regularly updates the Lambda code and wants to use stable code in
production.
Which combination of steps should the development team take to configure Lambda functions to meet both development and
production requirements? (Select TWO.)

https://docs.aws.amazon.com/AmazonElastiCache/latest/mem-ug/Strategies.html

Each time a developer publishes a new version of an AWS Lambda function, all the dependent event source mappings need to
be updated with the reference to the new version’s Amazon Resource Name (ARN). These updates are time consuming and
error-prone.
Which combination of actions should the developer take to avoid performing these updates when publishing a new Lambda
version? (Select TWO.)

An ecommerce company deploys more than 20 services behind Amazon API Gateway. The interaction between services is
complex. Each service can potentially call several others, making performance issues and errors difficult to identify.
Some individual API calls have experienced slow response times. The development team needs to quickly identify the
underlying causes of the slowdowns.
Which approach would MOST quickly identify the underlying cause of performance issues?
Report Content Errors

A company is developing a Python application that submits data to an Amazon DynamoDB table. The company requires
client-side encryption of specific data items and end-to-end protection for the encrypted data in transit and at rest.

Which combination of steps will meet the requirement to encrypt specific data items? (Select TWO.)
Correct. The AWS Database Encryption SDK provides end-to-end protection for your data in transit and at rest. You can
encrypt selected items or attribute values in a table.

For more information about DynamoDB client-side and server-side encryption, see Client-Side and Server-Side Encryption.

For more information about the AWS Database Encryption SDK, see What Is the AWS Database Encryption SDK?

For more information about how the AWS Database Encryption SDK works, see How the AWS Database Encryption SDK Works.

```markdown
A company is implementing an application on Amazon EC2 instances. The application needs to process incoming
transactions. When the application detects a transaction that is not valid, the application must send a chat message to
the company's support team. To send the message, the application needs to retrieve the access token to authenticate by
using the chat API.
A developer needs to implement a solution to store the access token. The access token must be encrypted at rest and in
transit. The access token must also be accessible from other AWS accounts.
Which solution will meet these requirements with the LEAST management overhead?

C. Use AWS Secrets Manager with an AWS Key Management Service (AWS KMS) customer managed key to store the access token.
Add a resource-based policy to the secret to allow access from other accounts. Update the IAM role of the EC2 instances
with permissions to access Secrets Manager. Retrieve the token from Secrets Manager. Use the decrypted access token to
send the message to the chat. Most Voted

知识点：
```

```markdown
A financial company must store original customer records for 10 years for legal reasons. A complete record contains
personally identifiable information (PII). According to local regulations, PII is available to only certain people in
the company and must not be shared with third parties. The company needs to make the records available to third-party
organizations for statistical analysis without sharing the PII.
A developer wants to store the original immutable record in Amazon S3. Depending on who accesses the S3 document, the
document should be returned as is or with all the PII removed. The developer has written an AWS Lambda function to
remove the PII from the document. The function is named removePii.
What should the developer do so that the company can meet the PII requirements while maintaining only one copy of the
document?

A. Set up an S3 event notification that invokes the removePii function when an S3 GET request is made. Call Amazon S3 by
using a GET request to access the object without PII.
B. Set up an S3 event notification that invokes the removePii function when an S3 PUT request is made. Call Amazon S3 by
using a PUT request to access the object without PII.
C. Create an S3 Object Lambda access point from the S3 console. Select the removePii function. Use S3 Access Points to
access the object without PII. Most Voted
D. Create an S3 access point from the S3 console. Use the access point name to call the GetObjectLegalHold S3 API
function. Pass in the removePii function name to access the object without PII.
```

知识点

```markdown

```
