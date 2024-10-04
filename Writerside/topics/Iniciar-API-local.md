# 3.5 Deploy Manually with SAM

Before we start building a fully automated CI/CD pipeline, you will learn how to build, package, and deploy a serverless 
application using the AWS SAM CLI.

![image_3.5.1.png](image_3.5.1.png)

It is important to learn the foundation of how to package and deploy a serverless application even though this workshop 
teaches how to automate deployments with a CI/CD pipeline. Learning how to perform manual deployments is also useful for 
developers who want to create a personal non-production stack.

## Let's talk about artifacts

Artifacts refer to the output of your build process in the context of CI/CD. Artifacts are typically in the form of a 
zip/tar file, container image, or a binary, for example. You take these artifacts and deploy them onto your different 
environments (i.e. Dev, Test, Prod). For serverless projects, zip artifacts must be uploaded to an S3 bucket for the 
Lambda service to pick them up. The SAM CLI takes care of managing this process of uploading artifacts to S3 and 
referencing them at deployment time.

> **Note**
> AWS Lambda supports [container images as a deployment packaging format](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-package.html#gettingstarted-package-images). SAM supports building container images. 
> Future versions of this workshop will walk through the process of building Lambda functions using the container 
> packaging format.

### The Zip file

The first artifact that gets generated in a serverless project is your application code and supporting libraries. 
By default, these are compressed in a zip file and uploaded to an S3 bucket by the SAM CLI during the package 
phase (more on this later).

### The Packaged Template

The second artifact that SAM CLI generates during the package phase is the packaged template. 
Which is a copy of your project's template.yaml, except that it references the location of the zip file (first artifact) 
in the S3 bucket. The following image shows an example of a packaged template.

![image_3.5.2.png](image_3.5.2.png)

Notice how the CodeUri references the zip file on an S3 bucket, rather than on a local directory. 
This is how AWS Lambda is able to pull your code at deployment time.

## Build the app

To build a SAM project, we are going to use the sam build command. This command iterates through the functions in your 
application, looking for the manifest file (such as `requirements.txt`, `package.json` or `pom.xml`) that contain 
dependencies, and automatically creates deployment artifacts.

From the root of the sam-app folder, run the following command in the terminal:

```shell
cd ~/environment/sam-app
sam build
```

```
Building codeuri: /home/ec2-user/environment/sam-app/hello-world runtime: nodejs16.x metadata: {} architecture: x86_64 functions: ['HelloWorldFunction']
Running NodejsNpmBuilder:NpmPack
Running NodejsNpmBuilder:CopyNpmrc
Running NodejsNpmBuilder:CopySource
Running NodejsNpmBuilder:NpmInstall
Running NodejsNpmBuilder:CleanUpNpmrc
Running NodejsNpmBuilder:LockfileCleanUp

Build Succeeded

Built Artifacts  : .aws-sam/build
Built Template   : .aws-sam/build/template.yaml

Commands you can use next
=========================
[*] Invoke Function: sam local invoke
[*] Test Function in the Cloud: sam sync --stack-name {stack-name} --watch
[*] Deploy: sam deploy --guided
```

### Build completed

When the build finishes successfully, you will see a new directory created in the root of the project named `.aws-sam`. 
It is a hidden folder, so if you want to see it in the IDE, **make sure you enable** `Show hidden files` in Cloud9 to see it.

![image_3.5.3.png](image_3.5.3.png)

### Explore the build folder

Take a moment to explore the content of the build folder. Notice that the unit tests are automatically excluded and 3rd 
party dependencies are included. SAM takes care of this for us.

![image_3.5.4.png](image_3.5.4.png)

The build folder includes the app.js file as the entrypoint for the Lambda application, the node_modules directory with 
the dependencies, and the package.json file which declares the application's dependencies.

```
admin:~/environment/node-sam-app $ ls -l .aws-sam/build/HelloWorldFunction/
total 8
-rw-r--r-- 1 ec2-user ec2-user 331 Oct 26  1985 app.js
drwxrwxr-x 5 ec2-user ec2-user  57 Mar  9 22:35 node_modules
-rw-r--r-- 1 ec2-user ec2-user 468 Oct 26  1985 package.json
```

## Deploy the app

The `sam deploy` command deploys your application by launching a CloudFormation stack. This command has a guided 
interactive mode, which you enable by specifying the `--guided parameter`. It is recommended to deploy with guided mode 
for the first time as it will capture the configuration for future deployments in a new file `samconfig.toml` described 
at the end of this section.

Run the following command in the same directory level where the `template.yaml` is located:

```shell
cd ~/environment/sam-app
sam deploy --guided
```

This will walk you through a series of questions. Your answers will be saved to the configuration file at the end 
which will speed up future deployments. Pressing the Enter key will accept the default value displayed in brackets for 
each question, for example `[sam-app]`. Letters in UPPERCASE are defaults, for example, pressing enter to `[y/N]` will 
default in `No`.

> **Missing authorization**
> Make sure to answer y to the question about missing authorization: HelloWorldFunction may not have authorization defined, 
> Is this okay? [y/N]: y

```
Configuring SAM deploy
======================

   Looking for config file [samconfig.toml] :  Not found

   Setting default arguments for 'sam deploy'
   =========================================
   Stack Name [sam-app]:
   AWS Region [us-west-2]:
   #Shows you resources changes to be deployed and require a 'Y' to initiate deploy
   Confirm changes before deploy [y/N]:
   #SAM needs permission to be able to create roles to connect to the resources in your template
   Allow SAM CLI IAM role creation [Y/n]:
   #Preserves the state of previously provisioned resources when an operation fails
   Disable rollback [y/N]:
   HelloWorldFunction may not have authorization defined, Is this okay? [y/N]: y
   Save arguments to configuration file [Y/n]:
   SAM configuration file [samconfig.toml]:
   SAM configuration environment [default]:
```

### Deployment completed

This command might take a few minutes to finish because it is creating the resources (Lambda function, API Gateway, 
and IAM roles) in the AWS account. When it completes successfully, you should see an output similar to the following:

```
CloudFormation outputs from deployed stack
---------------------------------------------------------------------------------------------------------------------------
Outputs
---------------------------------------------------------------------------------------------------------------------------
Key                 HelloWorldFunctionIamRole
Description         Implicit IAM Role created for Hello World function
Value               arn:aws:iam::123456789123:role/sam-app-node-HelloWorldFunctionRole-1QWE9DKNCP6V3

Key                 HelloWorldApi
Description         API Gateway endpoint URL for Prod stage for Hello World function
Value               https://01111gpgpg.execute-api.us-west-2.amazonaws.com/Prod/hello/

Key                 HelloWorldFunction
Description         Hello World Lambda Function ARN
Value               arn:aws:lambda:us-west-2:123456789123:function:sam-app-node-HelloWorldFunction-5RxoDCiBWUPV
---------------------------------------------------------------------------------------------------------------------------
```

Note the output value for `HelloWorldApi`. The Value is the HTTPS endpoint for your new API Gateway. You can load this 
URL in your browser or curl it in a terminal.

```shell
curl -s (insert HelloWorldApi Value)
```

```
curl -s https://01111gpgpg.execute-api.us-west-2.amazonaws.com/Prod/hello/

{"message":"hello my friend"}
```

### What just happened?

The guided deployment does few things for you. Let's take a quick look at what happened under the hood during the 
guided deployment to understand this process better.

1. Your codebase gets packaged as a zip file.
2. SAM creates an S3 bucket in your account, if it doesn't already exist.
3. Zip file is uploaded to the S3 bucket.
4. SAM creates the [packaged template](https://catalog.workshops.aws/complete-aws-sam/en-US/module-3-manual-deploy/10-bucket.md#the-packaged-template) that references the location of the zip file on S3.
5. The packaged template is also uploaded to the S3 bucket.
6. SAM starts the deployment via CloudFormation ChangeSets.

The first time you do a guided deployment, a new file `samconfig.toml` is created in the root of your project with your 
specified deployment parameters. This file speeds up future sam deploy commands by using the same parameters without 
having you to enter them again.

```toml
version = 0.1
[default]
[default.deploy]
[default.deploy.parameters]
stack_name = "sam-app"
s3_bucket = "aws-sam-cli-managed-default-samclisourcebucket-3ma345ba4af33"
s3_prefix = "sam-app"
region = "us-west-2"
capabilities = "CAPABILITY_IAM"
image_repositories = []
```

> Learn more about guided deployments and the `samconfig.toml` file with [this Blog Post](https://aws.amazon.com/blogs/compute/a-simpler-deployment-experience-with-aws-sam-cli).

## Inspect deployment

In the previous section, we deployed and tested our serverless application. If you are new to AWS SAM, it's good to 
understand how it creates resources with CloudFormation.

### Open the CloudFormation console

Navigate to the [AWS CloudFormation console](https://console.aws.amazon.com/cloudformation/home), make sure you are in 
the same region where you have been working on so far. You should see the new stack sam-app in the `CREATE_COMPLETE` status.

![image_3.5.5.png](image_3.5.5.png)

### View CloudFormation Outputs

Remember that our AWS SAM template defines three Outputs at the bottom of the `template.yaml` file.

```yaml
Outputs:
  HelloWorldApi:
    Description: "API Gateway endpoint URL for Prod stage for Hello World function"
    Value: !Sub "https://${ServerlessRestApi}.execute-api.${AWS::Region}.amazonaws.com/Prod/hello/"
  HelloWorldFunction:
    Description: "Hello World Lambda Function ARN"
    Value: !GetAtt HelloWorldFunction.Arn
  HelloWorldFunctionIamRole:
    Description: "Implicit IAM Role created for Hello World function"
    Value: !GetAtt HelloWorldFunctionRole.Arn
```

You saw in the previous section how the values of these variables are emitted to the console during deployment. 
It is useful to recognize that these are standard [CloudFormation Outputs](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/outputs-section-structure.html).

You can see these Outputs values in the CloudFormation console. Click on the `sam-app` stack and then go to the `Outputs` tab. 
In this tab, you will see the API Gateway URL, the Lambda function ARN, and the IAM Role ARN for the function.

### Delete the deployed app

In the next section, we are going to setup a CI/CD pipeline with SAM. Because of this, we don't need this SAM 
application that we manually deployed.

To delete the application, run `sam delete`. Answer y to all of the prompts.

```shell
cd ~/environment/sam-app
sam delete
```

```
  Are you sure you want to delete the stack sam-app in the region us-west-2 ? [y/N]: y
  Are you sure you want to delete the folder sam-app in S3 which contains the artifacts? [y/N]: y
```











