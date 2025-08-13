# A Guide to AWS Deployment for Developers

This guide provides comprehensive instructions for deploying applications on Amazon Web Services (AWS). It covers three standard patterns for backend, frontend, and container-based workloads.

### Prerequisites

Before beginning, ensure you have the following:

*   An active AWS account.
*   Application code hosted in a GitHub repository.
*   Docker Desktop installed and running locally (for the ECS deployment path).

---

## Part 1: Backend Deployment on Elastic Beanstalk (EBS)

> **What is Elastic Beanstalk?**
> Elastic Beanstalk (EBS) is a managed service that orchestrates various AWS services (like EC2, S3, and Load Balancers) to host web applications. It simplifies deployment by automating resource provisioning, configuration, and scaling based on the application code provided.

### Step 1: Launch Environment

1.  Navigate to the **Elastic Beanstalk** service in the AWS Console.
2.  Click **Create application**.

### Step 2: Configure Application

1.  **Environment tier:** Select **Web server environment**.
2.  **Application name:** Assign a logical name (e.g., `my-project-api`).
3.  **Environment name:** An environment name will be auto-generated. This can be modified.
4.  **Platform:**
    *   Select **Managed platform**.
    *   From the **Platform** dropdown, choose the appropriate runtime for your backend (e.g., **Node.js**, **Python**).
    *   Use the default recommended platform branch and version.
5.  **Application code:** Select **Sample application**. The actual application code will be deployed via a CI/CD pipeline.
6.  Click **Next**.

### Step 3: Configure Service Access and Security

1.  **Create an EC2 Key Pair:**
    *   In a new browser tab, navigate to the **EC2** service.
    *   In the left navigation pane, go to `Network & Security > Key Pairs`.
    *   Click **Create key pair**.
    *   **Name:** Provide a descriptive name (e.g., `ebs-access-key`).
    *   **Key pair type:** `RSA` | **Private key file format:** `.pem`.
    *   Click **Create key pair**. A `.pem` file will be downloaded. Store this file securely, as it cannot be downloaded again.

2.  **Configure Service Role:**
    *   Return to the Elastic Beanstalk creation wizard.
    *   In the **Service access** section, select **Create and use new service role** for the **Service role**.
    *   The default `aws-elasticbeanstalk-service-role` is sufficient.
    *   For the **EC2 key pair**, select the key pair created in the previous step.
    *   The EC2 instance profile will be created and configured automatically.

### Step 4: Configure Instances

1.  Click **Configure more options**.
2.  Locate the **Instances** configuration card and click **Edit**.
3.  Set the **Root volume (boot disk) type** to `gp3`. This offers better performance and cost-efficiency than the default `gp2`.
4.  Click **Save**.

### Step 5: Review and Launch

1.  Review the configuration details.
2.  Click **Submit** to initiate the environment creation. This process will take 5-10 minutes.
3.  Upon completion, a URL will be provided. Accessing it will display the sample application.

### CI/CD Automation with GitHub Actions

1.  **Create an IAM User for GitHub:**
    *   Navigate to the **IAM** service.
    *   Go to `Users > Create users`.
    *   **User name:** `github-actions-deployer`.
    *   Choose `I want to create an IAM user` and then use opt for auto generated password
    *   On the permissions page, select **Attach existing policies directly**.
    *   Search for and attach the `AmazonS3FullAccess` policy.
    *   Proceed through the remaining steps to create the user.
    *   On the final screen, copy the **Access key ID** and **Secret access key**. These credentials are shown only once.

2.  **Configure GitHub Secrets:**
    *   In your GitHub repository, navigate to `Settings > Secrets and variables > Actions`.
    *   Create the following two repository secrets:
        *   `AWS_ACCESS_KEY_ID`: The Access Key ID from the IAM user.
        *   `AWS_SECRET_ACCESS_KEY`: The Secret Access Key from the IAM user.

