# Microservice on AWS

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/0d7b0c79-b88d-454f-a6c3-31736ac720c2/e77a87d3-e532-4c72-9c35-4f829893ce21/Untitled.png)

**This is How I have Created Lambda IAM Role**

1. Opened the “Policies” page in IAM console. and clicked on “Create Policy”. and in the JSON section wrote the following code.

      • Permissions – Custom policy with permission to DynamoDB and CloudWatch Logs. This       custom policy has the permissions that the function needs to write data to DynamoDB and upload logs.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/0d7b0c79-b88d-454f-a6c3-31736ac720c2/ae9081f7-b477-49bf-9dcf-223317dc8c7f/Untitled.png)

{
"Version": "2012-10-17",
"Statement": [
{
"Sid": "Stmt1428341300017",
"Action": [
"dynamodb:DeleteItem",
"dynamodb:GetItem",
"dynamodb:PutItem",
"dynamodb:Query",
"dynamodb:Scan",
"dynamodb:UpdateItem"
],
"Effect": "Allow",
"Resource": "*"
},
{
"Sid": "",
"Resource": "*",
"Action": [
"logs:CreateLogGroup",
"logs:CreateLogStream",
"logs:PutLogEvents"
],
"Effect": "Allow"
}
]
}

1. Successfully Created Policy name as “Lambda-API-Gateway-Policy”.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/0d7b0c79-b88d-454f-a6c3-31736ac720c2/d99bb0f5-892c-486d-8cce-a2ed53f59746/Untitled.png)

1. Opened the “Roles” page in IAM Console.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/0d7b0c79-b88d-454f-a6c3-31736ac720c2/97ea42ac-d404-4743-87ed-7567ea8b12fc/Untitled.png)

4. Clicked on “Create Role” and in the Use Case, Choose “Lambda”.

    • Trusted entity – Lambda.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/0d7b0c79-b88d-454f-a6c3-31736ac720c2/1840e4af-4bcc-4e57-ac0b-6ea84efb7c6f/Untitled.png)

1. Searched for the Police name which I have created name as “Lambda-API-Gateway-Policy”.

      • Role name – **lambda-apigateway-role**.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/0d7b0c79-b88d-454f-a6c3-31736ac720c2/8e30581e-82cd-48f1-a2e6-78c447d0c2a3/Untitled.png)

1. Successfully Created the “Lambda-API-Gateway-Role”.

**This is How I have Created Lambda Function**

1. In the AWS Console, Searched for “Lambda” and clicked on “Create Lambda Function”

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/0d7b0c79-b88d-454f-a6c3-31736ac720c2/dd874eeb-3430-4d38-998a-eff9664d58a0/Untitled.png)

1. Function Name : LambdaFunctionOverHttps

      Runtime : Python 3.7

      Change default Execution Role (Use Existing role) : Lambda-API-Gateway-Role

      Clicked on “Create Function” and this is how Successfully Created Lambda Function

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/0d7b0c79-b88d-454f-a6c3-31736ac720c2/73b9cce9-12d4-46d1-9f86-8ad8e6166774/Untitled.png)

1. In the Lambda Function wrote the below Code and Clicked on “Deploy”

from **future** import print_function

import boto3
import json

print('Loading function')

def lambda_handler(event, context):
'''Provide an event that contains the following keys:

operation: one of the operations in the operations dict below

- tableName: required for operations that interact with DynamoDB
- payload: a parameter to pass to the operation being performed
'''
#print("Received event: " + json.dumps(event, indent=2))

operation = event['operation']

if 'tableName' in event:
dynamo = boto3.resource('dynamodb').Table(event['tableName'])

operations = {
'create': lambda x: dynamo.put_item(**x),
'read': lambda x: dynamo.get_item(**x),
'update': lambda x: dynamo.update_item(**x),
'delete': lambda x: dynamo.delete_item(**x),
'list': lambda x: dynamo.scan(**x),
'echo': lambda x: x,
'ping': lambda x: 'pong'
}

if operation in operations:
return operations[operation](notion://www.notion.so/event.get('payload'))
else:
raise ValueError('Unrecognized operation "{}"'.format(operation))

1. Configured Test Event and wrote following code.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/0d7b0c79-b88d-454f-a6c3-31736ac720c2/95d6d54a-d2fc-4ba9-9ec8-3d81bae2d4bf/Untitled.png)

{
"operation": "echo",
"payload": {
"somekey1": "somevalue1",
"somekey2": "somevalue2"
}
}

**Now Next Step to Create DynamoDB table and an API using our lambda as backend!**

Created the DynamoDB table that the Lambda function uses.

**To create a DynamoDB table**

