# 4. Creación de pipeline CI/CD

In this chapter, you are going to use a feature of SAM called [SAM Pipelines](https://aws.amazon.com/blogs/compute/introducing-aws-sam-pipelines-automatically-generate-deployment-pipelines-for-serverless-applications/). 
When you are ready to deploy your serverless application in an automated manner, you can generate a deployment pipeline 
for your CI/CD system of choice. AWS SAM provides a set of starter pipeline templates with which you can generate 
pipelines in minutes using the sam pipeline init command.

Currently, the AWS SAM CLI supports generating starter CI/CD pipeline configurations for the following providers:

* [AWS CodePipeline](https://aws.amazon.com/codepipeline/) 
* [Jenkins](https://www.jenkins.io/) 
* [GitLab CI/CD](https://docs.gitlab.com/ee/ci/) 
* [GitHub Actions](https://github.com/features/actions) 
* [Bitbucket Pipelines](https://support.atlassian.com/bitbucket-cloud/docs/get-started-with-bitbucket-pipelines/) 

![image_4.1.png](image_4.1.png)

The different CI/CD chapters in this module present the same concepts using different CI/CD providers. 
You can proceed with the technology of your choice or even try them both.

### 4.1. Selección de herramientas para CI/CD (AWS CodePipeline, Github Actions y otros) 

Prerequisites

* [AWS SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html), last version
* [Git client](https://git-scm.com/downloads)
* IAM Role like describen in [Reqisitos previos](Предварительные-требования.md).
* [Bitbucket](https://bitbucket.org) account

### Create a Git Repository

Any CI/CD pipeline starts with a code repository. In this module, we use GitHub.

Please login with your existing Github account and create a GitHub repository named `sam-app`: [https://github.com/new](Please login with your existing Github account and create a GitHub repository named sam-app: https://github.com/new)

![image_4.1.1.png](image_4.1.1.png)

The repository's visibility (public or private) does not matter for this workshop.

### Push the code

Add your GitHub repository URL as a remote on your local git project.

## How to build a pipeline

The best way to automate the creation of CI/CD pipelines is by provisioning them programmatically using Infrastructure 
as Code (IaC). This is useful in a microservices environment where you may have a pipeline per service. 
In such environments, there could be dozens or even hundreds of CI/CD pipelines. Having an automated way to create 
those CI/CD pipelines enables developers to move quickly without the burden of building them manually. SAM Pipelines, 
which you'll be using, is a tool to ease that burden.

### Different ways to create cloud infrastructure

Technical teams use different IaC tools and frameworks to create cloud resources programmatically. 
A few options are listed below.

* [AWS SAM](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/what-is-sam.html)
* [AWS CloudFormation](https://docs.aws.amazon.com/codepipeline/latest/userguide/tutorials.html) 
* [AWS CDK](https://docs.aws.amazon.com/cdk/latest/guide/codepipeline_example.html) 
* [Terraform](https://www.terraform.io/docs/providers/aws/r/codepipeline.html) 

In this workshop, we are using AWS SAM, exclusively. It's worth clarifying the differences between AWS SAM and 
CloudFormation, if you're unfamiliar. It may be helpful to think of SAM as a higher level programming language like 
C or C++, and CloudFormation as assembly language. We do our work in AWS SAM which generates and deploys CloudFormation 
templates on our behalf. In previous sections, we used sam local commands to test our serverless application locally. 
AWS SAM is a toolset meant to increase productivity when developing serverless applications and provides capabilities 
such as sam local that are not present in other IaC tools.

## Introducing AWS SAM Pipelines

[SAM Pipelines](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-pipeline-bootstrap.html) 
works by creating a set of configuration and infrastructure files you use to create and manage your CI/CD pipeline.

As of this writing, SAM Pipelines can bootstrap CI/CD pipelines for the following providers:

* [AWS CodePipeline](https://aws.amazon.com/codepipeline/) 
* [Jenkins](https://www.jenkins.io/) 
* [GitLab CI/CD](https://docs.gitlab.com/ee/ci/) 
* [GitHub Actions](https://github.com/features/actions) 
* [Bitbucket Pipelines](https://support.atlassian.com/bitbucket-cloud/docs/get-started-with-bitbucket-pipelines/)

> SAM Pipelines is a feature which bootstraps CI/CD pipelines for the listed providers. 
> This saves you the work of setting them up from scratch. However, you can use SAM as a deployment tool with any 
> CI/CD provider. You use various sam commands to build and deploy SAM applications, regardless of your CI/CD toolset. 
> Furthermore, the configurations SAM Pipelines creates are a convienence to get you started. You are free to edit these 
> CI/CD configuration files after SAM creates them.

SAM Pipelines creates appropriate configuration files for your CI/CD provider of choice. For example, when using 
GitHub Actions, SAM will synthesize a `.github/workflows/pipeline.yaml` file. This file defines your CI/CD pipeline 
using GitHub Actions.

### GitHub architecture

At the end of this section, we will have a self-updating CI/CD pipeline using GitHub that will perform the following steps.

1. Trigger after a commit to the main branch (on: `push` in the screenshot below)
2. Run unit tests via GitHub Actions (`test`)
3. Build and package the application code via GitHub Actions (`build-and-package`)
4. Deploy to a dev/test environment (`deploy-testing`)
5. Integration test (`integration-test`)
6. Deploy to a production environment (`deploy-prod`)

![image_4.3.1.png](image_4.3.1.png)


## Creating the SAM Pipeline

Now that the AWS SAM application is hosted in a GitHub repository, we will create CI/CD resources in AWS that support 
a two stage deployment pipeline for the serverless application environment. This is a one-time operation. 
We will be building the following infrastructure:

![image_4.2.2.png](image_4.2.2.png)

We will then create a GitHub Actions workflow that securely integrates with AWS using GitHub as an identify provider. 
When the dev task in the GitHub Actions workflow attempts to assume the dev pipeline execution role in the AWS account, 
IAM validates that the supplied OIDC token originates from a trusted source. Configuration in IAM allows role assumption 
from specified GitHub repositories and branches. AWS SAM Pipelines performs the initial heavy lifting of configuring 
both GitHub Actions and IAM using the principle of least-privileged.

We will be performing the following steps:

1. Create application deployment targets
   * Create the dev pipeline stage
   * Create the prod pipeline stage
4. Create GitHub Actions Workflow
5. Deploy GitHub Actions Workflow

> **Note**
> The sam pipeline bootstrap command will guide you through a long series of questions. In this section of the lab, 
> it's critical to answer the questions as documented below.

### Create the dev stage of the pipeline

Run the final step to creating a CI/CD pipeline in GitHub Actions is to use a GitHub source repository and two 
deployment targets in a GitHub Actions workflow.

```shell
cd ~/environment/sam-app
sam pipeline bootstrap --stage dev
```

A list of the questions and required answers for this workshop are enumerated below. Pay particular attention to select 
OpenID Connect (OIDC) for the user permissions provider and GitHub Actions when asked to select an OIDC provider. 
**Note that numbers may be different when choosing from an enumerated list**. The full output and answers are provided below 
as an additional reference.

1. Select a pipeline template to get started: `AWS Quick Start Pipeline Templates` (1)
2. Select CI/CD system: `GitHub Actions` (3)
3. Do you want to go through stage setup process now? [Y/n]:
4. [1] Stage definition. Stage name: `dev`
5. [2] Account details. Select a credential source to associate with this stage: `default (named profile)` (2)
6. Enter the region in which you want these resources to be created: `Your region of choice`
7. **Select a user permissions provider**: OpenID Connect (OIDC) (2)
8. **Select an OIDC provider**: GitHub Actions (1)
9. Enter the URL of the OIDC provider [[https://token.actions.githubusercontent.com](https://token.actions.githubusercontent.com)]: `return/enter`
10. Enter the OIDC client ID (sometimes called audience) [sts.amazonaws.com]: `return/enter`
11. Enter the GitHub organization that the code repository belongs to. If there is no organization enter your username instead: `Your GitHub username`
12. Enter GitHub repository name: sam-app
13. Enter the name of the branch that deployments will occur from [main]: `return/enter`
14. Enter the pipeline execution role ARN if you have previously ... []: `return/enter`
15. Enter the CloudFormation execution role ARN if you have previously ... []: `return/enter`
16. Please enter the artifact bucket ARN for your Lambda function. If you do not ... []: `return/enter`
17. Does your application contain any IMAGE type Lambda functions? [y/N]: `N`
18. Press enter to confirm the values above ... : `return/enter`
19. Should we proceed with the creation? [y/N]: `y`

```
sam pipeline bootstrap --stage dev

sam pipeline bootstrap generates the required AWS infrastructure resources to connect
to your CI/CD system. This step must be run for each deployment stage in your pipeline,
prior to running the sam pipeline init command.

We will ask for [1] stage definition, [2] account details, and
[3] references to existing resources in order to bootstrap these pipeline resources.

[1] Stage definition
Stage configuration name: dev

[2] Account details
The following AWS credential sources are available to use.
To know more about configuration AWS credentials, visit the link below:
https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html
	1 - Environment variables (not available)
	2 - default (named profile)
	q - Quit and configure AWS credentials
Select a credential source to associate with this stage: 2
Associated account 123456789012 with configuration dev.

Enter the region in which you want these resources to be created [us-west-2]:
Select a user permissions provider:
	1 - IAM (default)
	2 - OpenID Connect (OIDC)
Choice (1, 2): 2
Select an OIDC provider:
	1 - GitHub Actions
	2 - GitLab
	3 - Bitbucket
Choice (1, 2, 3): 1
Enter the URL of the OIDC provider [https://token.actions.githubusercontent.com]:
Enter the OIDC client ID (sometimes called audience) [sts.amazonaws.com]:
Enter the GitHub organization that the code repository belongs to. If there is no organization enter your username instead: sguillory6
Enter GitHub repository name: the-complete-sam-workshop-github-actions
Enter the name of the branch that deployments will occur from [main]:

[3] Reference application build resources
Enter the pipeline execution role ARN if you have previously created one, or we will create one for you []:
Enter the CloudFormation execution role ARN if you have previously created one, or we will create one for you []:
Please enter the artifact bucket ARN for your Lambda function. If you do not have a bucket, we will create one for you []:
Does your application contain any IMAGE type Lambda functions? [y/N]: N

[4] Summary
Below is the summary of the answers:
	1 - Account: 123456789012
	2 - Stage configuration name: dev
	3 - Region: us-west-2
	4 - OIDC identity provider URL: https://token.actions.githubusercontent.com
	5 - OIDC client ID: sts.amazonaws.com
	6 - GitHub organization: your-github-username
	7 - GitHub repository: sam-app
	8 - Deployment branch:  main
	9 - Pipeline execution role: [to be created]
	10 - CloudFormation execution role: [to be created]
	11 - Artifacts bucket: [to be created]
	12 - ECR image repository: [skipped]
Press enter to confirm the values above, or select an item to edit the value:

This will create the following required resources for the 'dev' configuration:
	- IAM OIDC Identity Provider
	- Pipeline execution role
	- CloudFormation execution role
	- Artifact bucket
Should we proceed with the creation? [y/N]: y
	Updating the required resources...
```

Once this completes, you will see output which looks like the following:

```
Successfully updated!
The following resources were created in your account:
	- Pipeline execution role
	- CloudFormation execution role
	- Artifact bucket
	- IAM OIDC Identity Provider
View the definition in .aws-sam/pipeline/pipelineconfig.toml,
run sam pipeline bootstrap to generate another set of resources, or proceed to
sam pipeline init to create your pipeline configuration file.
```

These resources were created with a CloudFormation stack that SAM Pipelines synthesized and launched. 
You may optionally navigate to the CloudFormation console and inspect this stack to see everything that was created.

In this step, SAM pipelines created an IAM OIDC identity provider highlighted below. IAM OIDC identity providers are 
entities in IAM that describe an external identity provider (IdP) service that supports the OpenID Connect (OIDC) standard, 
such as Google or Salesforce. For our purposes, the external identity provider is GitHub. The IAM OIDC identity provider 
is used to establish trust between your AWS account and GitHub. This will allow GitHub Actions to assume an AWS IAM role 
for deployment activity.

![image_4.2.3.png](image_4.2.3.png)

Create the prod stage of the pipeline

Now that you have created the necessary resources for the dev build stage, you need to go through the same steps for the prod stage.

Run the following commands:

```shell
cd ~/environment/sam-app
sam pipeline bootstrap --stage prod
```

A list of the questions and required answers for this workshop are enumerated below. 
**Note that numbers may be different when choosing from an enumerated list**. The full output and answers are provided below 
as an additional reference.

1. [2] Account details. Select a credential source to associate with this stage: `default (named profile)` (2)
2. Enter the region in which you want these resources to be created: `Your region of choice`
3. Enter the pipeline execution role ARN if you have previously ... []: `return/enter`
4. Enter the CloudFormation execution role ARN if you have previously ... []: `return/enter`
5. Please enter the artifact bucket ARN for your Lambda function. If you do not ... []: `return/enter`
6. Does your application contain any IMAGE type Lambda functions? [y/N]: `N`
7. Press enter to confirm the values above ... : `return/enter`
8. Should we proceed with the creation? [y/N]: `y`

![image_4.2.4.png](image_4.2.4.png)

### Create GitHub Actions Workflow

Now that AWS SAM has created the required supporting resources, the final step to creating a CI/CD pipeline in GitHub 
Actions is to use a GitHub source repository and two deployment targets in a GitHub Actions workflow.
Once this is complete, SAM will launch a new CloudFormation stack. This stack will create new resources for your prod 
deployment. You can optionally inspect this template in the CloudFormation console. It looks very similar to the dev 
pipeline stack.

Run the following commands:

```shell
cd ~/environment/sam-app
sam pipeline init
```

A list of the questions and required anwers for this workshop is enumerated below. Note that numbers may be different 
when choosing from an enumerated list. The full output and answers is provided below as an additional reference.

1. Select a pipeline template to get started: AWS Quick Start Pipeline Templates (1)
2. Select CI/CD system: GitHub Actions (3)
3. What is the git branch used for production deployments? [main]: `return/enter`
4. What is the template file path? [template.yaml]: `return/enter`
5. Select an index or enter the stage 1's configuration name (as provided during the bootstrapping): `1`
6. What is the sam application stack name for stage 1? [sam-app]: `sam-app-dev`
7. Select an index or enter the stage 2's configuration name (as provided during the bootstrapping): `2`
8. What is the sam application stack name for stage 2? [sam-app]: `sam-app-prod`

```
sam pipeline init generates a pipeline configuration file that your CI/CD system
can use to deploy serverless applications using AWS SAM.
We will guide you through the process to bootstrap resources for each stage,
then walk through the details necessary for creating the pipeline config file.

Please ensure you are in the root folder of your SAM application before you begin.

Select a pipeline template to get started:
	1 - AWS Quick Start Pipeline Templates
	2 - Custom Pipeline Template Location
Choice: 1

Cloning from https://github.com/aws/aws-sam-cli-pipeline-init-templates.git (process may take a moment)
Select CI/CD system
	1 - Jenkins
	2 - GitLab CI/CD
	3 - GitHub Actions
	4 - Bitbucket Pipelines
	5 - AWS CodePipeline
Choice: 3
You are using the 2-stage pipeline template.
 _________    _________
|         |  |         |
| Stage 1 |->| Stage 2 |
|_________|  |_________|

Checking for existing stages...

2 stage(s) were detected, matching the template requirements. If these are incorrect, delete .aws-sam/pipeline/pipelineconfig.toml and rerun

This template configures a pipeline that deploys a serverless application to a testing and a production stage.

What is the git branch used for production deployments? [main]:
What is the template file path? [template.yaml]:
We use the stage configuration name to automatically retrieve the bootstrapped resources created when you ran `sam pipeline bootstrap`.

Here are the stage configuration names detected in .aws-sam/pipeline/pipelineconfig.toml:
	1 - dev
	2 - prod
Select an index or enter the stage 1's configuration name (as provided during the bootstrapping): 1
What is the sam application stack name for stage 1? [sam-app]: the-complete-sam-workshop-github-actions-dev
Stage 1 configured successfully, configuring stage 2.

Here are the stage configuration names detected in .aws-sam/pipeline/pipelineconfig.toml:
	1 - dev
	2 - prod
Select an index or enter the stage 2's configuration name (as provided during the bootstrapping): 2
What is the sam application stack name for stage 2? [sam-app]: the-complete-sam-workshop-github-actions-prod
Stage 2 configured successfully.

SUMMARY
We will generate a pipeline config file based on the following information:
	Select a user permissions provider.: OpenID Connect (OIDC)
	What is the git branch used for production deployments?: main
	What is the template file path?: template.yaml
	Select an index or enter the stage 1's configuration name (as provided during the bootstrapping): 1
	What is the sam application stack name for stage 1?: the-complete-sam-workshop-github-actions-dev
	What is the pipeline execution role ARN for stage 1?: arn:aws:iam::750353892954:role/aws-sam-cli-managed-dev-pipe-PipelineExecutionRole-1MY2CND2E58CK
	What is the CloudFormation execution role ARN for stage 1?: arn:aws:iam::750353892954:role/aws-sam-cli-managed-dev-p-CloudFormationExecutionR-11CGGAM8W2IXZ
	What is the S3 bucket name for artifacts for stage 1?: aws-sam-cli-managed-dev-pipeline-artifactsbucket-1ppvh61mp2vj1
	What is the ECR repository URI for stage 1?:
	What is the AWS region for stage 1?: us-west-2
	Select an index or enter the stage 2's configuration name (as provided during the bootstrapping): 2
	What is the sam application stack name for stage 2?: the-complete-sam-workshop-github-actions-prod
	What is the pipeline execution role ARN for stage 2?: arn:aws:iam::750353892954:role/aws-sam-cli-managed-prod-pip-PipelineExecutionRole-152A2QJI3ORPD
	What is the CloudFormation execution role ARN for stage 2?: arn:aws:iam::750353892954:role/aws-sam-cli-managed-prod-CloudFormationExecutionR-O4DKAM9I6JS0
	What is the S3 bucket name for artifacts for stage 2?: aws-sam-cli-managed-prod-pipeline-artifactsbucket-nrg2vlxhrjg2
	What is the ECR repository URI for stage 2?:
	What is the AWS region for stage 2?: us-west-2
Successfully created the pipeline configuration file(s):
	- .github/workflows/pipeline.yaml
What we've just done is create the GitHub Actions workflow to create a full CI/CD Pipeline using GitHub Actions and other AWS services.

Your project should have the structure below (only the most relevant files and folders are shown).

└── sam-app
    ├── .github/workflows
    └──── pipeline.yaml         # (new) GitHub Action Workflow
    ├── events
    ├── hello-world/            # SAM application root
    ├── README.md
    ├── samconfig.toml          # Config file for manual SAM deployments
    └── template.yaml           # SAM template
```

You can optionally open up `pipeline.yaml` and other files to see what SAM Pipelines created for us. Looking at 
`pipeline.yaml` you can see that there are nearly 250 lines of GitHub Action workflow that SAM created. Think about 
how much time you just saved using SAM Pipelines rather than crafting this by hand!

> **We haven't created any CI/CD systems just yet!** You'll do that next. First, you need to commit your CI/CD 
> configuration files into your repository. Once that's done, your GitHub Action workflow will be created.

### Deploy GitHub Actions Workflow

To deploy the GitHub Actions workflow, we need to commit the new CI/CD configuration file to our repository and push to 
GitHub. Check the repository status in the terminal.

```shell
cd ~/environment/sam-app
git add .
git commit -m "Adding SAM CI/CD Pipeline definition"
git push
```

Now that the configuration has been pushed to your GitHub repository, GitHub Actions will take over, create, and execute 
your very first workflow based on the pipeline.yaml you committed.

## Inspect the pipeline

You will now have a new GitHub Actions Workflow. Navigate to the GitHub Actions Dashboard (replace YOUR_USER with your GitHub user):

https://github.com/YOUR_USER/sam-app/actions/workflows/pipeline.yaml 

You should see a single Workflow. If you've navigated here soon after pushing, then you will see your new Workflow 
executing its first run.

![image_4.2.5.png](image_4.2.5.png)

Let your workflow run every stage. After it finishes, all stages will be green.

![image_4.2.6.png](image_4.2.6.png)

> **Note**
> You may have noticed our sam-app-dev stage is deployed in a GitHub Actions job named deploy-testing. 
> The deploy-testing name is hardcoded in the pipeline.yaml file and does not relate in any way to our dev 
> application that we named `sam-app-dev`. When you use this template outside this workshop, you can rename deploy-testing 
> to anything appropriate.

### Test the application in dev and prod environments

Navigate to the CloudFormation console. After your first Workflow run has finished, you will notice two new stacks named 
am-app-dev and sam-app-prod. These are the names you provided during the SAM Pipelines wizard in the previous step.

GitHub Actions created the sam-app-dev stack during the deploy-testing Pipeline step. Similarly, GitHub Actions created 
sam-app-prod during the deploy-prod step.

Look at the Outputs tab for each of these CloudFormation stacks to see the API endpoints. You can use curl or other 
methods to verify the functionality of your two new APIs. You can export the URL endpoints for both stages in a terminal.

```shell
export DEV_ENDPOINT=$(aws cloudformation describe-stacks --stack-name sam-app-dev | jq -r '.Stacks[].Outputs[].OutputValue | select(startswith("https://"))')
export PROD_ENDPOINT=$(aws cloudformation describe-stacks --stack-name sam-app-prod | jq -r '.Stacks[].Outputs[].OutputValue | select(startswith("https://"))')

echo "Dev endpoint: $DEV_ENDPOINT"
echo "Prod endpoint: $PROD_ENDPOINT"

curl -s $DEV_ENDPOINT
curl -s $PROD_ENDPOINT
```

![image_4.2.7.png](image_4.2.7.png)

You may have noticed that unit tests are not being run in your pipeline. Let's fix that in the next section!

## Enable unit tests

To enable unit tests in your pipeline there are two steps.

1. Edit `pipeline.yaml` and add commands to the test build step.
2. Commit and push these changes.

Open up the `~/environment/sam-app/.github/workflows/pipeline.yaml` file in your editor. Search for the string `test:`. 
You will find a comment that shows where you can add commands to run your unit tests. Please add the commands that 
are appropriate for your chosen runtime.

```yaml
test:
  if: github.event_name == 'push'
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v3
    - run: |
        # trigger the tests here
        echo 'Running unit tests'
        cd hello-world
        npm install
        npm run test
```

### Commit and push changes

```shell
cd ~/environment/sam-app
git commit -am 'Enable unit tests in pipeline'
git push
```

### Watch the Workflow self-update and run tests

Open up your Workflow from the GitHub Actions dashboard. If you expand the test job, then you will see output from 
your new unit tests. Remember any change you make to the pipeline.yaml will be applied automatically once you commit.

![image_4.2.8.png](image_4.2.8.png)

Congratulations! You have created a CI/CD pipeline for a Serverless application!


### Edit pipeline.yaml

### 4.2. Configuración del repositorio (por ejemplo, Bitbucket) 

### 4.3. Configuración del entorno para la implementación 

### 4.4. Integración con AWS CodePipeline