# Lab: Deploy a Node.js App on AWS Fargate

This lab guides you through deploying a Node.js application on AWS Fargate using Docker, ECS, and ECR. It's designed for users authenticating via IAM User and working from their local Windows machine.

## 1. Prerequisites

* **1-a. Install Required Tools:**
    * **AWS CLI v2:**
        * Download and install the AWS CLI v2 from the official AWS website.
        * Verify installation: `aws --version`
    * **Docker Desktop for Windows:**
        * Download and install Docker Desktop for Windows.
        * Verify installation: `docker --version`
        * Verify Docker Desktop is running linux containers: Right-click the Docker icon in the system tray, and verify that "switch to linux containers" is not greyed out. If it is greyed out, click it.
    * **Node.js:**
        * Download and install Node.js from the official Node.js website.
        * Verify installation: `node -v`
* **1-b. (Only If No IAM Users Exist) Create an IAM User with Programmatic Access:**
*❗ Use this step ONLY if there are no existing IAM users with the required permissions. If IAM users are already available, use their credentials instead.
    * Go to AWS Console → IAM → Users → Add user.
    * Enter a username (e.g., `fargate-user`).
    * Under Access type, select ☑️ Access key - Programmatic access.
    * Attach the `AdministratorAccess` policy (for simplicity, or create custom permissions for ECS, ECR, and API Gateway). **Important:** Later in production you should use the least privilege principal.
    * Click Create user, then copy the Access Key ID & Secret Access Key.
* **1-c. Configure AWS CLI with the IAM user:**
    * Open PowerShell or Command Prompt.
    * Run: `aws configure`
    * Enter the Access Key ID & Secret Access Key.
    * Set your AWS region (e.g., `ap-southeast-1`).
    * Choose `json` as the output format.
    * Verify configuration: `aws sts get-caller-identity`
        * ✅ You should see your IAM User ARN.

## 2. Create a Simple Node.js App

* **2-a.** Create a project folder: `mkdir my-fargate-app && cd my-fargate-app`
* **2-b.** Initialize a Node.js project: `npm init -y`
* **2-c.** Install Express.js: `npm install express`
* **2-d.** Create `server.js` file: `code server.js` (or `notepad server.js`)
* **2-e.** Add the following code:

```javascript
const express = require('express');
const app = express();
const port = process.env.PORT || 3000;

app.get('/', (req, res) => {
    res.send('Hello from Fargate!');
});

app.listen(port, () => {
    console.log(`Server running on port ${port}`);
});
```

* **2-f.** Test locally: `node server.js`
* **2-g.** Open http://localhost:3000 in your browser.

* ## 3. Build and Push Docker Image to ECR

* **3-a.** Create a `Dockerfile`: In the `my-fargate-app` directory, create a file named `Dockerfile` (no extension).
* **3-b.** Add the following content:

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

* **3-c.** Create an ECR Repository (in your chosen region, e.g., `ap-southeast-1`): `aws ecr create-repository --repository-name my-node-app --region ap-southeast-1`
* **3-d.** Find your ECR URL: `aws ecr describe-repositories --region ap-southeast-1 | findstr repositoryUri`
* **3-e.** Authenticate Docker with ECR: Replace `<aws_account_id>` with your AWS account ID.

```bash
aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.ap-southeast-1.amazonaws.com
```

* **3-f.** Build & Push Image:

```bash
docker build -t my-node-app .
docker tag my-node-app:latest <aws_account_id>[.dkr.ecr.ap-southeast-1.amazonaws.com/my-node-app:latest](https://www.google.com/search?q=https://.dkr.ecr.ap-southeast-1.amazonaws.com/my-node-app:latest)
docker push <aws_account_id>[.dkr.ecr.ap-southeast-1.amazonaws.com/my-node-app:latest](https://www.google.com/search?q=https://.dkr.ecr.ap-southeast-1.amazonaws.com/my-node-app:latest)
```

## 4. Deploy to AWS Fargate (ECS)

* **4-a.** Create an ECS Cluster: `aws ecs create-cluster --cluster-name my-fargate-cluster --region ap-southeast-1`
* **4-b.** Create Task Definition: Create a file named `task-def.json` in the `my-fargate-app` directory.
* **4-c.** Add the following content (replace `<aws_account_id>`):

