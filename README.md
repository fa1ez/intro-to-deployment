# 🚀 Intro to Deployment

## 🖥️ Backend Deployment on AWS Elastic Beanstalk (EBS)

### 1. **Launch Elastic Beanstalk**
Navigate to **Elastic Beanstalk** in the AWS Console.

---

### 2. **Create Environment**
- Click **Create environment**
- Choose **Web server environment**

---

### 3. **Configure Application**
- **Application name**: `your-app-name`
- **Environment name**: `your-env-name`
- **Platform**: Choose a **managed platform** (e.g., **Node.js**)

---

### 4. **Service Access**

#### ✅ IAM Role with EBS Full Access
- Select an **IAM role** with **Elastic Beanstalk full access**

#### 🔐 EC2 Key Pair
- Go to `EC2 > Network & Security > Key Pairs`
- Create a new key pair:
  - Choose a **name**
  - Select **RSA**
  - Download the **.pem file**

#### 🛡️ Create a Role for EBS
- Go to **IAM > Roles > Create role**
- Choose any **use case**
- **Attach these permissions**:

  ![Permissions Screenshot](https://github.com/user-attachments/assets/467553bf-ec6b-4c63-af3f-caeb7c4f6f39)

> ✅ **Skip the next step in role creation**

---

### 5. **Configure EC2 Instance**

- In the **Instances** section:
  - Set **Root volume type** to `gp3 (SSD)`
- Keep all other settings default

---

### 🎉 Deployment Complete
A sample application will now be deployed automatically!

### Post deployemt using github actions
- create an IAM user if not already created and download the secrets of IAM user that you want to run github actions with
- add those secrets in github and add the workflow file attached in repo to the project and update name of project, env and region

<br>
<br>
<br>

<br>
<br>
<br>

## 🖥️ Frontend Deployemnt on S3 Bucket and cloudfront

### 1. Create S3 Bucket
- Give a **unique bucket name**.
- **Uncheck** "Block all public access".
- Click **Save changes**.

---

### 2. Enable Static Website Hosting
- Go to `Bucket > Properties > Static website hosting`.
  - Enable **Static website hosting**.
  - For **Index document**, enter: `index.html`.
  - For **Error document**, enter: `index.html` (if no separate error file is present).
- Save changes.

---

### 3. Update Bucket Permissions

#### 🛡️ Bucket Policy
- Go to `Bucket > Permissions > Bucket Policy`.
- Add the following policy:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::{{bucketName}}/*"
        }
    ]
}
```
- update CORS settings as well:

```json
[
    {
        "AllowedHeaders": [
            "*"
        ],
        "AllowedMethods": [
            "GET",
            "PUT",
            "POST",
            "HEAD"
        ],
        "AllowedOrigins": [*],
        "ExposeHeaders": [
            "ETag"
        ],
        "MaxAgeSeconds": 3000
    }
]
```
