# Настройка Canary и Linear deployment с SAM

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


A Canary Deployment is a technique that reduces the risk of deploying a new version of an application by slowly rolling 
out the changes to a small subset of users before rolling it out to the entire customer base.

![image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/canaries/canary-deployments.png)

## How canaries work

> **Note**
[Gradual deployment with canaries](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/automating-updates-to-serverless-apps.html) 
> is a native feature of SAM and does not require a CI/CD pipeline. This module does show screenshots of the CI/CD pipeline 
> created in the CodePipeline module. If you didn't create a CodePipeline you can still work through this module. 
> Know that you can still view the deployment status in AWS CodeDeploy, but you will not have a pipeline to inspect.

Before we jump into implementation, let's first understand how Canary Deployments work:

The concepts of blue/green and canary deployments have been around for a while and have been well-established as 
best-practices for reducing the risk of software deployments. In traditional applications, you slowly and incrementally 
update the servers in your fleet while simultaneously verifying application health. However, there is somewhat of an 
impedance mismatch when mapping these concepts to a serverless world. You can’t incrementally deploy your software 
across a fleet of servers when there are no servers!

Fortunately, there are a number of services and features involved to make this possible when operating serverless workloads.

### Lambda versions and aliases

AWS Lambda allows you to publish multiple versions of the same function. Each version has its own code and associated 
dependencies, as well as its own function settings (like memory allocation, timeout, and environment variables). 
You can refer to a given version by using a Lambda Alias. An alias is nothing but a name that can be pointed to a given 
version of a Lambda function.

1[image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/canaries/lambda-versions-aliases.png)

### Traffic shifting with Lambda aliases

With the introduction of alias traffic shifting, it is now possible to trivially implement canary deployments of 
Lambda functions. By updating additional version weights on an alias, invocation traffic is routed to the new function 
versions based on the weight specified. Detailed CloudWatch metrics for the alias and version can be analyzed 
during the deployment, or other health checks performed, to ensure that the new version is healthy before proceeding.

![image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/canaries/traffic-shifting.png)

### Traffic shifting with SAM and CodeDeploy

AWS CodeDeploy provides an intuitive turn-key implementation of this functionality integrated directly into AWS SAM. 
Traffic-shifted deployments can be declared in a SAM template and CodeDeploy manages the function rollout as part of 
the CloudFormation stack update. If something goes wrong, CloudWatch alarms can be configured to trigger a stack rollback.

![image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/canaries/traffic-shifting-codedeploy.png)

## Update SAM template

Open the SAM template (`sam-app/template.yaml`) in your project and add the `AutoPublishAlias` and `DeploymentPreference` 
blocks into the `HelloWorldFunction` properties section.

`~/environment/sam-app/template.yaml`

```yaml
Runtime: nodejs16.x
AutoPublishAlias: live
DeploymentPreference:
  Type: Canary10Percent5Minutes
Architectures:
  - x86_64
```

### Deployment Preference Types

For this workshop, we are using the Canary10Percent5Minutes strategy, which means that traffic is shifted in two increments. 
In the first increment, only 10% of the traffic is shifted to the new Lambda version, and after 5 minutes, the remaining 
90% is shifted. There are other deployment strategies you can choose in CodeDeploy:

* AllAtOnce
* Canary10Percent5Minutes
* Canary10Percent10Minutes
* Canary10Percent15Minutes
* Canary10Percent30Minutes
* Linear10PercentEvery1Minute
* Linear10PercentEvery2Minutes
* Linear10PercentEvery3Minutes
* Linear10PercentEvery10Minutes

The `Linear` strategy means that traffic is shifted in batches every X minutes. For example, `Linear10PercentEvery10Minutes` 
will shift an additional 10% of traffic every 10 minutes. The entire deployment will take approximately 100 minutes 
(more likely closer to 90 since the first batch is shifted when the deployment begins).

### Validate the SAM template

