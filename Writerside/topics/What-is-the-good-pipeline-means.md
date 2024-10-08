# Qué significa un buen pipeline CI/CD?

With serverless development, the term “deployment” can take on a whole new meaning. When developing serverless 
applications, you no longer deploy new application code to servers, because there are no servers.

Using infrastructure as code services, such as AWS CloudFormation, AWS Cloud Development Kit (AWS CDK), 
Terraform, and the Serverless Framework, developers are able to create AWS resources in an orderly and 
predictable fashion.

With AWS Lambda, a deployment can be as simple as an API call to create a function or update the function code.

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

## Additional resources

* [Generating starter CI/CD pipelines using AWS SAM Pipelines](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-generating-example-ci-cd.html)
* [CI/CD for Serverless and Containerized Applications](https://www.youtube.com/watch?v=01ewawuL-IY&t=419s)
