# Serverless Todo List API using AWS Lambda & DynamoDB

A simple serverless Todo List application built with **AWS Lambda**, **Amazon DynamoDB**, **API Gateway**, and a lightweight **HTML + JavaScript frontend**.

## Features

* Create a new todo
* Retrieve all todos
* Update existing todos
* Delete todos
* Fully serverless architecture
* RESTful API using API Gateway
* DynamoDB for persistent storage

---

## Architecture

```text
Frontend (HTML + JavaScript)
           │
           ▼
     API Gateway
           │
 ┌─────────┼─────────┐
 ▼         ▼         ▼
Create    Get      Update/Delete
Lambda   Lambda      Lambda
           │
           ▼
       DynamoDB
        (Todos)
```

---

## Tech Stack

| Layer     | Technology               |
| --------- | ------------------------ |
| Frontend  | HTML, JavaScript         |
| API Layer | AWS API Gateway          |
| Compute   | AWS Lambda               |
| Database  | Amazon DynamoDB          |
| IAM       | AWS IAM Roles & Policies |

---

## Project Structure

```text
todo-api/
├── lambdas/
│   ├── create_todo.py
│   ├── get_todos.py
│   ├── update_todo.py
│   └── delete_todo.py
├── index.html
└── README.md
```

---

## 1️⃣ Create DynamoDB Table

Create a DynamoDB table named:

```text
Todos
```

### Partition Key

| Attribute | Type   |
| --------- | ------ |
| id        | String |

---

## 2️⃣ Create IAM Role

Create an IAM role for Lambda execution and attach the following policies:

### Required Policies

* AmazonDynamoDBFullAccess
* AWSLambdaBasicExecutionRole

Assign this role to all Lambda functions.

---

## 3️⃣ Create Lambda Functions

Create the following Lambda functions:

| Function    | Purpose                 |
| ----------- | ----------------------- |
| create-todo | Create a new todo       |
| get-todo    | Retrieve all todos      |
| update-todo | Update an existing todo |
| delete-todo | Delete a todo           |

{lambda function SAMPLE CODES}

    1.create-todo 
               import json
               import boto3
               import uuid
               from datetime import datetime
               
               dynamodb = boto3.resource('dynamodb')
               table = dynamodb.Table('Todos')
               
               HEADERS = {
                   'Content-Type': 'application/json',
                   'Access-Control-Allow-Origin': '*',
                   'Access-Control-Allow-Headers': 'Content-Type',
                   'Access-Control-Allow-Methods': 'GET,POST,PUT,DELETE,OPTIONS'
               }
               
               def lambda_handler(event, context):
                   try:
                       body = json.loads(event['body'])
                       todo = {
                           'id': str(uuid.uuid4()),
                           'title': body['title'],
                           'completed': False,
                           'createdAt': datetime.utcnow().isoformat()
                       }
                       table.put_item(Item=todo)
                       return { 'statusCode': 201, 'headers': HEADERS, 'body': json.dumps(todo) }
                   except Exception as e:
                       return { 'statusCode': 500, 'headers': HEADERS, 'body': json.dumps({'error': str(e)}) }

          2. get-todo
                    import json
                    import boto3
                    
                    dynamodb = boto3.resource('dynamodb')
                    table = dynamodb.Table('Todos')
                    
                    HEADERS = {
                        'Content-Type': 'application/json',
                        'Access-Control-Allow-Origin': '*',
                        'Access-Control-Allow-Headers': 'Content-Type',
                        'Access-Control-Allow-Methods': 'GET,POST,PUT,DELETE,OPTIONS'
                    }
                    
                    def lambda_handler(event, context):
                        try:
                            result = table.scan()
                            return { 'statusCode': 200, 'headers': HEADERS, 'body': json.dumps(result['Items']) }
                        except Exception as e:
                            return { 'statusCode': 500, 'headers': HEADERS, 'body': json.dumps({'error': str(e)}) }
          
          3.update-todo
                         import json
                         import boto3
                         
                         dynamodb = boto3.resource('dynamodb')
                         table = dynamodb.Table('Todos')
                         
                         HEADERS = {
                             'Content-Type': 'application/json',
                             'Access-Control-Allow-Origin': '*',
                             'Access-Control-Allow-Headers': 'Content-Type',
                             'Access-Control-Allow-Methods': 'GET,POST,PUT,DELETE,OPTIONS'
                         }
                         
                         def lambda_handler(event, context):
                             try:
                                 todo_id = event['pathParameters']['id']
                                 body = json.loads(event['body'])
                                 table.update_item(
                                     Key={'id': todo_id},
                                     UpdateExpression='SET title = :t, completed = :c',
                                     ExpressionAttributeValues={
                                         ':t': body.get('title'),
                                         ':c': body.get('completed', False)
                                     }
                                 )
                                 return { 'statusCode': 200, 'headers': HEADERS, 'body': json.dumps({'message': 'Updated successfully'}) }
                             except Exception as e:
                                 return { 'statusCode': 500, 'headers': HEADERS, 'body': json.dumps({'error': str(e)}) }
          4.delete-todo
                              import json
                              import boto3
                              
                              dynamodb = boto3.resource('dynamodb')
                              table = dynamodb.Table('Todos')
                              
                              HEADERS = {
                                  'Content-Type': 'application/json',
                                  'Access-Control-Allow-Origin': '*',
                                  'Access-Control-Allow-Headers': 'Content-Type',
                                  'Access-Control-Allow-Methods': 'GET,POST,PUT,DELETE,OPTIONS'
                              }
                              
                              def lambda_handler(event, context):
                                  try:
                                      todo_id = event['pathParameters']['id']
                                      table.delete_item(Key={'id': todo_id})
                                      return { 'statusCode': 200, 'headers': HEADERS, 'body': json.dumps({'message': 'Deleted successfully'}) }
                                  except Exception as e:
                                      return { 'statusCode': 500, 'headers': HEADERS, 'body': json.dumps({'error': str(e)}) }