Run the following command in your terminal:

```shell
cd ~/environment/sam-app
sam validate --lint
```

If the template is correct, you will see `template.yaml is a valid SAM Template`. If you see an error, then you likely 
have an indentation issue on the YAML file. Double check and make sure it matches the screenshot shown above.

The open-source tool AWS CloudFormation Linter (cfn-lint) validates AWS SAM templates with the AWS SAM CLI by running 
sam `validate` with the `--lint` option. cfn-lint performs detailed validation on AWS CloudFormation templates using 
rules guided by the AWS CloudFormation resource specification and custom rules defined in a configuration file to 
validate against.

### Push the changes

In the terminal, run the following commands from the root directory of your `sam-app` project.

```shell
git add .
git commit -m "Add Canary deployment configuration to SAM"
git push
```

## Gradually deploy an update

Our pipeline is setup to deploy new code using the Canary10Percent5Minutes strategy. To see this in action, we need to 
push an update to our application.

### Make a code change

AWS SAM will not deploy anything when your code hasn't changed. Since our pipeline is using AWS SAM as the deployment 
tool, we need to make some changes to the application.

Change the message in your Lambda function's response code `I'm using canary deployments`. Remember to update the unit 
tests!

`~/environment/sam-app/hello-world/app.js`

```shell
response = {
  statusCode: 200,
  body: JSON.stringify({
    message: "I'm using canary deployments",
  }),
}
```

`~/environment/node-sam-app/hello-world/tests/unit/test-handler.js`

```shell
describe("Tests index", function () {
  it("verifies successful response", async () => {
    const result = await app.lambdaHandler(event, context)

    expect(result).to.be.an("object")
    expect(result.statusCode).to.equal(200)
    expect(result.body).to.be.an("string")

    let response = JSON.parse(result.body)

    expect(response).to.be.an("object")
    expect(response.message).to.be.equal("I'm using canary deployments")
  })
})
```

### Push the code

```shell
git add .
git commit -m "Changed return message"
git push
```

### Watch the canary

It will take a few minutes for your pipeline to get to the `DeployTest` stage which will start the canary deployment 
of the stage you named `sam-app-dev`.

Let's start a watcher which outputs your API's message every second and helps you notice when traffic shifting starts 
and completes. Hit the dev API endpoint every second with curl and print the return value to the screen. This command 
also appends the output to the outputs.txt file that you can inspect later. You can run this command from any directory.

```shell
watch -n 1 "curl -s $DEV_ENDPOINT | jq '.message' 2>&1 | tee -a outputs.txt"
```

You should see `Hello my friend` in the terminal.

If you completed the [CI/CD with CodePipeline module](), turn your attention to the CodePipeline console. Wait for your 
pipeline to get to the DeployTest stage. Once you see the stage turn blue with the In Progress status, navigate to the 
CodeDeploy console. If you didn't deploy a CI/CD pipline, look at the CodeDeploy console.

![image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/canaries/screenshot-canary-codedeploy-00.png)

In the CodeDeploy console, click on Deployments. You should see your deployment In progress. If you do not see 
a deployment, click the refresh icon. This may take a few minutes to show up! Click on 
the Deployment Id to see the details.

![image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/canaries/screenshot-canary-codedeploy-0.png)

The deployment status shows that 10% of the traffic has been shifted to the new version (the Canary). 
CodeDeploy will hold the remaining percentage until the specified time interval has ellapsed. 
In this case, we specified the interval to be 5 minutes.

![image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/canaries/screenshot-canary-codedeploy-1.png)

When you are in this stage, take a look at your terminal where you started the watch command. You will see 
the message occasionally flash to `I'm using canary deployments`.

![image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/canaries/code-pipeline-canary.gif)

After five minutes, CodeDeploy will shift the remaining traffic to the new version and the deployment will be done.

![image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/canaries/screenshot-canary-codedeploy-2.png)

