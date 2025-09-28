Serverless computing allows you to run a code without managing the server. You just deploy the
function and cloud provider runs them on demand.

Note: Python is widely supported in serverless environment.

### Key Concept
1. Function as Service (FaaS)
- Write a simple, single purpose functions
- The cloud providers handles scaling, Provisioning, HA, Monitoring

2. Event Driven (Functions are triggered via events)
3. Stateless (Each execution is independent)
4. Billing (Pay only what compute time used)


#### 1. AWS Lambda – Python
```python
# lambda_function.py

def lambda_handler(event, context):
    name = event.get("name", "World")
    return {
        "statusCode": 200,
        "body": f"Hello, {name}!"
    }
```

Deploying Lamda (zip your python file)
```
zip function.zip lambda_function.py
```

Create Lambda function:
```shell
aws lambda create-function \
  --function-name hello-python \
  --runtime python3.10 \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://function.zip \
  --role <IAM_ROLE_ARN>
```

Test with AWS CLI:
```shell
aws lambda invoke \
  --function-name hello-python \
  --payload '{"name": "Aditya"}' response.json
cat response.json
```
Steps:
```
API Gateway → HTTP requests
S3 → file uploads
SQS → queue messages
CloudWatch → cron jobs
```

But to do this, AWS CLI authentication is required:
```shell
export AWS_ACCESS_KEY_ID=YOUR_KEY
export AWS_SECRET_ACCESS_KEY=YOUR_SECRET
export AWS_DEFAULT_REGION=us-east-1
```


#### 2. GCP Cloud Functions – Python
simple hello_world.py function
```python
# main.py
def hello_world(request):
    request_json = request.get_json(silent=True)
    name = request_json.get('name') if request_json else 'World'
    return f'Hello, {name}!'
```

Deploying function:
```shell
gcloud functions deploy hello_world \
  --runtime python310 \
  --trigger-http \
  --allow-unauthenticated
```

Test:
```
curl -X POST https://REGION-PROJECT_ID.cloudfunctions.net/hello_world \
  -H "Content-Type: application/json" \
  -d '{"name":"Aditya"}'
```

Disadvantage:
1. Cold starts (startup latency for infrequent functions)
2. Execution time limits (e.g., AWS Lambda = 15 min)
3. Stateless (need external storage)
4. Debugging / local testing harder than traditional apps

ASCII Diagram:
```
            ┌──────────────┐
            │   Trigger    │
            │ (HTTP, S3,  │
            │  Queue, etc)│
            └─────┬────────┘
                  │
                  ▼
          ┌───────────────┐
          │   Cloud FaaS  │
          │  (AWS Lambda, │
          │  GCP Function)│
          └─────┬─────────┘
                │
        ┌───────┴──────────┐
        │       Python      │
        │    Function Code  │
        └───────┬──────────┘
                │
       ┌────────┴─────────┐
       │ Output / Storage │
       │ (S3, GCS, DB)   │
       └────────┬─────────┘
                │
         ┌──────┴───────┐
         │ Monitoring / │
         │  Logs        │
         │(CloudWatch / │
         │ Stackdriver) │
         └──────────────┘
```
