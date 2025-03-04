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