1. Opened the DynamoDB console.
2. Chosen “Create table”.
3. Created a table with the following settings.
    - Table name –Apigateway-Lambda
    - Primary key – id (string)
    
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/0d7b0c79-b88d-454f-a6c3-31736ac720c2/ef92189d-ec1b-4f53-9b1a-00e5fd90afc7/Untitled.png)
    
    4. Chosen “Create”.
    
    ### Creating an API
    
    **To create the API**
    
    1. Searched for  “API Gateway console”.
    2. Clicked “Create API”.
    3.  Scroll down and select "Build" for REST API.
    4. I have given the API name as "DynamoDBOperations", keep everything as is, clicked "Create API".
    
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/0d7b0c79-b88d-454f-a6c3-31736ac720c2/dff91e53-34a0-4892-87fc-a21ba634a990/Untitled.png)
    
    1. Each API is collection of resources and methods that are integrated with backend HTTP endpoints, Lambda functions, or other AWS services. Typically, API resources are organised in a resource tree according to the application logic. At this time I only have the root resource, but then I have added a resource next.
    2. Clicked "Actions", then clicked "Create Resource".
    
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/0d7b0c79-b88d-454f-a6c3-31736ac720c2/239c31ab-0a99-47f9-a73b-264d27cd118c/Untitled.png)
    
    1. I have given Input as "ManagerDynamoDB" in the Resource Name, Resource Path will get populated. and I have Clicked "Create Resource”.
    2. Now to  create a POST Method for this API. With the "/managerdynamodb" resource selected, Clicked "Actions" again and clicked "Create Method".
    
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/0d7b0c79-b88d-454f-a6c3-31736ac720c2/e7171e6b-c256-40a9-9730-45eac815cd80/Untitled.png)
    
    1. Then Selected "POST" from drop down , then clicked checkmark.
    2. The integration came up automatically with "Lambda Function" option selected. 
    
      Selected "LambdaFunctionOverHttps" function that I created earlier. As I had Started typing the name,  function name  showed up. Selected the same and clicked "Save". A popup window came up to add resource policy to the lambda to be invoked by this API. Clicked "Ok"
    
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/0d7b0c79-b88d-454f-a6c3-31736ac720c2/09b5ac8a-0a3f-4480-a373-f815968c42d9/Untitled.png)
    
    1. Here, I am done with API-Lambda integration.
    
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/0d7b0c79-b88d-454f-a6c3-31736ac720c2/2b96f0a2-63e1-4517-9fba-8bb24930f211/Untitled.png)
    
    ### Deploying the API
    
    In this step, I have deployed the API that I have created to a stage called “production”.
    
    1. Clicked "Actions", selected "Deploy API"
    2. 1. Now this asked about a stage. Selected "[New Stage]" for "Deployment stage". Given "Production" as "Stage name". Clicked "Deploy"
    
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/0d7b0c79-b88d-454f-a6c3-31736ac720c2/54270177-144e-41bf-8be9-6f2f7547e9e6/Untitled.png)
    
    1.  Now this is the correct way to run the solution! To invoke this API endpoint, I need the endpoint url. In the "Stages" screen, expanded the stage "Production", selected "POST" method, and copied the "Invoke URL" from screen.
    
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/0d7b0c79-b88d-454f-a6c3-31736ac720c2/ab93250d-f76d-4350-afe2-a3bd015259f4/Untitled.png)
    
    1. Now in this step I have used an online tool called “Postman”.
    2. I have selected “Post” tab and Pasted the link I have copied.
    
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/0d7b0c79-b88d-454f-a6c3-31736ac720c2/da3485cd-5a75-4e93-af38-1bab25248121/Untitled.png)
    
    ### Running Solution
    
    1. The Lambda function supports using the create operation to create an item in my DynamoDB table. To request this operation, I used the following JSON:
    
    {
    "operation": "create",
    "tableName": "Apigateway-Lambda",
    "payload": {
    "Item": {
    "id": "1234ABCD",
    "number": 5
    }
    }
    }
    
    1. To execute My API from local machine, I have used Postman and Curl command.
    - To run this from Postman, I have selected "POST" , pasteed the API invoke url. Then under "Body" selected "raw" and pasted the above JSON. Clicked "Send". API executed and return "HTTPStatusCode" 200.
    
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/0d7b0c79-b88d-454f-a6c3-31736ac720c2/4ef15f34-4d61-4c10-b735-75d8c5af4213/Untitled.png)
    
    1. When I went back to “DynamoDB” and scan for Item Count, following is the result I got, this means I have got the Result after cross checking with the help of DynamoDB.
    
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/0d7b0c79-b88d-454f-a6c3-31736ac720c2/467a893e-1165-4b06-bdb2-a08b5190b702/Untitled.png)
    
    1. Now, I can change the number of count in the JSON lambda Function and get the result N number of times. To get all the inserted items from the table, I have use the "list" operation of Lambda using the same API. After I have Passed the following JSON to the API, and it  returned all the items from the Dynamo table
    
    {
    "operation": "list",
    "tableName": "Apigateway-Lambda",
    "payload": {
    }
    }
    
    This is how, I have created “**Microservice**” IAM Roles, Lambda, DynamoDB Table, API Gateway and the Operations such as “Echo”, “Read”, “Create” and many more.
