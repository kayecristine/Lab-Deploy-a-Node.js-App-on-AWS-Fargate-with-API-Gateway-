# Lab: Deploy a Node.js App on AWS Fargate (Windows Edition - Troubleshooting Included)

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
* **1-b. Create an IAM User with Programmatic Access:**
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


