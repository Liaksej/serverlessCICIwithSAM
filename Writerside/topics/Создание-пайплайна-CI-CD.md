# Построение CI/CD pipelines для serverless проектов с SAM

* [Создание CI/CD пайплайна с помощью SAM](#)
* [Структура проекта и файлов, отвчающих за работу pipeline](#)
* [Структура проекта и файлов, отвчающих за работу pipeline](#)
* [Конфигурация CI/CD пайплайна для работы с SAM](#)
* [Настройка Canary и Linear deployment с SAM](#)

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
