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