### Runtime

```text
Python 3.x
```

Upload the corresponding Python code to each Lambda function.

---

## 4️⃣ Configure API Gateway

Create a REST API and add the following resources.

### Resource

```text
/todo
```

### Methods

| Method | Lambda Function |
| ------ | --------------- |
| POST   | create-todo     |
| GET    | get-todo        |
| PUT    | update-todo     |
| DELETE | delete-todo     |

### Additional Resource for Update/Delete

```text
/todo/{id}
```

Attach:

```text
PUT    → update-todo
DELETE → delete-todo
```

Enable **CORS** for all methods.

---

## 5️⃣ Deploy API

Deploy the API Gateway stage.

After deployment, copy the generated Invoke URL:

```text
https://your-api-id.execute-api.region.amazonaws.com/prod
```

Update your frontend JavaScript:

```javascript
const API_URL = "YOUR_INVOKE_URL";
```

---

## API Endpoints

### Create Todo

```http
POST /todo
```

Request:

```json
{
  "title": "Learn AWS Lambda"
}
```

Response:

```json
{
  "id": "uuid",
  "title": "Learn AWS Lambda",
  "completed": false,
  "createdAt": "2026-01-01T12:00:00"
}
```

---

### Get Todos

```http
GET /todo
```

Response:

```json
[
  {
    "id": "uuid",
    "title": "Learn AWS Lambda",
    "completed": false
  }
]
```

---

### Update Todo

```http
PUT /todo/{id}
```

Request:

```json
{
  "title": "Learn AWS Serverless",
  "completed": true
}
```

Response:

```json
{
  "message": "Updated successfully"
}
```

---

### Delete Todo

```http
DELETE /todo/{id}
```

Response:

```json
{
  "message": "Deleted successfully"
}
```

---

## CORS Configuration

All Lambda functions include the following CORS headers:

```python
HEADERS = {
    "Content-Type": "application/json",
    "Access-Control-Allow-Origin": "*",
    "Access-Control-Allow-Headers": "Content-Type",
    "Access-Control-Allow-Methods": "GET,POST,PUT,DELETE,OPTIONS"
}
```

---

## 🚀 Deployment Flow

1. Create DynamoDB table.
2. Create IAM role.
3. Create Lambda functions.
4. Upload function code.
5. Configure API Gateway.
6. Enable CORS.
7. Deploy API.
8. Copy Invoke URL.
9. Update frontend API URL.
10. Start using your Todo application.

---

## 📸 Demo Workflow

```text
Create Todo
    ↓
Store in DynamoDB
    ↓
Retrieve Todos
    ↓
Update Todo
    ↓
Delete Todo
```

---

## Author

**Piyush Patel**

GitHub: @Piyushpatel27
