# AWS Lambda

## What It Is

Lambda is a serverless compute service — you upload code, and AWS runs it in response to events without you provisioning or managing servers. You pay only for the compute time actually used, billed per millisecond.

## Core Concepts

| Concept | Description |
|---|---|
| **Function** | The unit of deployment — your code plus configuration (memory, timeout, runtime) |
| **Trigger/Event Source** | What invokes the function (API Gateway, S3 event, DynamoDB stream, EventBridge schedule, etc.) |
| **Handler** | The specific method in your code that Lambda calls to start execution |
| **Cold Start** | The latency incurred when Lambda has to initialize a new execution environment |
| **Execution Role** | The IAM role granting the function permissions to access other AWS services |

## Common Trigger Patterns

| Trigger | Use Case |
|---|---|
| **API Gateway** | Build a REST/HTTP API backed entirely by Lambda (serverless API) |
| **S3 Event** | Run code when a file is uploaded/deleted (e.g., resize an image on upload) |
| **EventBridge (CloudWatch Events)** | Scheduled tasks (cron-like) or reacting to AWS service events |
| **DynamoDB Streams** | React to changes in a DynamoDB table in near real-time |
| **SQS** | Process messages from a queue asynchronously |

## Example: Minimal Python Handler

```python
def lambda_handler(event, context):
    name = event.get("queryStringParameters", {}).get("name", "world")
    return {
        "statusCode": 200,
        "body": f"Hello, {name}!"
    }
```

## Basic Commands (AWS CLI)

```bash
# Create a function
aws lambda create-function \
  --function-name hello-world \
  --runtime python3.12 \
  --role arn:aws:iam::123456789012:role/lambda-execution-role \
  --handler app.lambda_handler \
  --zip-file fileb://function.zip

# Invoke a function
aws lambda invoke \
  --function-name hello-world \
  --payload '{"queryStringParameters": {"name": "Muzammil"}}' \
  response.json

# Update function code
aws lambda update-function-code \
  --function-name hello-world \
  --zip-file fileb://function.zip
```

## Lambda Pricing

Billed on two dimensions:
1. **Number of requests** — first 1M requests/month free, then per-request cost
2. **Duration** — GB-seconds (memory allocated × execution time)

Because of this, right-sizing memory (which also scales CPU proportionally) directly affects both cost and speed.

## Cold Starts

A cold start happens when Lambda has to spin up a new execution environment (new container) instead of reusing a "warm" one. Mitigation strategies:
- Use **Provisioned Concurrency** for latency-sensitive APIs (keeps environments pre-warmed, at extra cost)
- Keep deployment packages small
- Choose lighter runtimes when cold-start latency matters (e.g., Node.js/Python tend to cold-start faster than JVM-based runtimes)

## Lambda in a Serverless Architecture (Cloud Resume Challenge pattern)

```
Browser → API Gateway → Lambda → DynamoDB
                                     │
                          (e.g., visitor counter)
```

This is the typical pattern used in the Cloud Resume Challenge project in this repository: a static S3-hosted frontend calls an API Gateway endpoint, which triggers a Lambda function that reads/writes a DynamoDB table (e.g., to track resume page visits).

## Best Practices

- Keep functions small and single-purpose (one function = one job)
- Never hardcode secrets — use environment variables + AWS Secrets Manager/Parameter Store
- Set appropriate timeouts (don't leave the default if your function needs more/less time)
- Use an execution role with least-privilege permissions, scoped only to what the function needs
- Monitor with CloudWatch Logs and set up alarms for errors/throttles

## Interview Prep

**Q: What is a cold start, and how would you reduce its impact?**
A cold start is the added latency when Lambda has to initialize a fresh execution environment instead of reusing a warm one — it includes downloading code, starting the runtime, and running any init code. For latency-sensitive workloads, Provisioned Concurrency keeps a set number of environments pre-warmed, at extra cost. Keeping deployment packages small and dependencies minimal also helps.

**Q: How is Lambda billed, and how does that affect design decisions?**
Billing is based on number of invocations plus GB-seconds (memory × execution duration). Because increasing memory also increases CPU proportionally, a function can sometimes run faster — and end up *cheaper* overall — with more memory allocated, since duration drops enough to offset the higher per-millisecond cost. This is a common real-world tuning exercise.

**Q: Walk me through a serverless API architecture using Lambda.**
A client sends an HTTP request to API Gateway, which routes it to a Lambda function. The function executes business logic — potentially reading/writing to DynamoDB or another data store — and returns a response through API Gateway back to the client. No servers are provisioned or managed at any point; everything scales automatically with request volume.

**Q: When would you *not* use Lambda?**
For long-running processes (Lambda has a max 15-minute execution limit), workloads needing persistent in-memory state between invocations, or applications with very high, sustained, predictable traffic where a container-based approach (ECS/EKS) might be more cost-effective than paying per-invocation.
