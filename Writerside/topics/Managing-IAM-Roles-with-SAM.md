# Managing IAM Roles with SAM

![image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/connectors/what-is-an-iam-role.png)

So far in this workshop, we haven't discussed what the Lambda function is allowed to do or how to control access 
to other AWS services or resources. Because the sample application and Lambda function merely returns 
a `Hello World!` message you may assume that it doesn't depend on other AWS services. This is a natural assumption, 
but the function does rely on at least one other AWS service. Can you guess which one?

In this module, you'll learn different ways to grant least-privledge access to AWS resources from your Lambda functions. 
By the end of this module you will have an understanding of the three different mechanisms to manage Lambda's access:

* AWS SAM Policy Templates
* AWS SAM Connectors
* Handwritten IAM policies

## IAM Execution Role

Before we get started, let's cover how Lambda is able, or not able, to access different AWS services and resources.

Below is the `template.yaml` file from the hellow world example from Module 1. Notice that you don't see any explicit 
permissions for the Lambda function:

```yaml
Resources:
  HelloWorldFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: hello-world/
      Handler: app.lambdaHandler
      Runtime: nodejs16.x
      Events:
        HelloWorld:
          Type: Api
          Properties:
            Path: /hello
            Method: get
```

By default, Lambda functions write logs to to AWS CloudWatch Logs. If Lambda writes to CloudWatch without explicit 
permission from you, how is this working?

Every Lambda function has an AWS Identity and Access Management (IAM) role called an execution role. 
An execution role grants the function permission to access AWS services and resources. In the "Hello World" 
example that you have been using, SAM has created an execution role for you with the minimum permissions 
to access CloudWatch Logs.

Let's go have a look at that function's execution role in the AWS console.

> **Note**
> If you haven't worked Module 1 that is fine, just follow along with the screen shots below.

