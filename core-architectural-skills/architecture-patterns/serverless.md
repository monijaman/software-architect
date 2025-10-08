# ⚡ Serverless Architecture

Serverless architecture is a cloud computing model where the cloud provider manages the infrastructure layer, allowing developers to focus purely on writing code. Functions are executed in response to events without managing servers or scaling concerns.

## Table of Contents
- [Core Concepts](#core-concepts)
- [Function as a Service (FaaS)](#function-as-a-service-faas)
- [Backend as a Service (BaaS)](#backend-as-a-service-baas)
- [Serverless Patterns](#serverless-patterns)
- [Implementation Examples](#implementation-examples)
- [Benefits & Challenges](#benefits--challenges)
- [Best Practices](#best-practices)

---

## Core Concepts

### Function as a Service (FaaS)
**FaaS** allows you to run code in response to events without managing servers. The cloud provider handles scaling, patching, and infrastructure management.

### Backend as a Service (BaaS)
**BaaS** provides managed backend services like databases, authentication, and storage, eliminating the need to build and maintain these components.

### Event-Driven Execution
Serverless functions are triggered by events such as:
- HTTP requests (API Gateway)
- Database changes
- File uploads
- Scheduled tasks
- Queue messages
- IoT sensor data

### Stateless Functions
Functions should be stateless and idempotent, with any required state stored in external services like databases or caches.

---

## Function as a Service (FaaS)

### AWS Lambda Example
```javascript
// AWS Lambda function triggered by API Gateway
exports.handler = async (event, context) => {
  try {
    const { httpMethod, path, body } = event;

    switch(`${httpMethod} ${path}`) {
      case 'POST /users':
        return await createUser(JSON.parse(body));
      case 'GET /users':
        return await getUsers();
      default:
        return {
          statusCode: 404,
          body: JSON.stringify({ error: 'Not found' })
        };
    }
  } catch (error) {
    console.error('Error:', error);
    return {
      statusCode: 500,
      body: JSON.stringify({ error: 'Internal server error' })
    };
  }
};

async function createUser(userData) {
  // Validate input
  if (!userData.email || !userData.name) {
    return {
      statusCode: 400,
      body: JSON.stringify({ error: 'Email and name required' })
    };
  }

  // Save to database
  const user = await saveUserToDynamoDB(userData);

  return {
    statusCode: 201,
    body: JSON.stringify(user)
  };
}
```

### Azure Functions Example
```csharp
// Azure Function triggered by HTTP request
public static class UserFunctions
{
    [FunctionName("CreateUser")]
    public static async Task<IActionResult> CreateUser(
        [HttpTrigger(AuthorizationLevel.Function, "post", Route = "users")] HttpRequest req,
        ILogger log)
    {
        log.LogInformation("Creating new user");

        string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
        var userData = JsonConvert.DeserializeObject<UserData>(requestBody);

        if (string.IsNullOrEmpty(userData?.Email) || string.IsNullOrEmpty(userData?.Name))
        {
            return new BadRequestObjectResult("Email and name are required");
        }

        var user = await SaveUserToCosmosDB(userData);

        return new CreatedResult($"/users/{user.Id}", user);
    }
}
```

---

## Backend as a Service (BaaS)

### Authentication with Firebase Auth
```javascript
// Firebase Authentication integration
const admin = require('firebase-admin');
admin.initializeApp();

exports.verifyToken = async (req, res, next) => {
  const authHeader = req.headers.authorization;

  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'Unauthorized' });
  }

  const token = authHeader.split(' ')[1];

  try {
    const decodedToken = await admin.auth().verifyIdToken(token);
    req.user = decodedToken;
    next();
  } catch (error) {
    res.status(401).json({ error: 'Invalid token' });
  }
};
```

### Database with FaunaDB
```javascript
// FaunaDB serverless database operations
const faunadb = require('faunadb');
const q = faunadb.query;

const client = new faunadb.Client({
  secret: process.env.FAUNA_SECRET
});

exports.createUser = async (userData) => {
  return await client.query(
    q.Create(
      q.Collection('users'),
      {
        data: {
          ...userData,
          createdAt: q.Now()
        }
      }
    )
  );
};

exports.getUser = async (userId) => {
  return await client.query(
    q.Get(q.Ref(q.Collection('users'), userId))
  );
};
```

---

## Serverless Patterns

### API Gateway Pattern
**API Gateway Pattern** uses a serverless API gateway to route requests to appropriate functions and handle cross-cutting concerns.

```yaml
# AWS API Gateway + Lambda configuration
functions:
  createUser:
    handler: src/handlers/users.create
    events:
      - http:
          path: users
          method: post
          cors: true

  getUser:
    handler: src/handlers/users.get
    events:
      - http:
          path: users/{id}
          method: get
          cors: true

  listUsers:
    handler: src/handlers/users.list
    events:
      - http:
          path: users
          method: get
          cors: true
```

### Event Streaming Pattern
**Event Streaming Pattern** processes events from streams like Kinesis or EventBridge using serverless functions.

```javascript
// AWS Lambda processing Kinesis stream
exports.processEvents = async (event) => {
  const records = event.Records;

  for (const record of records) {
    try {
      const eventData = JSON.parse(
        Buffer.from(record.kinesis.data, 'base64').toString()
      );

      await processEvent(eventData);
    } catch (error) {
      console.error('Error processing event:', error);
      // Send to dead letter queue or retry
    }
  }
};

async function processEvent(eventData) {
  switch(eventData.type) {
    case 'user.created':
      await sendWelcomeEmail(eventData.userId);
      break;
    case 'order.placed':
      await updateInventory(eventData.items);
      break;
  }
}
```

### Scheduled Tasks Pattern
**Scheduled Tasks Pattern** runs functions on a schedule using cloud scheduler services.

```javascript
// AWS Lambda with CloudWatch Events (scheduled)
exports.dailyCleanup = async (event) => {
  console.log('Running daily cleanup');

  try {
    // Clean up old temporary files
    await cleanupTempFiles();

    // Archive old logs
    await archiveOldLogs();

    // Send daily report
    await sendDailyReport();

    console.log('Daily cleanup completed');
  } catch (error) {
    console.error('Daily cleanup failed:', error);
    throw error; // Lambda will retry
  }
};
```

### Fan-Out Pattern
**Fan-Out Pattern** processes a single event by triggering multiple parallel functions.

```javascript
// AWS Step Functions orchestrating fan-out
{
  "Comment": "Process order with fan-out",
  "StartAt": "ValidateOrder",
  "States": {
    "ValidateOrder": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:validate-order",
      "Next": "ProcessPaymentAndInventory"
    },
    "ProcessPaymentAndInventory": {
      "Type": "Parallel",
      "Branches": [
        {
          "StartAt": "ProcessPayment",
          "States": {
            "ProcessPayment": {
              "Type": "Task",
              "Resource": "arn:aws:lambda:process-payment",
              "End": true
            }
          }
        },
        {
          "StartAt": "UpdateInventory",
          "States": {
            "UpdateInventory": {
              "Type": "Task",
              "Resource": "arn:aws:lambda:update-inventory",
              "End": true
            }
          }
        }
      ],
      "Next": "SendConfirmation"
    },
    "SendConfirmation": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:send-confirmation",
      "End": true
    }
  }
}
```

---

## Implementation Examples

### File Processing Pipeline
```javascript
// AWS Lambda processing S3 file uploads
const AWS = require('aws-sdk');
const s3 = new AWS.S3();
const sharp = require('sharp');

exports.processImage = async (event) => {
  const bucket = event.Records[0].s3.bucket.name;
  const key = decodeURIComponent(event.Records[0].s3.object.key);

  try {
    // Get original image
    const originalImage = await s3.getObject({
      Bucket: bucket,
      Key: key
    }).promise();

    // Resize image
    const resizedImage = await sharp(originalImage.Body)
      .resize(800, 600, { fit: 'inside' })
      .jpeg({ quality: 80 })
      .toBuffer();

    // Save resized image
    const newKey = `resized/${key}`;
    await s3.putObject({
      Bucket: bucket,
      Key: newKey,
      Body: resizedImage,
      ContentType: 'image/jpeg'
    }).promise();

    console.log(`Processed image: ${key} -> ${newKey}`);
  } catch (error) {
    console.error('Error processing image:', error);
    throw error;
  }
};
```

### Real-time Data Processing
```javascript
// AWS Lambda processing IoT sensor data
const AWS = require('aws-sdk');
const dynamodb = new AWS.DynamoDB.DocumentClient();

exports.processSensorData = async (event) => {
  const records = event.Records;

  const batchWriteItems = records.map(record => {
    const sensorData = JSON.parse(record.body);

    return {
      PutRequest: {
        Item: {
          sensorId: sensorData.sensorId,
          timestamp: sensorData.timestamp,
          temperature: sensorData.temperature,
          humidity: sensorData.humidity,
          processedAt: new Date().toISOString()
        }
      }
    };
  });

  try {
    await dynamodb.batchWrite({
      RequestItems: {
        'SensorData': batchWriteItems
      }
    }).promise();

    // Check for alerts
    for (const record of records) {
      const sensorData = JSON.parse(record.body);

      if (sensorData.temperature > 30) {
        await sendTemperatureAlert(sensorData);
      }
    }

    console.log(`Processed ${records.length} sensor readings`);
  } catch (error) {
    console.error('Error processing sensor data:', error);
    throw error;
  }
};
```

---

## Benefits & Challenges

### Benefits
- **Zero Infrastructure Management**: Focus on code, not servers
- **Automatic Scaling**: Handle any load automatically
- **Cost Efficiency**: Pay only for actual execution time
- **Fast Deployment**: Deploy functions independently
- **Built-in High Availability**: Cloud provider handles redundancy
- **Event-Driven**: Natural fit for event-driven architectures

### Challenges
- **Cold Starts**: Initial execution latency for infrequently used functions
- **Vendor Lock-in**: Tied to specific cloud provider's services
- **Debugging Complexity**: Distributed tracing across functions
- **Testing**: Need to test functions in isolation and integration
- **Monitoring**: Track performance across many small functions
- **State Management**: Handle state externally (databases, caches)
- **Cost Monitoring**: Unexpected costs from high-frequency execution

---

## Best Practices

### Function Design
1. **Keep Functions Small**: Single responsibility principle
2. **Optimize Package Size**: Minimize deployment package size
3. **Use Appropriate Memory**: Balance cost and performance
4. **Handle Timeouts**: Set appropriate timeout limits
5. **Environment Variables**: Use for configuration, not secrets

### Performance Optimization
1. **Minimize Cold Starts**: Keep functions warm with scheduled invocations
2. **Connection Reuse**: Reuse database connections across invocations
3. **Caching**: Use in-memory caches for frequently accessed data
4. **Async Processing**: Don't block on synchronous operations
5. **Batch Operations**: Process multiple items in single invocation

### Security Best Practices
1. **Least Privilege**: Grant minimal required permissions
2. **Input Validation**: Validate all inputs thoroughly
3. **Secrets Management**: Use secure secret storage services
4. **VPC Configuration**: Place functions in VPC for additional security
5. **Logging**: Comprehensive logging without exposing sensitive data

### Cost Optimization
1. **Right-sizing**: Choose appropriate memory allocation
2. **Monitor Usage**: Track function execution metrics
3. **Optimize Triggers**: Minimize unnecessary invocations
4. **Caching**: Reduce database and API calls
5. **Batch Processing**: Process multiple events together

### Common Pitfalls to Avoid
- **Monolithic Functions**: Don't put everything in one large function
- **Synchronous Dependencies**: Avoid tight coupling between functions
- **Large Dependencies**: Minimize package size and dependencies
- **Infinite Loops**: Careful with recursive function calls
- **Resource Leaks**: Always clean up resources properly

---

*Return to [Architecture Patterns Overview](./README.md) or [Core Architectural Skills](../README.md)*