3.  **Create the Workflow File:**
    *   In your repository, create the directory `.github/workflows/`.
    *   Inside this directory, create a file named `deploy-ebs.yml`.
    *   Add the following YAML configuration, updating the placeholder values as indicated.

    ```yaml
    name: Deploy to Elastic Beanstalk

    on:
      push:
        branches: [ "main" ]

    jobs:
      deploy:
        runs-on: ubuntu-latest

        steps:
        - name: Checkout Repository
          uses: actions/checkout@v4

        - name: Create Deployment Package
          run: zip -r deploy.zip . -x ".git/*" ".github/*"

        - name: Deploy to Elastic Beanstalk
          uses: einaregilsson/beanstalk-deploy@v21
          with:
            aws_access_key: ${{ secrets.AWS_ACCESS_KEY_ID }}
            aws_secret_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
            application_name: my-project-api # UPDATE with your EBS application name
            environment_name: my-project-api-env # UPDATE with your EBS environment name
            region: us-east-1 # UPDATE with your AWS region
            deployment_package: deploy.zip
    ```

---

## Part 2: Frontend Deployment on S3 & CloudFront

> **What are S3 and CloudFront?**
> **S3 (Simple Storage Service)** is a highly scalable object storage service used here to host static website assets (HTML, CSS, JavaScript). **CloudFront** is a Content Delivery Network (CDN) that caches these assets at edge locations globally, significantly reducing latency for end-users.

### Step 1: Create and Configure S3 Bucket

1.  Navigate to the **S3** service.
2.  Click **Create bucket**.
3.  **Bucket name:** Provide a globally unique name (e.g., `my-app-frontend-assets`).
4.  **Block Public Access settings:** Uncheck **Block all public access** and acknowledge the warning.
5.  Click **Create bucket**.

### Step 2: Enable Static Website Hosting

1.  Select the newly created bucket and navigate to the **Properties** tab.
2.  Scroll to **Static website hosting** and click **Edit**.
3.  Select **Enable**.
4.  **Index document:** `index.html`.
5.  **Error document:** `index.html` (common for Single Page Applications).
6.  Click **Save changes**.

### Step 3: Configure Bucket Policy