1. Open the [Functions page of the Lambda console](https://console.aws.amazon.com/lambda/home#/functions).
2. Locate the function in the list (use the search bar if necessary) and click on it.

![image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/connectors/select-lambda-function.png)

3. Click the "Configuration" tab, and then choose "Permissions" on the left menu.

![image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/connectors/select-configuration-tab.png)

![image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/connectors/select-permissions.png)

4. In the Resource summary section, choose a service from the dropdown list to see permissions related 
to that service. In this instance, there should only be one service listed, Amazon CloudWatch Logs.

![image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/connectors/resource-summary-1.png)

5. Under Resource summary, review the services and resources that the function can access. In the information alert 
box we see that these permissions come from an AWS Managed Policy, `AWSLambdaBasicExecutionRole`

![image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/connectors/resource-summary-2.png)

6. Finally, click on the execution role link to open the IAM console.

![image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/connectors/open-iam-console.png)

![image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/connectors/lambda-execution-role.png)

This IAM Role defines what your Lambda function can access. IAM works on the principle of least-privledge: 
without explict permissions, your Lambda function will not be able to access other AWS resources. The execution 
role can have zero or more policies attached to it. In the coming sections you'll learn how to manage this execution 
role and add your own policies.

In this example, there is a single policy named `AWSLambdaBasicExecutionRole`. `AWSLambdaBasicExecutionRole` is 
an [AWS managed policy](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-vs-inline.html#aws-managed-policies) 
that is created and administered by AWS. AWS managed policies are designed to provide 
permissions for many common use cases. Every Lambda function needs a minimum set of permissions to use 
CloudWatch logs. The `AWSLambdaBasicExecutionRole` defines these permissions which, by default, apply to 
all Lambda functions.

But, what happens when your Lambda function needs access to other AWS resources like a DynamoDB table? 
The answer is that the function will require additional permissions, such as `dynamodb:Query` or `dynamodb:Scan`.

![image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/connectors/lambda-dynamodb-permissions.png)

To allow the Lambda function to access DynamoDB, you would need to augment the function definition in 
`template.yaml` by adding the necessary permissions. Once the correct `dynamoddb:` permissions are added 
the function's execution role, your function would be able to perform actions on your DynamoDB table.

To demonstrate the various options for specifying IAM permissions in SAM, you will update the sample 
application so that it reads and writes from and to a DynamoDB table.

![image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/connectors/sam-workshop-connectors-architecture.png)

## Initialize project

> **Be sure to select the correct option**
> We show the Serverless API option as option 3 below. As new quick start templates are released, the exact location 
> of this option may change. If you are interested in exploring the quick start templates, you can find them at 
> [AWS SAM CLI Application Templates](https://github.com/aws/aws-sam-cli-app-templates) 

Run the following command to scaffold a new project:

```shell
sam init
```

In the wizard, select

* `AWS QuickSart Templates` and
* `Serverless API`

Do **not** use the shortcut to use the latest Python version. _Note that the numbered answers may be different in your terminal._

```
Which template source would you like to use?
        1 - AWS Quick Start Templates <---- SELECT THIS
        2 - Custom Template Location
Choice: 1

Choose an AWS Quick Start application template
        1 - Hello World Example
        2 - Data processing
        3 - Hello World Example with Powertools for AWS Lambda
        4 - Multi-step workflow
        5 - Scheduled task
        6 - Standalone function
        7 - Serverless API <---- SELECT THIS TEMPLATE
        ...
Template: 7

Use the most popular runtime and package type? (Python and zip) [y/N]: n
```

Next, select your preferred runtime and version. For this Module, choose (3) `nodejs20.x`:

```
Which runtime would you like to use?
        1 - dotnet6
        2 - nodejs20.x
        3 - nodejs18.x
        4 - nodejs16.x
Runtime: 2
```

Select no for X-ray tracing and leave `sam-app` as the `Project name`.

```
Based on your selections, the only Package type available is Zip.
We will proceed to selecting the Package type as Zip.

Based on your selections, the only dependency manager available is npm.
We will proceed copying the template using npm.

Would you like to enable X-Ray tracing on the function(s) in your application?  [y/N]: n

Project name [sam-app]:
```

You should now see a new folder, `sam-app`, containing the generated scaffolding for the project. 
Notice that there are three JavaScript files, each which represents a Lambda function and specific responsibility.

* `get-all-items.js` -> retrieves all items stored in the DynamoDB table
* `get-by-id.js` -> retrieves an item by its unique ID
* `put-item.js` -> write an item to the DynamoDB table

![image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/connectors/project-structure.png)

Feel free to open up any of these files to see how they use the AWS SDK is used to access DynamoDB.

## Deploy the app

Since this is the first time you are deploying this application, you will use the `--guided` flag with the `sam deploy` command.

```shell
cd ~/environment/sam-app
sam build
sam deploy --guided
```

> **Missing authorization**
> Make sure to answer y to the question about the Lambda functions not having authorization defined: 
> `getAllItemsFunction may not have authorization defined, Is this okay? [y/N]: y`

### Deployment completed

This command will take a few minutes to finish because it is creating 
the resources (Lambda functions, API Gateway and IAM roles, etc.). After the deployment 
completes successfully you will see an output similar to the following:

```
CloudFormation outputs from deployed stack
---------------------------------------------------------------------------------------------------------------------------
Outputs
---------------------------------------------------------------------------------------------------------------------------
Key                 WebEndpoint
Description         API Gateway endpoint URL for Prod stage
Value               https://q9kbh1buk3.execute-api.us-west-2.amazonaws.com/Prod/
---------------------------------------------------------------------------------------------------------------------------
```

Export the new HTTPS endpoint:

```shell
export ENDPOINT=$(aws cloudformation describe-stacks --stack-name sam-app | \
    jq -r '.Stacks[].Outputs[].OutputValue | select(startswith("https://"))')
```

### Test the endpoint

As a sanity check, make a request to your new serverless application to make sure it's working. 
Since your new API is reading from a DynamoDB table, you can expect and empty response.

```shell
curl -s $ENDPOINT
```

The response will be an empty array.

```shell
[]
```

### Add data

Add a few items to the DynamoDB table so that you will have some data to query. This executes the putItem API with three unique items.

```shell
curl -d "{\"id\": \"id1\",\"name\": \"name1\"}" -X POST $ENDPOINT
curl -d "{\"id\": \"id2\",\"name\": \"name2\"}" -X POST $ENDPOINT
curl -d "{\"id\": \"id3\",\"name\": \"name3\"}" -X POST $ENDPOINT
```

Use the `getAllItems` API to see the items you just added:

```shell
curl -s $ENDPOINT | jq
```

You should see a list of items.

```json
[
  {
    "id": "id3",
    "name": "name3"
  },
  {
    "id": "id1",
    "name": "name1"
  },
  {
    "id": "id2",
    "name": "name2"
  }
]
```

Finally, fetch individual items with the `getItemById` API:

```shell
curl -s $ENDPOINT/id1 | jq
curl -s $ENDPOINT/id2 | jq
curl -s $ENDPOINT/id3 | jq
```

## SAM Policy Templates

[SAM Policy Templates](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-policy-templates.html) 
are shortcuts that make it easy to add scoped permissions to Lambda functions and 
Step Functions workflows. These are specific to SAM, and can make the work of IAM permissions management easier. 
The AWS SAM team maintains a number of policy templates for common integrations like DynamoDB and S3. 
In this module you'll learn how SAM Policy Templates work, and how to use them in your own applications.

### Your first policy template

In the last section to used your API to read and write data from and to a DynamoDB table. 
SAM automatically creates a basic execution role for each Lambda function in your SAM application 
that allows the functions to access CloudWatch Logs. But in this new application, how did 
the Lambda functions have access to DyanamoDB?

Open the `template.yaml` file and look at the Policies section in the getAllItemsFunction definition.

```yaml
 # This is a Lambda function config associated with the source code: get-all-items.js
  getAllItemsFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: src/handlers/get-all-items.getAllItemsHandler
      Runtime: nodejs18.x
      Architectures:
        - x86_64
      MemorySize: 128
      Timeout: 100
      Description: A simple example includes a HTTP get method to get all items from a DynamoDB table.
      Policies:
        # Give Create/Read/Update/Delete Permissions to the SampleTable
        - DynamoDBCrudPolicy:
            TableName: !Ref SampleTable
      Environment:
        Variables:
          # Make table name accessible as environment variable from function code during execution
          SAMPLE_TABLE: !Ref SampleTable
      Events:
        Api:
          Type: Api
          Properties:
            Path: /
            Method: GET
```

Notice the `DynamoDBCrudPolicy` listed under the `Policies` propery. The key name, `DynamoDBCrudPolicy`, 
refers to a [specific policy template](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-policy-template-list.html#dynamo-db-crud-policy). 
This policy template expects a value for the `TableName` variable 
which SAM uses to add a number IAM permissions to your Lambda's execution role. As you may guess, 
SAM adds create, read, update and delete access (CRUD) so that the function can perform 
the following actions on the `SampleTable` DynamoDB table:

* put items
* get items
* scan items

### Remove permissions

Next, you'll remove these permissions and see the effects.

Take away the the `DynamoDBCrudPolicy` property by commenting it out in the template.yaml file.

```yaml
# This is a Lambda function config associated with the source code: get-all-items.js
getAllItemsFunction:
  Type: AWS::Serverless::Function
  Properties:
    Handler: src/handlers/get-all-items.getAllItemsHandler
    # ...
    #      Policies:
    #        - DynamoDBCrudPolicy:
    #            TableName: !Ref SampleTable
    # ...
```

Next, make the following changes to the get-all-items.js Lambda function code. In the catch block, append the 
exception to the response so that we will have better diagnostic information if there is an error. 
The highlighted lines show the changes to make. It's also ok to copy and paste the entire code block below 
and replace your existing function.

```javascript
// Create clients and set shared const values outside of the handler.

// Create a DocumentClient that represents the query to add an item
import { DynamoDBClient, ResourceNotFoundException } from '@aws-sdk/client-dynamodb';
import { DynamoDBDocumentClient, ScanCommand } from '@aws-sdk/lib-dynamodb';
const client = new DynamoDBClient({});
const ddbDocClient = DynamoDBDocumentClient.from(client);

// Get the DynamoDB table name from environment variables
const tableName = process.env.SAMPLE_TABLE;

// A simple example includes a HTTP get method to get all items from a DynamoDB table.
export const getAllItemsHandler = async (event) => {
    if (event.httpMethod !== 'GET') {
        throw new Error(`getAllItems only accept GET method, you tried: ${event.httpMethod}`);
    }
    // All log statements are written to CloudWatch
    console.info('received:', event);

    // get all items from the table (only first 1MB data, you can use `LastEvaluatedKey` to get the rest of data)
    // https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/DynamoDB/DocumentClient.html#scan-property
    // https://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_Scan.html
    var params = {
        TableName : tableName
    };

    let response = {}

    try {
        const data = await ddbDocClient.send(new ScanCommand(params));
        response = {
            statusCode: 200,
            body: JSON.stringify(data.Items)
        };
    } catch (ResourceNotFoundException) {
        response = {
            statusCode: 404,
            body: "Unable to call DynamoDB. Table resource not found. " + ResourceNotFoundException
        };
    };

    // All log statements are written to CloudWatch
    console.info(`response from: ${event.path} statusCode: ${response.statusCode} body: ${response.body}`);
    return response;
}
```

Next, build and deploy the application.

```shell
cd ~/environment/sam-app
sam build && sam deploy
```

After the deployment completes, make a request to the getAllItems API. You'll see an error that looks similar 
to the following, with details for your Lambda function and DynamoDB table:

```shell
curl -s $ENDPOINT
```

```
Unable to call DynamoDB. Table resource not found. AccessDeniedException: ...
is not authorized to perform: dynamodb:Scan on resource: ... because no identity-based policy allows the dynamodb:Scan action
```

As expected, the `getAllItemsFunction` Lambda function throws an error because it no longer has permission to scan the DynamoDB table.

Fix the function by uncommenting the `DynamoDBCrudPolicy` so that it again has the required permissions.

> **Note**
> To understand SAM Policy Templates, you need some familiarity with [IAM Policy syntax](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies.html). 
> While SAM's Policy Templates are convenient they are not a substitute for reading and understanding IAM policies.

### Least privledges

Open the other two JavaScript files and look at the Policies statements. Notice that they also reference 
the DynamoDBCrudPolicy. Each function needs specific permissions (read, write or scan) to do their jobs, 
but DynamoDBCrudPolicy grants all of these permissions. We can restrict access to the DyanmoDB table 
further with different SAM policy templates while still allowing each function to do their jobs. If you look 
at the list of SAM Policy templates, there are some more appropriate templates for scoping the permissions 
for our three functions:

| Function       | SAM Policy Template |
|----------------|---------------------|
| put-item       | DynamoDBWritePolicy |
| get-item-by-id | DynamoDBReadPolicy  |
| get-all-items  | DynamoDBReadPolicy  |

In the `template.yaml` file, change the Policy of each function to the more narrowly scoped policy listed above. 
Note that only the changes are shown in the yaml below.

```yaml
 getAllItemsFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: src/handlers/get-all-items.getAllItemsHandler
      ...
      Policies:
        - DynamoDBReadPolicy:
            TableName: !Ref SampleTable
      ...

  getByIdFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: src/handlers/get-by-id.getByIdHandler
      ...
      Policies:
        - DynamoDBReadPolicy:
            TableName: !Ref SampleTable
      ...

  putItemFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: src/handlers/put-item.putItemHandler
      ...
      Policies:
        - DynamoDBWritePolicy:
            TableName: !Ref SampleTable
      ...
```

After you've made these changes redeploy the application and verify that our APIs still work.

```shell
cd ~/environment/sam-app
sam build && sam deploy
```

Test the put-items api first:

```shell
curl -d "{\"id\": \"id4\",\"name\": \"name4\"}" -X POST $ENDPOINT
```

Next, test the get-all-items API:

```shell
curl -s $ENDPOINT | jq 'sort_by(.id)'
```

Item 4 is now in the list.

```json
[
  {
    "id": "id1",
    "name": "name1"
  },
  {
    "id": "id2",
    "name": "name2"
  },
  {
    "id": "id3",
    "name": "name3"
  },
  {
    "id": "id4",
    "name": "name4"
  }
]
```

And finally, test the get-item-by-id API:

```shell
curl -s $ENDPOINT/id1 | jq
```

```json
{
  "id": "id1",
  "name": "name1"
}
```

You just learned how SAM Policy Templates make it easy to grant Lambda functions access to specific AWS resources without needing to hand-write IAM policies.

## Inline and Managed Policies

In the last module you learned that SAM Policy templates are an easy way to grant access to different AWS resources 
from your Lambda functions. As useful and easy as they are you may sometimes need to be more specific 
in how you handle IAM access. In other ciccumstances there may not be a policy template that fits with 
what you need. There could also be an [AWS managed policy](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-vs-inline.html#aws-managed-policies) that you could use to simplify your application.

SAM allows you to hand craft your own IAM Policies just as you would in raw CloudFormation. 
It also allows you to refer to AWS managed policies by name or ARN. In this module you'll 
learn how to add your own low-level policies and AWS managed policies in your SAM applications.

### Authoring an inline policy

In the last module you used the `DynamoDBCrudPolicy` SAM policy template to grant CRUD access to a DynamoDB table.

```yaml
  # This is a Lambda function config associated with the source code: get-all-items.js
  getAllItemsFunction:
      ...
      Policies:
        # Give Create/Read/Update/Delete Permissions to the SampleTable
        - DynamoDBCrudPolicy:
            TableName: !Ref SampleTable
      ...
```

In that module, the `Policies` property references a single SAM Policy template. However, `Policies` can also include an inline IAM policy.

Make the following change to your `template.yaml` file, replacing the `DynamoDBCrudPolicy` statement with the 
inline IAM policy. The highlighed lines below show the change to make.

```yaml

26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
# This is a Lambda function config associated with the source code: get-all-items.js
getAllItemsFunction:
  Type: AWS::Serverless::Function
  Properties:
    Handler: src/handlers/get-all-items.getAllItemsHandler
    Runtime: nodejs16.x
    Architectures:
      - x86_64
    MemorySize: 128
    Timeout: 100
    Description: A simple example includes a HTTP get method to get all items from a DynamoDB table.
    Policies:
      - Statement:
          - Sid: DynamoDBReadPolicy
            Effect: Allow
            Action:
              - dynamodb:Scan
            Resource:
              - Fn::Sub:
                  - "arn:${AWS::Partition}:dynamodb:${AWS::Region}:${AWS::AccountId}:table/${tableName}"
                  - tableName: !Ref SampleTable
    Environment:
      Variables:
        # Make table name accessible as environment variable from function code during execution
        SAMPLE_TABLE: !Ref SampleTable
    Events:
      Api:
        Type: Api
        Properties:
          Path: /
          Method: GET
```

Go ahead and make this change to your template.yaml and redeploy the application.

```shell
cd ~/environment/sam-app
sam build && sam deploy
```

Test the endpoint after the deployment is complete to see how the Lambda function still has `dynamodb:Scan` access to the DynamoDB Table.

```shell
curl -s $ENDPOINT | jq 'sort_by(.id)'
```

```json
[
  {
    "id": "id1",
    "name": "name1"
  },
  {
    "id": "id2",
    "name": "name2"
  },
  {
    "id": "id3",
    "name": "name3"
  },
  {
    "id": "id4",
    "name": "name4"
  }
]
```

### Expanding an inline policy

You can see how this pattern is easy extend when your functions need specific IAM policies. 
The example below shows how you could grant `DescribeLogGroups` and `DescribeLogStreams` permissions 
to your Lambda function. Note the Resource field refers to CloudWatch log groups that start with `/aws/lambda/lambd-`.

```yaml
getAllItemsFunction:
  Type: AWS::Serverless::Function
  Properties:
    Handler: src/handlers/get-all-items.getAllItemsHandler
    ...
    Policies:
      - Statement:
          - Sid: DynamoDBReadPolicy
            Effect: Allow
            Action:
              - dynamodb:Scan
            Resource:
              - Fn::Sub:
                  - "arn:${AWS::Partition}:dynamodb:${AWS::Region}:${AWS::AccountId}:table/${tableName}"
                  - tableName: !Ref SampleTable
          - Sid: ViewLogs
            Effect: Allow
            Action:
            - logs:DescribeLogGroups
            - logs:DescribeLogStreams
            Resource: !Sub "arn:aws:logs:${AWS::Region}:${AWS::AccountId}:log-group:/aws/lambda/lambda-*"
```

### Using AWS managed policies

Managed IAM policies are administered by AWS. These policies are generic and convienent, 
making it simple to grant access to a broad set of resources for common use cases. You can read through 
the [full list of managed policies on the Reference Guide](https://docs.aws.amazon.com/aws-managed-policy/latest/reference/CloudWatchEventsFullAccess.html). Two specific examples 
are `AmazonEC2ReadOnlyAccess` and `CloudWatchEventsFullAccess`.

Imagine a Lambda function that needs to read details for all of your EC2 instances. Adding a least privileged 
inline policy would be tedious if there were dozens (or hundreds!) of EC2 instances. 
In these cases, adding a managed policy is easy and efficient. Always be aware of your security posture 
when adding managed policies! You likely wouldn't want to grant `ec2:TerminateInstances` 
to a Lambda function for all of your instances.

The example below shows how you can add the `AmazonEC2ReadOnlyAccess` managed policy alongside 
your inline policy statements to grant EC2 read access to this Lambda function.

```yaml
  getAllItemsFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: src/handlers/get-all-items.getAllItemsHandler
      ...
      Policies:
        - arn:aws:iam::aws:policy/AmazonEC2ReadOnlyAccess
        - Statement:
          - Sid: DynamoDBReadPolicy
            Effect: Allow
            Action:
              - dynamodb:Scan
            Resource:
              - Fn::Sub:
                  - "arn:${AWS::Partition}:dynamodb:${AWS::Region}:${AWS::AccountId}:table/${tableName}"
                  - tableName: !Ref SampleTable
          - Sid: ViewLogs
            Effect: Allow
            Action:
            - logs:DescribeLogGroups
            - logs:DescribeLogStreams
            Resource: !Sub "arn:aws:logs:${AWS::Region}:${AWS::AccountId}:log-group:/aws/lambda/lambda-*"

```

If you add the `AmazonEC2ReadOnlyAccess` managed policy, navigate to the function's IAM execution role to see it added.

![image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/connectors/add-aws-managed-policy.png)

You just learned how to add your own IAM Policy Statements to give your Lambda functions access to AWS resouces. 
You learned how these are different from SAM Policy Templates, and how to use AWS managed policy alongside other policy statements.

## SAM Connectors

Besides making it easier to create, build, test, and deploy serverless applications, AWS SAM now further simplifies 
permission management between serverless components with AWS SAM Connectors. SAM Connectors were 
developed to address the following customer challenges:

* Not everyone is an IAM expert
  * Currently, in order to successfully create IAM policies, customers must understand the IAM actions and scopes for all the AWS services they use.
* Generating well scoped policies is tedious
  * Creating well scoped policies require multiple iterations of trial and error, leading developers to create overly permissive ones.
* Application maintenance requires constant IAM policy grooming
  * Continuous maintenance and development common to serverless applications requires continuous updates to IAM policies, slowing development.

The goals of SAM Connectors are:

* Empower the developer to focus on modeling resource relationships, not access permissions
* Reduce the AWS expertise required to successfully build safe production-ready serverless applications
* Increase the iteration speed while developing serverless applications
* Work with existing operational governance controls to reduce friction of getting serverless applications to production

AWS SAM Connectors support AWS Step Functions, Amazon DynamoDB, AWS Lambda, Amazon SQS, Amazon SNS, 
Amazon API Gateway, Amazon EventBridge and Amazon S3, with more resources planned in the future. 
If you are interested in the complete list of services that SAM Connectors supports visit the [AWS SAM Connector Reference](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/reference-sam-connector.html) 

AWS SAM policy templates are an existing feature that helps builders deploy serverless applications with minimally 
scoped IAM policies. Because there are a finite number of templates, they’re a good fit when a template exists for 
the services you’re using. Connectors are best for those getting started and who want to focus on modeling the flow 
of data and events within their applications. Connectors will take the desired relationship model and create 
the permissions for the relationship to exist and function as intended.

### What are AWS SAM Connectors?

Connectors are an AWS Serverless Application Model (AWS SAM) abstract resource type, 
identified as AWS::Serverless::Connector, that provides simple and well-scoped permissions between your serverless 
application resources. Use the Connectors resource attribute by embedding it within a source resource. 
Then, define your destination resource and describe how data or events should flow between those resources. 
AWS SAM then composes the access policies necessary to facilitate the required interactions.

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
...
Resources:
  <source-resource-logical-id>:
    Type: <resource-type>
    ...
    Connectors:
      <connector-name>:
        Properties:
          Destination:
            <properties-that-identify-destination-resource>
          Permissions:
            <permission-types-to-provision>
  ...
```

Let's modify our application to use AWS SAM Connectors to maintain secure best practices.

For each of the three Lambda functions, remove the `Policies` attribute (under the `Properties` attribute) and 
add a `Connectors` attribute as shown in the highlighted lines blow. In each of the connectors, 
note that the destination is the DynamoDB table `SampleTable`.

```yaml
Resources:
  # Each Lambda function is defined by properties:
  # https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md#awsserverlessfunction

  # This is a Lambda function config associated with the source code: get-all-items.js
  getAllItemsFunction:
    Type: AWS::Serverless::Function
    Connectors:
      GetItemsConnector:
        Properties:
          Destination:
            Id: SampleTable
          Permissions:
            - Read
    Properties:
      Handler: src/handlers/get-all-items.getAllItemsHandler
      Runtime: nodejs18.x
      Architectures:
        - x86_64
      MemorySize: 128
      Timeout: 100
      Description: A simple example includes a HTTP get method to get all items from a DynamoDB table.
      Environment:
        Variables:
          # Make table name accessible as environment variable from function code during execution
          SAMPLE_TABLE: !Ref SampleTable
      Events:
        Api:
          Type: Api
          Properties:
            Path: /
            Method: GET
  # Each Lambda function is defined by properties:
  # https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md#awsserverlessfunction

  # This is a Lambda function config associated with the source code: get-by-id.js
  getByIdFunction:
    Type: AWS::Serverless::Function
    Connectors:
      GetItemByIdConnector:
        Properties:
          Destination:
            Id: SampleTable
          Permissions:
            - Read
    Properties:
      Handler: src/handlers/get-by-id.getByIdHandler
      Runtime: nodejs18.x
      Architectures:
        - x86_64
      MemorySize: 128
      Timeout: 100
      Description: A simple example includes a HTTP get method to get one item by id from a DynamoDB table.
      Environment:
        Variables:
          # Make table name accessible as environment variable from function code during execution
          SAMPLE_TABLE: !Ref SampleTable
      Events:
        Api:
          Type: Api
          Properties:
            Path: /{id}
            Method: GET
  # Each Lambda function is defined by properties:
  # https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md#awsserverlessfunction

  # This is a Lambda function config associated with the source code: put-item.js
  putItemFunction:
    Type: AWS::Serverless::Function
    Connectors:
      PutItemConnector:
        Properties:
          Destination:
            Id: SampleTable
          Permissions:
            - Write
    Properties:
      Handler: src/handlers/put-item.putItemHandler
      Runtime: nodejs18.x
      Architectures:
        - x86_64
      MemorySize: 128
      Timeout: 100
      Description: A simple example includes a HTTP post method to add one item to a DynamoDB table.
      Environment:
        Variables:
          # Make table name accessible as environment variable from function code during execution
          SAMPLE_TABLE: !Ref SampleTable
      Events:
        Api:
          Type: Api
          Properties:
            Path: /
            Method: POST
```

Let's deploy the application with these changes.

```shell
cd ~/environment/sam-app
sam build && sam deploy --no-confirm-changeset
```

Test the `getAllItems` API and you should see the same familiar output.

```shell
curl -s $ENDPOINT | jq
```

Congradulations! That concludes our look at AWS SAM Connectors and the AWS SAM Permissions module.

