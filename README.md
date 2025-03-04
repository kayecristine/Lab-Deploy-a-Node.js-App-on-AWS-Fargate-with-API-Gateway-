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
    * Go to AWS