```json
{
  "family": "my-node-app",
  "networkMode": "awsvpc",
  "executionRoleArn": "arn:aws:iam::<aws_account_id>:role/ecsTaskExecutionRole",
  "containerDefinitions": [
    {
      "name": "my-node-app",
      "image": "<aws_account_id>[.dkr.ecr.ap-southeast-1.amazonaws.com/my-node-app:latest](https://www.google.com/search?q=https://.dkr.ecr.ap-southeast-1.amazonaws.com/my-node-app:latest)",
      "portMappings": [
        {
          "containerPort": 3000,
          "hostPort": 3000
        }
      ]
    }
  ],
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512"
}
```

* **4-d.** Create `ecsTaskExecutionRole`: Go to IAM, Roles, create role, select AWS Service, select Elastic Container Service, select Elastic Container Service Task, next permissions, search for and attach the `AmazonECSTaskExecutionRolePolicy` policy, next tags, next review, name the role `ecsTaskExecutionRole`.
* **4-e.** Then replace the `<aws_account_id>` in the `executionRoleArn` with your aws account ID.
* **4-f.** Register the task definition: `aws ecs register-task-definition --cli-input-json file://task-def.json --region ap-southeast-1`
* **4-g.** Create a Security Group: Go to the EC2 console, and create a security group in the same VPC as your subnets. Add an inbound rule to allow TCP traffic on port 3000 from `0.0.0.0/0`.
* **4-h.** Verify Subnet settings: Go to the VPC console, and select subnets. Select two public subnets that are in different Availability zones. For each subnet, click actions, edit subnet settings, and verify that "Enable auto-assign public IPv4 address" is checked.
* **4-i.** Run the Task on Fargate: Replace `subnet-xxxxx` and `sg-xxxxx` with your subnet and security group IDs.

```bash
aws ecs run-task --cluster my-fargate-cluster --task-definition my-node-app --launch-type FARGATE --network-configuration "awsvpcConfiguration={subnets=[subnet-xxxxx, subnet-xxxxx],securityGroups=[sg-xxxxx],assignPublicIp=ENABLED}" --region ap-southeast-1
```

* **4-j.** Get the Public IP Address (Manual Console Retrieval):
    * Go to the AWS ECS console.
    * Select your cluster (`my-fargate-cluster`).
    * Go to the "Tasks" tab.
    * Find the running task for your `my-node-app` service and click on the Task ID.
    * Go to the "Configuration" tab.
    * In the "Network" section, find the "Public IP address" and copy it.

* **4-k.** Test the Application: Open a web browser and navigate to `http://<public-ip-of-fargate-task>:3000`. You should see "Hello from Fargate!" in your browser.

# 5. Create the HTTP API

* **5-a.** First, create a new HTTP API:

```powershell
aws apigatewayv2 create-api `
  --name my-http-api `
  --protocol-type HTTP
```

* **5-b.** Create an HTTP Proxy Integration

Now, create an HTTP Proxy integration that forwards requests to your backend.

```powershell
aws apigatewayv2 create-integration `
  --api-id 3tv85g48d5 `
  --integration-type HTTP_PROXY `
  --integration-uri http://<your-backend-ip>:3000 `  # Replace with your actual backend URL
  --integration-method GET `
  --payload-format-version "1.0"
```
   * This will return an `IntegrationId` (e.g., i0o805b). Save it for the next step.

* **5-c.** Create a Route to Attach the Integration

You need to define a route that will handle requests, such as `GET /my-endpoint`:

```powershell
aws apigatewayv2 create-route `
  --api-id 3tv85g48d5 `
  --route-key "GET /my-endpoint" `
  --target "integrations/i0o805b"
```
   * Replace "i0o805b" with the actual IntegrationId from 5-b.

* **5-d.** Create a Stage & Deploy the API

Now, create a stage (e.g., `"prod"`) and deploy the API:

```powershell
aws apigatewayv2 create-stage `
  --api-id 3tv85g48d5 `
  --stage-name prod `
  --auto-deploy
```

This ensures that changes to the API are automatically deployed, so you don’t have to manually redeploy in the future.

Test the API

* **5-e.** Check if your API Gateway is now forwarding requests properly:

```powershell
curl https://3tv85g48d5.execute-api.ap-southeast-1.amazonaws.com/prod/my-endpoint
```

   * Replace 3tv85g48d5 with your actual API Gateway ID.

If your backend is working correctly, you should get a valid response from http://<your-backend-ip>:3000.

