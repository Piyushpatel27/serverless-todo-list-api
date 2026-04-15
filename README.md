frontend: html+js
backend (serverless using AWS): 
     DynamoDB
     todo-api/  
          ├── lambdas/  
          │    ├── create_todo.py   
          │    ├── get_todos.py     
          │    ├── update_todo.py   
          │    └── delete_todo.py  
    API gateway: POST, GET, PUT , DELETE methods




lambda function CODES
   
    
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


          
