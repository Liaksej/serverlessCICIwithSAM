# Что такое SAM

## AWS SAM

AWS SAM is an open source framework you can use to build your serverless applications. It provides you with a 
shorthand syntax to express your functions, APIs, databases, and event source mappings.

During your deployments, SAM then transforms and expands the SAM syntax into an AWS CloudFormation syntax. 
CloudFormation can then provision your resources with reliable deployment capabilities, making the deployment of your 
serverless application simpler.

### SAM templates

AWS SAM templates are an extension of the AWS CloudFormation templates, with some additional components that make them 
easier for you to work with. Some of these additional components include the following:

* Create AWS CloudFormation compatible templates using shorthand syntax.
* Use infrastructure as code to define your Lambda functions, API Gateway APIs, serverless application from the AWS 
Serverless Application Repository, and DynamoDB tables.
* If any errors are detected while deploying your template, AWS CloudFormation will roll back the template and delete 
any resources that were created, leaving your environment exactly as it was before the deployment.

### Example of a SAM template

AWS SAM requires the use of the transform directive and a resource block with a corresponding type. 
The transform directive takes an entire template written in the AWS SAM syntax and transforms and expands it into a 
compliant AWS CloudFormation template. You can also optionally include any resource in a SAM template.

![image](https://explore.skillbuilder.aws/files/a/w/aws_prod1_docebosaas_com/1728054000/OkLFRSay1U8Sv7H0Idus2Q/tincan/675621_1654804371_p1g5509l8kdo9ri4lrh1vhpr5f4_zip/assets/76NnqtBLaO8PxNVq_j-e6BlWiBKpuO2jG.png)

### Deploying SAM templates with the SAM CLI

This diagram summarizes the process of developing with AWS SAM. You begin by writing your Lambda function code and 
defining all of your serverless resources inside an AWS SAM template. You can use the SAM CLI to emulate the Lambda 
environment and perform local tests on your Lambda functions. After the code and templates are validated, you can then 
use the SAM package command to create a deployment package, which is essentially a .zip file that SAM stores in Amazon S3. 
After that, the SAM deploy command instructs AWS CloudFormation to deploy the .zip file to create resources inside of your AWS console.

![image](https://explore.skillbuilder.aws/files/a/w/aws_prod1_docebosaas_com/1728054000/OkLFRSay1U8Sv7H0Idus2Q/tincan/675621_1654804371_p1g5509l8kdo9ri4lrh1vhpr5f4_zip/assets/FRRRFN5zNMSL3EE2_cxjqTIMYuVN7kQ_n.png)

