# Spring Boot REST API Deployment Using AWS Lambda and API Gateway

## 1. Project Overview

This project demonstrates how to deploy a Spring Boot application as an AWS Lambda function and expose its REST APIs through Amazon API Gateway.

Instead of running the Spring Boot application on a traditional server such as EC2, the Spring Boot application is packaged as a JAR and executed through AWS Lambda.

Whenever a client sends a GET or POST request to API Gateway, API Gateway invokes the configured Lambda function.

### Architecture

```text
             Client / Postman
                    |
                    |
             HTTP GET / POST
                    |
                    v
             API Gateway
                    |
                    |
             Lambda Integration
                    |
                    v
             AWS Lambda Function
                    |
                    |
          Spring Boot Application
             (JAR Deployment)
                    |
                    v
             REST Controller
                    |
                    v
             Business Logic
                    |
                    v
             Response to Client
```

## 2. AWS Services Used

| AWS Service | Purpose |
|---|---|
| AWS Lambda | Executes the Spring Boot application |
| API Gateway | Exposes REST endpoints through HTTP |
| IAM Role | Provides permissions to Lambda |
| CloudWatch | Monitors Lambda execution logs |
| S3 (Optional) | Can store deployment JAR files |

## 3. Prerequisites

- AWS Account
- Java JDK
- Spring Boot project
- Maven
- AWS Management Console access
- Postman or cURL
- API Gateway
- AWS Lambda

## 4. Create Spring Boot Application

Create a Spring Boot application with the required REST endpoints.

Example:

```java
@RestController
@RequestMapping("/api")
public class EmployeeController {

    @GetMapping("/employees")
    public String getEmployees() {
        return "GET API executed successfully";
    }

    @PostMapping("/employees")
    public String createEmployee() {
        return "POST API executed successfully";
    }
}
```

## 5. Configure Spring Boot for AWS Lambda

A normal Spring Boot application starts using an embedded web server.

AWS Lambda requires a Lambda-compatible handler.

For Spring Boot applications, AWS Serverless Java Container can be used to adapt Spring Boot's servlet application to AWS Lambda.

Add the appropriate dependency to `pom.xml`:

```xml
<dependency>
    <groupId>com.amazonaws.serverless</groupId>
    <artifactId>aws-serverless-java-container-springboot3</artifactId>
    <version>2.1.5</version>
</dependency>
```

**Note:** Select a compatible library version and Spring Boot version for your project.

Create a Lambda handler:

```java
public class StreamLambdaHandler
        implements RequestStreamHandler {

    private static SpringBootLambdaContainerHandler<AwsProxyRequest, AwsProxyResponse> handler;

    static {
        try {
            handler = SpringBootLambdaContainerHandler.getAwsProxyHandler(
                    Application.class
            );
        } catch (ContainerInitializationException e) {
            throw new RuntimeException("Spring Boot initialization failed", e);
        }
    }

    @Override
    public void handleRequest(
            InputStream input,
            OutputStream output,
            Context context
    ) throws IOException {

        handler.proxyStream(input, output, context);
    }
}
```

Replace `Application.class` with your actual Spring Boot main application class.

## 6. Package Spring Boot Application

Build the application using Maven:

```bash
mvn clean package
```

The generated JAR will be available inside:

```text
target/
```

Ensure the JAR contains the Lambda handler and all required dependencies.

For AWS Lambda, use the deployment package format and packaging instructions supported by your selected Serverless Java Container version.

## 7. Create AWS Lambda Function

1. Open AWS Management Console.
2. Navigate to AWS Lambda.
3. Click Create Function.
4. Select Author from scratch.
5. Enter a function name.
6. Select a compatible Java runtime.
7. Select or create an IAM execution role.
8. Create the Lambda function.

### Upload Deployment Package

1. Open the Lambda function.
2. Navigate to Code.
3. Upload the deployment package.
4. Configure the handler.

Example handler:

```text
com.example.StreamLambdaHandler::handleRequest
```

Replace the package name with your actual handler package.

Configure the Lambda memory, timeout and runtime according to application requirements.

## 8. Create API Gateway

1. Open AWS API Gateway.
2. Click Create API.
3. Select HTTP API or REST API, depending on your project.
4. Configure the API integration.
5. Select AWS Lambda as the integration type.
6. Select your Lambda function.
7. Add the required routes.

Example routes:

| HTTP Method | Endpoint |
|---|---|
| GET | /api/employees |
| POST | /api/employees |

8. Configure the integration to invoke your Lambda function.
9. Deploy the API to a stage.
10. Copy the generated Invoke URL.

**Important:** API Gateway needs permission to invoke your Lambda function. Configure the Lambda resource-based permission for API Gateway.

## 9. Test GET API

Using Postman:

```http
GET https://your-api-id.execute-api.region.amazonaws.com/api/employees
```

Expected response:

```text
GET API executed successfully
```

## 10. Test POST API

Using Postman:

```http
POST https://your-api-id.execute-api.region.amazonaws.com/api/employees
```

Expected response:

```text
POST API executed successfully
```

## 11. Monitor Lambda Execution

To check whether API Gateway successfully invoked Lambda:

1. Open AWS Lambda.
2. Select your Lambda function.
3. Navigate to Monitor.
4. Open CloudWatch Logs.
5. Check the execution logs.

CloudWatch helps identify:
- Lambda invocation errors
- Spring Boot initialization issues
- API execution errors
- Timeout exceptions
- Application logs

## 12. Important Concepts to Remember

### What happens when GET or POST is called?

1. Client sends an HTTP request.
2. API Gateway receives the request.
3. API Gateway identifies the matching route and HTTP method.
4. API Gateway invokes the configured Lambda function.
5. Lambda executes the Spring Boot handler.
6. The handler forwards the request to Spring Boot.
7. Spring Boot identifies the matching controller method.
8. Controller processes the request and returns a response.
9. Lambda returns the response to API Gateway.
10. API Gateway sends the response to the client.

### Is Spring Boot running continuously?

No. Lambda is an event-driven, serverless execution environment.

Lambda can initialize the Spring Boot application during a cold start and reuse an execution environment for subsequent invocations. AWS may also create additional execution environments for concurrent requests.

### Do we need EC2?

No. In this architecture, EC2 is not required because Lambda executes the application.

### Do we need API Gateway for Lambda?

No. Lambda can also be invoked by other AWS services or directly through AWS SDK/CLI. API Gateway is used here to expose HTTP endpoints.

## 13. Troubleshooting

| Issue | Possible Solution |
|---|---|
| 502 Bad Gateway | Check Lambda response format and execution logs |
| 403 Forbidden | Check API Gateway permissions and route configuration |
| 500 Internal Server Error | Check Spring Boot and Lambda logs |
| Handler not found | Verify fully qualified handler name |
| Timeout | Increase Lambda timeout and investigate startup time |
| ClassNotFoundException | Verify deployment package and dependencies |
| Spring Boot initialization error | Check runtime and library compatibility |

## 14. Key Takeaways

- Spring Boot REST APIs can run inside AWS Lambda.
- AWS Serverless Java Container adapts Spring Boot to Lambda.
- API Gateway exposes HTTP endpoints.
- GET and POST requests invoke the same Lambda function when configured with the same integration.
- Lambda executes the Spring Boot controller logic.
- CloudWatch provides execution logs.
- EC2 is not required for this serverless architecture.

## 15. AWS Architecture Summary

```text
Postman / Browser
       |
       v
   API Gateway
       |
       v
   AWS Lambda
       |
       v
Spring Boot JAR
       |
       v
REST Controller
       |
       v
   Response
```