In the terminal where you ran the watch command, press Ctrl-C to stop. Count up the number of messages for each message 
string to validate the traffic was shifted gradually. Of course, the ratio of new to old return messages depends on when 
you started and stopped the watch command.

```shell
sort outputs.txt  | uniq -c
```

```
228 "hello my friend"
84 "I'm using canary deployments"
```

You can also see the sequence of return values by opening `outputs.txt`.

```
cat outputs.txt
```

## Monitor canary health

Canary deployments are considerably more successful if the code is being monitored during the deployment. 
You can configure CodeDeploy to automatically roll back the deployment if a specified CloudWatch metric has breached 
a threshold. Common metrics to monitor are Lambda Invocation errors or Invocation Duration (latency), for example.

### Define a CloudWatch Alarm

Add the following alarm definition to the `template.yaml` file in the Resources section after 
the `HelloWorldFunction` definition.

This block defines an Amazon CloudWatch Alarm. The alarm is triggered when the Lambda function throws errors. 
The alarm threshold is crossed when there is one or more errors in a given minute, for two consecutive minutes.

```yaml
CanaryErrorsAlarm:
  Type: AWS::CloudWatch::Alarm
  Properties:
    AlarmDescription: Lambda function canary errors
    ComparisonOperator: GreaterThanThreshold
    EvaluationPeriods: 2
    MetricName: Errors
    Namespace: AWS/Lambda
    Period: 60
    Statistic: Sum
    Threshold: 0
    Dimensions:
      - Name: Resource
        Value: !Sub "${HelloWorldFunction}:live"
      - Name: FunctionName
        Value: !Ref HelloWorldFunction
      - Name: ExecutedVersion
        Value: !GetAtt HelloWorldFunction.Version.Version
```

### Enable canary and alarm for production

Alarms and canaries are great for our production deployment. You may not want or need to use canary deployments for 
non-production environments. Using an `AllAtOnce` strategy for our development stage will make deployments faster. 
Let's configure our serverless application to use a canary deployment and the new CloudWatch alarm only for 
the `sam-app-prod` stage using a CloudFormation `Condition`.

First, create a `IsProduction` Condition statement after the `Globals` section near the top of `template.yaml`.

Next, change the `DeploymentPreference` to use this new `IsProduction` condition.

```yaml
DeploymentPreference:
  Type: !If [IsProduction, "Canary10Percent5Minutes", "AllAtOnce"]
  Alarms: !If [IsProduction, [!Ref CanaryErrorsAlarm], []]
```

Your `template.yaml` should look like this:

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Transform: AWS::Serverless-2016-10-31
Description: Sample SAM Template for sam-app

Globals:
  Function:
    Timeout: 3

Conditions:
  IsProduction: !Equals [!Ref "AWS::StackName", "sam-app-prod"]

Resources:
  HelloWorldFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: hello-world/
      Handler: app.lambdaHandler
      Runtime: nodejs16.x
      AutoPublishAlias: live
      DeploymentPreference:
        Type: !If [IsProduction, "Canary10Percent5Minutes", "AllAtOnce"]
        Alarms: !If [IsProduction, [!Ref CanaryErrorsAlarm], []]
      Events:
        HelloWorld:
          Type: Api
          Properties:
            Path: /hello
            Method: get

  CanaryErrorsAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmDescription: Lambda function canary errors
      ComparisonOperator: GreaterThanThreshold
      EvaluationPeriods: 2
      MetricName: Errors
      Namespace: AWS/Lambda
      Period: 60
      Statistic: Sum
      Threshold: 0
      Dimensions:
        - Name: Resource
          Value: !Sub "${HelloWorldFunction}:live"
        - Name: FunctionName
          Value: !Ref HelloWorldFunction
        - Name: ExecutedVersion
          Value: !GetAtt HelloWorldFunction.Version.Version

