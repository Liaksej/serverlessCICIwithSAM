# 3.4 Prueba local de funciones Lambda

Now that you have a SAM application. You will learn how to run it and test it locally using the AWS SAM CLI. 
This is important because its part of the day to day development workflow. It helps you verify if the application is 
behaving as expected, debug what's wrong, and fix any issues before pushing your changes to a central repository.

![image_3.4.1.png](image_3.4.1.png)

### Install dependencies

Before we run the application locally, it's a common practice to install third-party libraries or dependencies that 
your application might be using. These dependencies are defined in a file that varies depending on the runtime, for 
example package.json for Node.js projects, requirements.txt for Python, and pom.xml for Java.

```shell
cd ~/environment/sam-app/hello-world
npm install
```

Example output:

```
npm notice created a lockfile as package-lock.json. You should commit this file.
npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@~2.3.1 (node_modules/chokidar/node_modules/fsevents):
npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@2.3.2: wanted {"os":"darwin","arch":"any"} (current: {"os":"linux","arch":"x64"})

added 100 packages from 72 contributors and audited 101 packages in 3.879s

18 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
```

### Run using SAM CLI

There are two ways of running a Serverless app locally:

By invoking an individual Lambda function ([SAM Local Invoke reference](https://docs.aws.amazon.com/en_pv/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-local-invoke.html))

```shell
cd ~/environment/sam-app
sam local invoke --event events/event.json
```

By running a local HTTP server that simulates API Gateway

```shell
cd ~/environment/sam-app
sam local start-api --port 8080
```

In this module, we will run a local HTTP server that simulates API Gateway (2).

> **Note**  
>If you receive a permissions error pulling the docker image, you may have to authenticate to the ECR Public repository 
> with the command shown in the following code block.

```shell
aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws
```

> **Note**  
>In a Cloud9 workspace, you must use port 8080, 8081 or 8082 to be able to open the URL in the local browser for preview.

### Test your endpoint

Once your local server is running, we can send HTTP requests to test it. Choose one of the following options:

> **Note**
> The first request will take several seconds as SAM downloads a Docker image. Continue reading to learn why SAM does this.

#### Option A) Using CURL

Without stopping the running process, open a new terminal.

![image3.4.2.png](image3.4.2.png)

Test your endpoint by running a CURL command that triggers an HTTP GET request.

```shell
curl http://localhost:8080/hello
```

#### Option B) Using a browser window

In Cloud9, go to the top menu and chose **Tools > Preview > Preview Running Application**. A browser tab will open, append 
`/hello` to the end of the URL. This will invoke your Lambda function locally.

![image_3.4.3.png](image_3.4.3.png)

> **Note**
> Notice how SAM is pulling a Docker container image. This is how SAM is able to simulate the Lambda runtime locally 
> and run your function. The first invocation might take a few seconds due to the `docker pull` command, but subsequent 
> invocations will be faster. SAM will pull the appropriate container based on the runtime you have configured in `template.yaml`.
