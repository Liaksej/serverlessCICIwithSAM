# What is the good pipeline means?

With serverless development, the term “deployment” can take on a whole new meaning. When developing serverless applications, you no longer deploy new application code to servers, because there are no servers.

Using infrastructure as code services, such as AWS CloudFormation, AWS Cloud Development Kit (AWS CDK), Terraform, and the Serverless Framework, developers are able to create AWS resources in an orderly and predictable fashion.

With AWS Lambda, a deployment can be as simple as an API call to create a function or update the function code.

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


## Automating the Deployment Pipeline

Modern applications frequently deploy updates to implement new features, but updating or changing a production application 
is often risky and might introduce bugs. It's important to consider your deployment strategy and determine how it can 
help you deploy safely and gradually.

### Safe and gradual deployments

When you check a piece of code into source control, you don’t want to wait for a human to manually approve it or have 
each piece of code run through different quality checks. You also don't want to release a piece of code to production 
as soon as you check it in. This lesson shows how to automate a safe, gradual CI/CD deployment process.

### Lambda versioning and aliases
To understand deployment strategies, you first need to understand the concept of Lambda versions and Lambda aliases.

#### Lambda versions

When you create a Lambda function, there is only one version called $LATEST. Any time you publish a version, 
Lambda takes a snapshot copy of $LATEST to create the new version. This copy cannot be modified.
![image](https://explore.skillbuilder.aws/files/a/w/aws_prod1_docebosaas_com/1728054000/OkLFRSay1U8Sv7H0Idus2Q/tincan/675621_1654804371_p1g5509l8kdo9ri4lrh1vhpr5f4_zip/assets/4C4dFG6_qEg9cO2I_NrnKtOK71kkkLSY7.gif)

#### Lambda aliases

A Lambda alias is a pointer to a specific function version. By default, an alias points to a single Lambda version. 
When the alias is updated to point to a different function version, all incoming request traffic will be redirected to 
the updated Lambda function version.

![image](https://explore.skillbuilder.aws/files/a/w/aws_prod1_docebosaas_com/1728054000/OkLFRSay1U8Sv7H0Idus2Q/tincan/675621_1654804371_p1g5509l8kdo9ri4lrh1vhpr5f4_zip/assets/2U1qqxVxhgyt0nD0_KXE0-taAyoDoOTa1.gif)

### Deployment strategies

With Lambda traffic shifting, you can send a small subset of traffic to your newest function version while keeping 
the majority of incoming production traffic to your old, stable version. Some of the following deployment strategies 
use traffic shifting. Traffic shifting helps you validate that your new Lambda version works as expected, before sending 
all production traffic to it.

#### All-at-once

All-at-once deployments instantly shift traffic from the original (old) Lambda function to the updated (new) Lambda 
function, all at one time. All-at-once deployments can be beneficial when the speed of your deployments matters. 
In this strategy, the new version of your code is released quickly, and all your users get to access it immediately.

![image](https://explore.skillbuilder.aws/files/a/w/aws_prod1_docebosaas_com/1728054000/OkLFRSay1U8Sv7H0Idus2Q/tincan/675621_1654804371_p1g5509l8kdo9ri4lrh1vhpr5f4_zip/assets/XoHW_kQiATFSkaTP_Qq2k-YqtjYkHdDAI.gif)

#### Canary

In a canary deployment, you deploy your new version of your application code and shift a small percentage of production 
traffic to point to that new version. After you have validated that this version is safe and not causing errors, 
you direct all traffic to the new version of your code.

![img](https://explore.skillbuilder.aws/files/a/w/aws_prod1_docebosaas_com/1728054000/OkLFRSay1U8Sv7H0Idus2Q/tincan/675621_1654804371_p1g5509l8kdo9ri4lrh1vhpr5f4_zip/assets/IxwTO6k4B5VpRaf7_J7YZxn9DxihXXYLq.gif)

#### Linear

A linear deployment is similar to canary deployment. In this strategy, you direct a small amount of traffic to your new 
version of code at first. After a specified period of time, you automatically increment the amount of traffic that you 
send to the new version until you’re sending 100 percent of production traffic.

![image](https://explore.skillbuilder.aws/files/a/w/aws_prod1_docebosaas_com/1728054000/OkLFRSay1U8Sv7H0Idus2Q/tincan/675621_1654804371_p1g5509l8kdo9ri4lrh1vhpr5f4_zip/assets/h3yvwiEveK-7GGQt_XNEUddKdeu5tek0J.gif)

### Comparing deployment strategies

To help you decide which deployment strategy to use for your application, you'll need to consider each option's consumer 
impact, rollback, event model factors, and deployment speed. The following comparison table illustrates these points.

| Deployment    | Consumer Impact                                  | Rollback                                      | Event Model Factors                     | Deployment Speed |
|---------------|--------------------------------------------------|-----------------------------------------------|-----------------------------------------|------------------|
| All-at-once   | All at once                                      | Redeploy older version                        | Any event model at low concurrency rate | Immediate        |
| Canary/Linear | 1-10% typical initial traffic shift, then phased | Revert 100% of traffic to previous deployment | Better for high-concurrency workloads   | Minutes to hours |

### Deployment preferences with AWS SAM

Traffic shifting with aliases is directly integrated into AWS SAM. If you’d like to use all-at-once, canary, or linear 
deployments with your Lambda functions, you can embed that directly into your AWS SAM templates. You can do this in the 
deployment preferences section of the template. AWS CodeDeploy uses the deployment preferences section to manage the 
function rollout as part of the AWS CloudFormation stack update. SAM has several pre-built deployment preferences you 
can use to deploy your code. See the following table for examples. 

| Deployment Preferences Type   | Description                                                                                                 |
|-------------------------------|-------------------------------------------------------------------------------------------------------------|
| Canary10Percent30Minutes      | Shifts 10 percent of traffic in the first increment. The remaining 90 percent is deployed 30 minutes later. |
| Canary10Percent5Minutes       | Shifts 10 percent of traffic in the first increment. The remaining 90 percent is deployed 5 minutes later.  |
| Canary10Percent10Minutes      | Shifts 10 percent of traffic in the first increment. The remaining 90 percent is deployed 10 minutes later. |
| Canary10Percent15Minutes      | Shifts 10 percent of traffic in the first increment. The remaining 90 percent is deployed 15 minutes later. |
| Linear10PercentEvery10Minutes | Shifts 10 percent of traffic every 10 minutes until all traffic is shifted.                                 |
| Linear10PercentEvery1Minute   | Shifts 10 percent of traffic every minute until all traffic is shifted.                                     |
| Linear10PercentEvery2Minutes  | Shifts 10 percent of traffic every 2 minutes until all traffic is shifted.                                  |
| Linear10PercentEvery3Minutes  | Shifts 10 percent of traffic every 3 minutes until all traffic is shifted.                                  |
| AllAtOnce                     | Shifts all traffic to the updated Lambda functions at one time.                                             |

## Creating a deployment pipeline

When you check a piece of code into source control, you don’t want to wait for a human to manually approve it or have
each piece of code run through different quality checks. A CI/CD pipeline can help automate the steps required to 
release your software deployment and standardize on a core set of quality checks. Here is an example of a deployment 
pipeline:

![image](https://explore.skillbuilder.aws/files/a/w/aws_prod1_docebosaas_com/1728054000/OkLFRSay1U8Sv7H0Idus2Q/tincan/675621_1654804371_p1g5509l8kdo9ri4lrh1vhpr5f4_zip/assets/el7WB5XC_CYE2DS2_zgdU_lh4XEjpPTZ1.jpg)

A CI/CD pipeline is mainly made up of four steps: Source, Build, Test, Production. CI/CD can be pictured as a pipeline, 
where new code is submitted on one end, tested over a series of stages (source, build, test, staging, and production), 
and then published as production-ready code.

![image](https://explore.skillbuilder.aws/files/a/w/aws_prod1_docebosaas_com/1728054000/OkLFRSay1U8Sv7H0Idus2Q/tincan/675621_1654804371_p1g5509l8kdo9ri4lrh1vhpr5f4_zip/assets/DZ1UoaCWMfRMXFVp_ky8CN063ZfgTWXbv.jpg)

AWS offers a tool for each phase of the pipeline. These tools include:

* AWS CodeCommit for Source
* AWS CodeBuild for Build and Test
* AWS CodeDeploy for Production
* AWS CodePipeline for fully managed continuous delivery

To learn more about CI/CD on AWS, see the [CI/CD on AWS whitepaper](https://docs.aws.amazon.com/whitepapers/latest/cicd_for_5g_networks_on_aws/cicd-on-aws.html).


### Automating your deployment pipeline

![image](https://explore.skillbuilder.aws/files/a/w/aws_prod1_docebosaas_com/1728054000/OkLFRSay1U8Sv7H0Idus2Q/tincan/675621_1654804371_p1g5509l8kdo9ri4lrh1vhpr5f4_zip/assets/MGQJYdFXiGyUhga2_1WZyHTw-qYqrfmAF.jpg)

This architecture shows how you can use AWS CodePipeline with AWS SAM. With CodePipeline, every time there is a code 
change or a new code commit, you can kick off this pipeline, where you build, test, and deploy that code change.

## AWS SAM Pipelines

AWS SAM Pipelines is a feature of AWS SAM that automates the process of creating a continuous delivery pipeline. 
AWS SAM Pipelines provides templates for popular CI/CD systems, such as AWS CodePipeline, Jenkins, GitHub Actions, 
Bitbucket Pipelines, and GitLab CI/CD. Pipeline templates include AWS deployment best practices to help with 
multi-account and multi-region deployments. AWS environments such as dev and production typically exist in different 
AWS accounts. Development teams can configure safe deployment pipelines, without making unintended changes to 
infrastructure. You can also supply your own custom pipeline templates to help to standardize pipelines across 
development teams. 

AWS SAM Pipelines is composed of two commands:

1. **`sam pipeline bootstrap`**, a configuration command that creates the AWS resources and permissions required to 
deploy application artifacts from your code repository into your AWS environments. 
2. **`sam pipeline init`**, an initialization command that generates a pipeline configuration file that your 
CI/CD system can use to deploy serverless applications using AWS SAM.

With two separate commands, you can manage the credentials for operators and developers separately. 
Operators can use sam pipeline bootstrap to provision AWS pipeline resources. 
This can reduce the risk of production errors and operational costs. Developers can then focus on building without 
having to set up the pipeline infrastructure by running the sam pipeline init command.

You can also combine these two commands by running **`sam pipeline init --bootstrap`**. 
This takes you through the entire guided bootstrap and initialization process.

SAM Pipelines creates appropriate configuration files for your CI/CD provider of choice. 
For example, when using AWS CodePipeline, SAM will synthesize an AWS CloudFormation template file named 
`codepipeline.yaml`. This template defines multiple AWS resources that work together to deploy a serverless 
application automatically.

> SAM Pipelines saves you the work of setting your pipelines up from scratch. However, 
> the configurations that SAM Pipelines creates are simply a convenience to get you started. 
> You are free to edit these CI/CD configuration files after SAM creates them. 

### Creating pipeline deployment resources

The **`sam pipeline init --bootstrap`** command guides you through a series of questions to help produce the template 
file that creates the AWS resources and permissions required to deploy application artifacts from your code repository 
into your AWS environments. The **`--bootstrap`** option helps you to set up AWS pipeline stage resources before the template 
file is initialized.

![image](https://explore.skillbuilder.aws/files/a/w/aws_prod1_docebosaas_com/1728054000/OkLFRSay1U8Sv7H0Idus2Q/tincan/675621_1654804371_p1g5509l8kdo9ri4lrh1vhpr5f4_zip/assets/vJzn3yu210rD5bRN_pLHYBxb-qY_fcx8I.png)

After AWS SAM has created the supporting resources, the guided walkthrough will prompt you to create the 
CloudFormation template that will define our entire CI/CD pipeline. To deploy the CI/CD pipeline template file, 
you use the **`sam deploy`** command.

> It’s important to recognize that you’re using SAM’s ability to launch arbitrary CloudFormation templates. 
> SAM isn’t building or deploying your serverless application here, rather launching the CI/CD template that was 
> created through the SAM Pipelines bootstrap process. 

### Deploy SAM templates with one command

Previously, deploying applications through the SAM CLI took multiple steps and required you to provide an Amazon S3 
bucket for the Lambda deployment package. If you would like to use your own S3 bucket for your Lambda deployment package, 
you can still continue to use the **`sam package`** and **`sam deploy`** commands to deploy your SAM template. However, now you 
can use a single command, sam deploy, to deploy your serverless applications, and the SAM CLI will create and manage 
this S3 bucket for you. The following diagram is an example of the terminal output when the **`sam deploy`** command runs. 

![image](https://explore.skillbuilder.aws/files/a/w/aws_prod1_docebosaas_com/1728054000/OkLFRSay1U8Sv7H0Idus2Q/tincan/675621_1654804371_p1g5509l8kdo9ri4lrh1vhpr5f4_zip/assets/1mulJ7tiGWsGLV2q_-WGv20luHdXQbcJ8.jpg)

![image](https://explore.skillbuilder.aws/files/a/w/aws_prod1_docebosaas_com/1728054000/OkLFRSay1U8Sv7H0Idus2Q/tincan/675621_1654804371_p1g5509l8kdo9ri4lrh1vhpr5f4_zip/assets/Awg4mSynE89H67Q9_hAMbIicEvRvs1vFH.png)

The **`sam deploy`** command also comes with a guided interactive mode (**`sam deploy --guided`**). 
This mode walks you through the parameters required for deployment, provides default options, and saves your 
input for the given application. You can also see changes made to the application stack that will be deployed through 
the **`sam deploy`** command output and configure the command to ask for confirmation on changes before deploying.

## Additional resources

* [Generating starter CI/CD pipelines using AWS SAM Pipelines](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-generating-example-ci-cd.html)
* [CI/CD for Serverless and Containerized Applications](https://www.youtube.com/watch?v=01ewawuL-IY&t=419s)