1.  Navigate to the **Permissions** tab of your bucket.
2.  Under **Bucket policy**, click **Edit**.
3.  Paste the following JSON policy, replacing `{{your-unique-bucket-name}}` with your bucket's name.

    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Sid": "PublicReadGetObject",
                "Effect": "Allow",
                "Principal": "*",
                "Action": "s3:GetObject",
                "Resource": "arn:aws:s3:::{{your-unique-bucket-name}}/*"
            }
        ]
    }
    ```
4.  Click **Save changes**.

### Step 4: Upload Frontend Assets (optional if automated with pipeline)

1.  Build your frontend application locally to generate a `build` or `dist` directory.
2.  Navigate to the **Objects** tab of your S3 bucket.
3.  Upload the *contents* of your `build` or `dist` directory to the root of the bucket.

### CI/CD Automation with GitHub Actions

1.  **Create an IAM User for GitHub:**
    *   Navigate to the **IAM** service.
    *   Go to `Users > Create users`.
    *   **User name:** `github-actions-deployer`.
    *   Choose `I want to create an IAM user` and then use opt for auto generated password
    *   On the permissions page, select **Attach existing policies directly**.
    *   Search for and attach the `AmazonS3FullAccess` policy.
    *   Proceed through the remaining steps to create the user.
    *   On the final screen, copy the **Access key ID** and **Secret access key**. These credentials are shown only once.

2.  **Configure GitHub Secrets:**
    *   In your GitHub repository, navigate to `Settings > Secrets and variables > Actions`.
    *   Create the following two repository secrets:
        *   `AWS_ACCESS_KEY_ID`: The Access Key ID from the IAM user.
        *   `AWS_SECRET_ACCESS_KEY`: The Secret Access Key from the IAM user.

3.  **Create the Workflow File:**
    *   In your repository, create the directory `.github/workflows/`.
    *   Inside this directory, create a file named `main.yml`.
    *   Add the following YAML configuration, updating the placeholder values as indicated.

    ```yaml
    name: GitHub Pages
    on:
       push:
          branches:
             - master
    jobs:
       deploy:
          runs-on: ubuntu-latest
          steps:
            # Configure AWS Credentials
            - name: Configure AWS Credentials
              uses: aws-actions/configure-aws-credentials@v1
              with:
                aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
                aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
                aws-region: us-west-1

            # Checkout the repository
            - name: Checkout
              uses: actions/checkout@v2

            # Cache Node.js dependencies
            - name: Cache Node.js modules
              uses: actions/cache@v3
              with:
                path: ~/.npm
                key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
                restore-keys: |
                  ${{ runner.os }}-node-

            # Setup Node.js environment
            - name: Setup Node.js
              uses: actions/setup-node@v2
              with:
                node-version: 18
      
            # Install dependencies
            - name: Install dependencies
              run: npm ci --prefer-offline
      
            # Build the application
            - name: Build
              run: npm run build
      
            # Deploy to S3
            - name: Deploy to S3
              run: aws s3 sync ./{{build folder}} s3://{{bucket_name}} --delete
      
            # Invalidate CloudFront cache
            - name: Invalidate CloudFront Cache
              uses: chetan/invalidate-cloudfront-action@v2
              env:
                DISTRIBUTION: ${{ secrets.AWS_DISTRIBUTATION_ID }}
                PATHS: "/*"
                AWS_REGION: us-west-1
                AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
                AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

    ```


### Step 5: Create CloudFront Distribution

1.  Navigate to the **CloudFront** service.
2.  Click **Create a CloudFront distribution**.
3.  **Origin domain:** Select your S3 bucket from the dropdown list.
4.  **Viewer protocol policy:** Select **Redirect HTTP to HTTPS**.
5.  **Default root object:** Enter `index.html`.
6.  Click **Create distribution**.
7.  Distribution deployment can take 10-15 minutes. Once the status is `Enabled`, use the **Distribution domain name** (e.g., `d12345abcdef.cloudfront.net`) to access your site.

---

## Part 3: Container Deployment on ECS with Fargate

> **What is ECS?**
> **Elastic Container Service (ECS)** is a container orchestration service for deploying, managing, and scaling Docker containers. When used with **AWS Fargate**, it provides a serverless compute engine, removing the need to manage the underlying EC2 instances.

### Step 1: Create a Dockerfile

In the root of your application repository, create a `Dockerfile`.

```dockerfile
# Use an official Python runtime as a base image
FROM python:3.10-slim

# Set the working directory inside the container
WORKDIR /app

# Copy requirements first for better layer caching
COPY requirements.txt .

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy the rest of the application code
COPY . .

# Expose the application port
EXPOSE 8000

