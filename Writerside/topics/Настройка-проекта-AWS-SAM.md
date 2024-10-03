# 3. Настройка проекта AWS SAM

### 3.1. Создание нового проекта SAM
The AWS Serverless Application Model (AWS SAM) is an open-source framework that developers use to build production-grade 
serverless applications on AWS.

A serverless application is a combination of Lambda functions, event sources, and other resources that work together to 
perform tasks. But a serverless application is more than just a Lambda function—it can include additional resources such 
as APIs, databases, and event source mappings.

In this chapter, you will learn how to quickly scaffold a SAM application and understand how it is structured.

#### Initialize project

AWS SAM provides you with a command line tool, the AWS SAM CLI, which makes it easy for you to create and manage 
serverless applications. In particular, scaffolding a new project becomes easier by creating the initial skeleton of an 
application from which you can continue building your project.

Run the following command to scaffold a new project:

```shell
sam init
```

In the wizard, select AWS QuickStart Templates and Hello World Example. Do not use the shortcut to use the latest Python version.

```
Choose an AWS Quick Start application template
        1 - Hello World Example
        2 - Data processing
        3 - Hello World Example with Powertools for AWS Lambda
        4 - Multi-step workflow
        5 - Scheduled task
        6 - Standalone function
        7 - Serverless API
        8 - Infrastructure event management
        9 - Lambda Response Streaming
        10 - Serverless Connector Hello World Example
        11 - Multi-step workflow with Connectors
        12 - GraphQLApi Hello World Example
        13 - Full Stack
        14 - Lambda EFS example
        15 - DynamoDB Example
        16 - Machine Learning
Template: 1

Use the most popular runtime and package type? (Python and zip) [y/N]: n
```

Next, select your preferred runtime and version. Make sure to select the correct version as shown below.

```
Which runtime would you like to use?
        1 - aot.dotnet7 (provided.al2)
        2 - dotnet8
        3 - dotnet6
        4 - go (provided.al2)
        5 - go (provided.al2023)
        6 - graalvm.java11 (provided.al2)
        7 - graalvm.java17 (provided.al2)
        8 - java21
        9 - java17
        10 - java11
        11 - java8.al2
        12 - nodejs20.x
        13 - nodejs18.x
        14 - nodejs16.x
        15 - python3.9
        16 - python3.8
        17 - python3.12
        18 - python3.11
        19 - python3.10
        20 - ruby3.3
        21 - ruby3.2
        22 - rust (provided.al2)
        23 - rust (provided.al2023)
```

```
nodejs20.x
```

Select Zip as the package type and leave sam-app as the Project name.

```
What package type would you like to use?
        1 - Zip
        2 - Image
Package type: 1

Based on your selections, the only dependency manager available is npm.
We will proceed copying the template using npm.

Select your starter template
        1 - Hello World Example
        2 - Hello World Example TypeScript
Template: 2

Would you like to enable X-Ray tracing on the function(s) in your application?  [y/N]: n

Would you like to enable monitoring using CloudWatch Application Insights?
For more info, please view https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/cloudwatch-application-insights.html [y/N]: n

Project name [sam-app]:
```
Project should now be initialized

You should see a new folder sam-app created with a basic Hello World scaffolding.


>If you are interested in learning more about initializing SAM projects, 
you can find the full reference for the sam init command in the [SAM CLI reference](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-init.html).


### 3.2. Описание файлов и структуры проекта (template.yaml, app.py и т.д.) 

### 3.3. Описание принципов разработки Lambda на TypeScript с использованием esbuild

### 3.4. Локальное тестирование функций Lambda 

### 3.5. Запуск локального API