Outputs:
  # ServerlessRestApi is an implicit API created out of Events key under Serverless::Function
  # Find out more about other implicit resources you can reference within SAM
  # https://github.com/awslabs/serverless-application-model/blob/master/docs/internals/generated_resources.rst#api
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

## Introduce an error

Monitoring the health of your canary allows CodeDeploy to make a decision to whether a rollback is needed. 
If our CloudWatch Alarm gets to ALARM status, CodeDeploy will roll back the deployment automatically.

### Introduce an error on purpose

Let's break the Lambda function on purpose so that the `CanaryErrorsAlarm` is triggered during deployment. 
Update the Lambda code to throw an error on every invocation. Make sure to update the unit test, otherwise 
the build will fail.

`~/environment/sam-app/hello-world/app.js`

```javascript
let response

exports.lambdaHandler = async (event, context) => {
  throw new Error("This will cause a deployment rollback")
  // try {
  //     response = {
  //         "statusCode": 200,
  //         "body": JSON.stringify({
  //             message: "I'm using canary deployments",
  //         })
  //     }
  // } catch (err) {
  //     console.log(err);
  //     return err;
  // }

  // return response
}
```

```javascript
// 'use strict';

// const app = require('../../app.js');
// const chai = require('chai');
// const expect = chai.expect;
// var event, context;

// describe('Tests index', function () {
//     it('verifies successful response', async () => {
//         const result = await app.lambdaHandler(event, context)

//         expect(result).to.be.an('object');
//         expect(result.statusCode).to.equal(200);
//         expect(result.body).to.be.an('string');

//         let response = JSON.parse(result.body);

//         expect(response).to.be.an('object');
//         expect(response.message).to.be.equal("hello my friend with canaries");
//     });
// });
```

### Push the changes

In the terminal, run the following commands from the root directory of your `sam-app` project.

```shell
cd ~/environment/sam-app
git add .
git commit -m "Breaking the lambda function on purpose"
git push
```

### Generate traffic

Once you've pushed the code, you will need to generate traffic on your production API Gateway endpoint. 
**If you don't generate traffic for your Lambda function, the CloudWatch alarm will not be triggered!**

##### If you haven't exported the PROD_ENDPOINT, run the following command. {collapsible="true"}
```shell
export PROD_ENDPOINT=$(aws cloudformation describe-stacks --stack-name sam-app-prod | jq -r '.Stacks[].Outputs[].OutputValue | select(startswith("https://"))')
echo "$PROD_ENDPOINT"
```

Start a `watch` command which will hit this endpoint twice per second.

```shell
watch -n 0.5 "curl -s $PROD_ENDPOINT"
```

## Monitor rollback

_If you created a CI/CD pipeline_, navigate to your pipeline in the CodePipeline console and keep an eye on 
its progression. You will see that the `DeployTest` stage is deployed quickly since it's using the `AllAtOnce` strategy. 
Even though our code is broken, thanks to our hard coded error, it's deployed automatically in the `DeployTest` stage.

![image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/canaries/canary-deployment-dev-success.png)

Once your pipeline moves to the `DeployProd` stage, things gets more interesting. In the terminal window where you are 
running the `watch` command, you'll notice the message flash from `I'm using canary deployments` to `Internal server error`.

![image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/canaries/code-pipeline-errors.gif)

After a few minutes, you will see CodeDeploy mark this deployment as failed and roll back to the previous version. 
The Internal server error messages will go away as all traffic is shifted back to the previous version.

![image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/canaries/canary-deployment-prod-rollback.png)

Navigate to the [AWS CodeDeploy Console](https://console.aws.amazon.com/codedeploy/home) Deployments page. 
Look at the Deployment which may be In-Progress or Stopped, depending on whether the rollback is complete. 
Click on the Deployment Id to see its details.

You will see that CodeDeploy detected `CanaryErrorsAlarm` has triggered and stopped the deployment.

![image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/canaries/screenshot-codedeploy-rollback.png)