# Command to start the application (example for Uvicorn/FastAPI)
CMD ["uvicorn", "application:application", "--host", "0.0.0.0", "--port", "8000", "--workers", "4"]
```

### Step 2: Create an ECR Repository

1.  Navigate to the **Elastic Container Registry (ECR)** service.
2.  Click **Create repository**.
3.  **Visibility settings:** Choose **Private**.
4.  **Repository name:** Provide a name (e.g., `my-python-app`).
5.  Click **Create repository**. Note the repository **URI** for later use.

### Step 3: Create an ECS Cluster

1.  Navigate to the **ECS** service.
2.  In the left navigation pane, click **Clusters**, then **Create cluster**.
3.  **Cluster name:** `my-app-cluster`.
4.  **Infrastructure:** Select **AWS Fargate (Serverless)**.
5.  Click **Create**.

### Step 4: Create a Task Definition

1.  In ECS, select **Task Definitions** from the navigation pane, then **Create new task definition**.
2.  **Task definition family:** `my-app-task`.
3.  **Launch type:** **AWS Fargate**.
4.  **Task size:** Select an appropriate CPU and memory configuration (e.g., `0.5 vCPU`, `1 GB memory`).
5.  In the **Container - 1** section:
    *   **Name:** `my-python-app`.
    *   **Image URI:** Enter the ECR repository URI from Step 2, appended with `:latest`.
    *   **Port mappings:** Add a mapping for the port your application exposes (e.g., `8000`).
6.  Click **Create**.

### Step 5: Create an ECS Service

1.  Navigate to your ECS cluster and select the **Services** tab. Click **Create**.
2.  **Launch type:** **FARGATE**.
3.  **Family:** Select the task definition created previously.
4.  **Service name:** `my-app-service`.
5.  **Desired tasks:** `1`.
6.  In the **Networking** section:
    *   Select a VPC and subnets (defaults are acceptable for initial setup).
    *   For **Security group**, click **Create new security group**. Name it `my-app-sg` and add an inbound rule for `Custom TCP` on port `8000` from source `Anywhere-IPv4 (0.0.0.0/0)`.
    *   Enable **Public IP**.
7.  Click **Create**.

### Step 6: Automate with GitHub Actions for ECS

1.  **Configure IAM User and GitHub Secrets:**
    *   Use the `github-actions-deployer` IAM user from the EBS setup.
    *   Attach an additional policy: `AmazonEC2ContainerRegistryFullAccess`.
    *   Use the same `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` secrets in GitHub.

2.  **Configure GitHub Variables:**
    *   In your repository `Settings > Secrets and variables > Actions`, go to the **Variables** tab.
    *   Create the following repository variables:
        *   `AWS_REGION`: The AWS region of your resources (e.g., `us-east-1`).
        *   `ECR_REPOSITORY`: The name of your ECR repository (e.g., `my-python-app`).
        *   `ECS_CLUSTER`: The name of your ECS cluster (e.g., `my-app-cluster`).
        *   `ECS_SERVICE`: The name of your ECS service (e.g., `my-app-service`).
        *   `ECS_TASK_DEFINITION`: The filename for the task definition JSON (e.g., `task-definition.json`).
        *   `CONTAINER_NAME`: The container name from your task definition (e.g., `my-python-app`).

3.  **Create the Workflow File:** Create `.github/workflows/deploy-ecs.yml` in your repository.

    ```yaml
    name: Deploy to Amazon ECS

    on:
      push:
        branches: [ "main" ]

    env:
      AWS_REGION: ${{ vars.AWS_REGION }}
      ECR_REPOSITORY: ${{ vars.ECR_REPOSITORY }}
      ECS_CLUSTER: ${{ vars.ECS_CLUSTER }}
      ECS_SERVICE: ${{ vars.ECS_SERVICE }}
      ECS_TASK_DEFINITION: ${{ vars.ECS_TASK_DEFINITION }}
      CONTAINER_NAME: ${{ vars.CONTAINER_NAME }}

    jobs:
      deploy:
        name: Deploy
        runs-on: ubuntu-latest
        permissions:
          contents: read

        steps:
        - name: Checkout
          uses: actions/checkout@v4

        - name: Configure AWS credentials
          uses: aws-actions/configure-aws-credentials@v4
          with:
            aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
            aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
            aws-region: ${{ env.AWS_REGION }}

        - name: Login to Amazon ECR
          id: login-ecr
          uses: aws-actions/amazon-ecr-login@v2

        - name: Build, tag, and push image to Amazon ECR
          id: build-image
          env:
            ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
            IMAGE_TAG: ${{ github.sha }}
          run: |
            docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
            docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
            echo "image=$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG" >> $GITHUB_OUTPUT

        - name: Download existing task definition
          run: |
            aws ecs describe-task-definition --task-definition ${{ vars.ECS_TASK_DEFINITION | replace('.json', '') }} --query taskDefinition > ${{ env.ECS_TASK_DEFINITION }}

        - name: Fill in new image ID in task definition
          id: task-def
          uses: aws-actions/amazon-ecs-render-task-definition@v1
          with:
            task-definition: ${{ env.ECS_TASK_DEFINITION }}
            container-name: ${{ env.CONTAINER_NAME }}
            image: ${{ steps.build-image.outputs.image }}

        - name: Deploy new task definition to Amazon ECS
          uses: aws-actions/amazon-ecs-deploy-task-definition@v1
          with:
            task-definition: ${{ steps.task-def.outputs.task-definition }}
            service: ${{ env.ECS_SERVICE }}
            cluster: ${{ env.ECS_CLUSTER }}
            wait-for-service-stability: true
    ```
