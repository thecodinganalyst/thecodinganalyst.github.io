---
title: "Creating a simple serverless application with AWS SAM on local"
excerpt_separator: "<!--more-->"
categories:
  - Tutorial
tags:
  - AWS
  - AWS SAM
  - AWS Lambda
  - Cloud Computing
---

Amazon Web Services Serverless Application Model (AWS SAM) is a platform to help developers quick-start a lambda application with all the essentials and basic code needed.

It’s not perfect if you need to make it working on your local as well as on AWS, so I am documenting some help here.

**Prerequisites**:  [](https://aws.amazon.com/)[AWS Account](https://aws.amazon.com/),  [AWS Cli](https://aws.amazon.com/cli/),  [AWS SDK for NodeJS](https://aws.amazon.com/developers/getting-started/nodejs/),  [SAM Cli](https://docs.aws.amazon.com/serverless-application-model/index.html),  [Docker](https://www.docker.com/)

<!--more-->

Installing the SAM Cli – https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html

```
sam init
```

```
Which template source would you like to use?
	1 - AWS Quick Start Templates
	2 - Custom Template Location
Choice: 1
```

```
Which runtime would you like to use?
	1 - nodejs12.x
	2 - python3.8
	3 - ruby2.7
	4 - go1.x
	5 - java11
	6 - dotnetcore3.1
	7 - nodejs10.x
	8 - python3.7
	9 - python3.6
	10 - python2.7
	11 - ruby2.5
	12 - java8.al2
	13 - java8
	14 - dotnetcore2.1
Runtime: 1
```

```
Project name [sam-app]:
```

```
Cloning app templates from https://github.com/awslabs/aws-sam-cli-app-templates.git

AWS quick start application templates:
	1 - Hello World Example
	2 - Step Functions Sample App (Stock Trader)
	3 - Quick Start: From Scratch
	4 - Quick Start: Scheduled Events
	5 - Quick Start: S3
	6 - Quick Start: SNS
	7 - Quick Start: SQS
	8 - Quick Start: Web Backend
Template selection: 8
```

I’m selecting option 8, Web Backend template here, and it will create the following 3 lambda functions to access and update the data in a dynamodb table.

1.  Get All Items
2.  Get Item By Id
3.  Put Item

```
-----------------------
Generating application:
-----------------------
Name: sam-starter
Runtime: nodejs12.x
Dependency Manager: npm
Application Template: quick-start-web
Output Directory: .

Next steps can be found in the README file at ./sam-starter/README.md
```

Open the project name folder in your favourite IDE, and you can see all the necessary files and folders are created for you.

-   Project Name Folder
    -   __tests__
        -   unit
            -   handlers
                -   get-all-items.test.js
                -   get-by-id.test.js
                -   put-item-test.js
    -   events
        -   event-get-all-items.json
        -   event-get-by-id.json
        -   event-post-item.json
    -   src
        -   handlers
            -   get-all-items.js
            -   get-by-id.js
            -   put-item.js
    -   gitignore
    -   buildspec.yml
    -   env.json
    -   package.json
    -   README.md
    -   template.yml

The table below shows what each of the folders and files are to be used for.

template.yml

SAM Instruction to generate the necessary resources  
(lambda functions, dynamodb, etc) with AWS CloudFormation

README.md

Markdown file describing the project

env.json

For specifying the environment variables when you run the project in local

package.json

Compiler information for nodejs to build your code

buildspec.yml

Instructions for AWS CodeBuild to build the application

gitignore

Git instructions to instruct the git source code repository to ignore uploading the files and folders stated in this file

src

The source code folder

events

The event files for invoking the lambda functions locally

__tests__

The unit tests for testing your code before deploying

To build the application, run the following command in the terminal.

```
sam build
```

```
Build Succeeded

Built Artifacts  : .aws-sam/build
Built Template   : .aws-sam/build/template.yaml

Commands you can use next
=========================
[*] Invoke Function: sam local invoke
[*] Deploy: sam deploy --guided
```

Upon completion, the .aws-sam folder will be created for each of the 3 functions. You can invoke any of the 3 functions with sam local invoke command. The function names are the resources defined in the template.yml.

```
sam local invoke getAllItemsFunction --event event/event-get-all-items.json
```

If you are building it for the first time, SAM will pull the docker image and try to build it before it invokes the function. However, you will hit with the following error.

```
{"errorType":"ResourceNotFoundException","errorMessage":"Requested resource not found"}
```

From the message, we can tell that it is trying to connect to the dynamodb, and it is definitely failing, because we are running this on local, and we have not deployed anything to AWS yet. So now, we need to set up a dynamodb on our local machine, and my preferred way of doing this is to run it from a docker container. To do this, we’ll create a docker folder under the project name folder, and create a docker-compose.yml file with the following content.

```
version: '3.7'
networks:
  sam-local:
    driver: bridge
services:
  dynamodb-local:
    image: amazon/dynamodb-local:latest
    container_name: dynamodb-local
    networks:
      - sam-local
    ports:
      - "8000:8000"
```

Having this file helps us start up the dynamodb container easily by running  docker-compose up, instead of remembering the commands. The instructions in the docker-compose.yml tells docker to create a network named sam-local, and create a container from the latest dynamodb image from amazon, name it dynamodb-local, and lets it join the sam-local network, listening on port 8000. The first 8000 in in the ports is the port on the host computer, while the second 8000 is the port to use from within the container.

Navigate to the docker folder in your terminal, and run the following command.

```
docker-compose up
```

This will start the dynamodb container on your local machine, and you can see it from the docker desktop.

![](https://i0.wp.com/thecodinganalyst.com/wp-content/uploads/2020/10/Screenshot-2020-10-25-at-9.47.33-PM.png?resize=660%2C418&ssl=1)

Docker Desktop

After we get the database up and running, we need to create the table, so that our lambda function can access. Create a file named create-table with the following content.

```
aws dynamodb create-table \
  --table-name Note \
  --attribute-definitions \
    AttributeName=id,AttributeType=S \
  --key-schema \
    AttributeName=id,KeyType=HASH \
  --provisioned-throughput   ReadCapacityUnits=5,WriteCapacityUnits=5 \
  --endpoint-url http://localhost:8000
```

The above code will create a table named Note, with a single primary key named id, of String type. The endpoint-url parameter specifies the connection to dynamodb is at localhost port 8000.

If you are on a mac or linux, grant the executable permission to the file

```
chmod +x create-table
```

Then execute it.

```
./create-table
```

If you look at the javascript source files, the table name is specified to be retrieved from the environment variable named SAMPLE_TABLE.

```
// Get the DynamoDB table name from environment variables
const tableName = process.env.SAMPLE_TABLE;
```

We can specify the value of the environment variable in the AWS portal, but when we are running on local, the file for our environment variable is already created and specified in the env.json

```
{
    "getAllItemsFunction": {
        "SAMPLE_TABLE": "<TABLE-NAME>"
    },
    "getByIdFunction": {
        "SAMPLE_TABLE": "<TABLE-NAME>"
    },
    "putItemFunction": {
        "SAMPLE_TABLE": "<TABLE-NAME>"
    }
}
```

The <TABLE-NAME> is actually just a placeholder. So we’ll need to change it to our table name which we have just created.

```
{
    "getAllItemsFunction": {
        "SAMPLE_TABLE": "Note"
    },
    "getByIdFunction": {
        "SAMPLE_TABLE": "Note"
    },
    "putItemFunction": {
        "SAMPLE_TABLE": "Note"
    }
}
```

That’s not all. As we are running our dynamodb on local, we’ll need to specify the endpoint in our dynamodb.DocumentClient() constructor. Open up the get-all-items.js in src/handlers and look for the line below.

```
const docClient = new dynamodb.DocumentClient();
```

Add the endpoint to the constructor parameter. For the full list of the options, visit  [https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/DynamoDB.html#constructor-property](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/DynamoDB.html#constructor-property)

```
const docClient = new dynamodb.DocumentClient({endpoint: http://dynamodb-local:8000});
```

The “dynamodb-local” is the container name which we have specified in our docker-compose.yml. We cannot use localhost as we will be running this set of code in SAM, which is in another docker container. So it’s a container to container interaction, the dynamodb container can be reference to as localhost from your local machine, and that’s why you can use localhost as the endpoint when you run the aws dynamodb command. But the dynamodb container is not localhost to the SAM container.

Now, we can compile our code again.

```
sam build
```

To invoke the getAllItemsFunction, we need a few more parameters.

```
sam local invoke getAllItemsFunction --event events/event-get-all-items.json --env-vars env.json --docker-network docker_sam-local
```

The event parameter is for providing the event json to trigger the lambda function. Lambda functions are standalone functions, that only runs when an event is triggered, so we need an event to trigger it and provide the context for the lambda function to run.

The env-vars parameter specifies where the environment parameters are to be retrieved from.

The docker-network parameter specifies the docker network to use, so that both the SAM container and our local dynamodb container are in the same network. Docker will add the folder name, followed by an underscore, for the prefix of the docker network. In our case, it is “docker_sam-local”. sam-local is the network we specified for the dynamodb-local container to connect, in our docker-compose.yml.

For the full list of parameters for the sam local invoke command, visit  [https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-local-invoke.html](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-local-invoke.html).

Running the above command will briefly start up the container to run the get-all-items.js, return the result, and the container will exit. If you have a docker desktop opened up, you will see a container created for a brief moment and destroyed. Anyway, you should see the following result in the last line after running the command.

```
{"statusCode":200,"body":"[]"}
```

This is specified in the last line “return response” from our get-all-items.js. The function returns a status code of 200 and the list of items from the Note table in our dynamodb. Since we didn’t add any records to the table, the function will return an empty array.

That’s great! But having the local endpoint url hardcoded in the script will cause us issues when we deploy to AWS, as we shouldn’t be referencing a local endpoint.

```
const docClient = new dynamodb.DocumentClient({endpoint: http://dynamodb-local:8000});
```

Instead we should have a parameter to indicate whether the code is running on local or production. If it is running on local, then we will set the endpoint, which the url should also be provided from the parameter. To do so, we need to add the 2 parameters in our template.yml, after the Transform section.

```
Parameters:
  Environment:
    Type: String
    Default: Prod
    AllowedValues: 
      - Prod
      - Local
    Description: Prod for running on AWS, Local for running on local connecting to local DynamoDB
  DynamodbEndpoint:
    Type: String
    Description: For specifying the dynamodb endpoint url when running on local
```

We can’t use the parameter directly from our script, so we need to create environment variables in the respective function to utilise the parameter values. Look for the Environment key under the getAllItemsFunction, and edit it to become as follows:

```
Environment:
        Variables:
          SAMPLE_TABLE: !Ref SampleTable
          ENVIRONMENT: !Ref Environment
          DYNAMODB_ENDPOINT: !Ref DynamodbEndpoint
```

Go to our code (get-all-items.js) and update the first few lines to become as such

```
// Get the DynamoDB table name from environment variables
const tableName = process.env.SAMPLE_TABLE;

// If code is running on local, create the options to set the endpoint for the dynamodb document client
let environment = process.env.ENVIRONMENT;
let docClientOptions = {};
if(environment == "Local"){
    let endpoint_url = process.env.DYNAMODB_ENDPOINT;
    docClientOptions = { endpoint: endpoint_url }
}

// Create a DocumentClient that represents the query to add an item
const dynamodb = require('aws-sdk/clients/dynamodb');
const docClient = new dynamodb.DocumentClient(docClientOptions);
```

In order to provide the parameters to our invoke function, we need to add the –parameter-overrides to our command. However the command is getting too complex to remember, so i’d create a folder named local, and create a file named invoke-get-all-items.

```
mkdir local
cd local
```

```
sam local invoke getAllItemsFunction --event ../events/event-get-all-items.json --env-vars ../env.json --parameter-overrides 'ParameterKey=Environment,ParameterValue=Local ParameterKey=DynamodbEndpoint,ParameterValue=http://dynamodb-local:8000' --docker-network docker_sam-local -t ../template.yml
```

```
chmod +x invoke-get-all-items
./invoke-get-all-items
```

You should get the same result as above.

```
{"statusCode":200,"body":"[]"}
```

Try to do the same for the other 2 functions. The full code is available at  [https://github.com/thecodinganalyst/sam-starter](https://github.com/thecodinganalyst/sam-starter)

In the next article, I will be creating the frontend application to utilise this code.