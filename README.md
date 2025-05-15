
# 🛠️ MERN Stack Blog App Deployment with Terraform and Ansible

## 📋 Overview -- Student: SDA1011 Abdulaziz

This project demonstrates the deployment of a full-stack MERN blog application using **Terraform** for infrastructure provisioning and **Ansible** for configuration management. The goal is to automate the entire lifecycle, from provisioning cloud resources to running and serving the application backend and frontend.

---

## 🚀 Architecture

```
┌────────────────────┐        ┌──────────────────────┐        ┌──────────────────────┐
│ React Frontend     │        │ Node.js Backend      │        │ MongoDB Atlas        │
│ (S3 Static Website)│───────▶│ (EC2 - Ubuntu 22.04) │───────▶│ (Cloud Database)     │
└────────────────────┘        └──────────────────────┘        └──────────────────────┘
                                        │
                                        ▼
                              ┌────────────────────┐
                              │ S3 Media Storage   │
                              │ (IAM Secured)      │
                              └────────────────────┘
```

---

## ⚙️ Tools & Technologies

- **Terraform** – Infrastructure as Code (IaC) for AWS provisioning
- **Ansible** – Configuration automation for backend setup and frontend deployment
- **AWS EC2** – Backend hosting (Ubuntu 22.04)
- **S3 Buckets** – Frontend hosting & media uploads
- **MongoDB Atlas** – Fully managed NoSQL database
- **PM2** – Node.js process manager

---

## 📦 Prerequisites

- AWS CLI configured (IAM credentials)
- SSH key pair created in AWS
- Terraform ≥ v1.0
- Ansible ≥ v2.10
- MongoDB Atlas account and a free-tier cluster

---

## 🌐 Infrastructure Setup (Terraform)

1. Navigate to the Terraform configuration directory:

   ```bash
   cd terraform
   ```

2. Define the following variables:
   - SSH key name
   - AWS region
   - Unique names for S3 buckets (frontend + media)

3. Initialize and apply the infrastructure:

   ```bash
   terraform init
   terraform apply
   ```

4. Upon completion, take note of:
   - EC2 public IP
   - S3 bucket names and URLs
   - IAM user credentials for media upload
   - Frontend bucket website endpoint

5. Retrieve and store outputs as environment variables or in an Ansible variables file:

   ```bash
   terraform output -raw ec2_public_ip
   terraform output -raw s3_user_access_key
   terraform output -raw s3_user_secret_key
   # ...and others
   ```

---

## 🔐 Secure Access (Linux SSH Setup)

Ensure your SSH key is placed in `~/.ssh` and permissions are properly configured:

```bash
chmod 600 ~/.ssh/your-key.pem
```

Update your Ansible inventory accordingly:

```ini
[backend]
backend_server ansible_host=<EC2_PUBLIC_IP> ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/your-key.pem

[all:vars]
ansible_python_interpreter=/usr/bin/python3
```

---

## 🔧 Application Deployment (Ansible)

1. Move into the Ansible directory:

   ```bash
   cd ../ansible
   ```

2. Create an `extra_vars.yml` with Terraform output values:

   ```yaml
   ---
   s3_user_access_key: "<your-access-key>"
   s3_user_secret_key: "<your-secret-key>"
   media_bucket_name: "<media-bucket-name>"
   media_bucket_url: "<media-bucket-url>"
   frontend_bucket_name: "<frontend-bucket-name>"
   frontend_bucket_website_endpoint: "<frontend-url>"
   ec2_public_ip: "<ec2-public-ip>"
   ```

3. Run the playbook:

   ```bash
   ansible-playbook -i inventory/hosts deploy-playbook.yml --extra-vars "@extra_vars.yml"
   ```

This will:
- Set up Node.js, NVM, and PM2
- Clone the blog app repo and configure environment variables
- Install dependencies and launch the backend
- Build the frontend and deploy it to S3

---

## 🔍 Testing

- **Frontend URL:** Access via the S3 static website endpoint provided in Terraform output.
- **Backend API:** Confirm availability at `http://<EC2_PUBLIC_IP>:5000/api`
- **Media Uploads:** Create a blog post with an image to verify S3 upload functionality.
- **MongoDB:** Validate that data is written to your Atlas cluster.

---

## 🧼 Cleanup

After verifying the deployment, you can safely tear down all resources:

```bash
cd terraform
terraform destroy
```

Don’t forget to:
- Delete your MongoDB user and cluster if created for this project
- Remove IP whitelists on MongoDB Atlas
- Revoke IAM credentials used for media uploads

---

## ✅ Deployment Summary

| Component        | Status          |
|------------------|-----------------|
| EC2 Backend      | ✅ Running       |
| MongoDB Atlas    | ✅ Connected     |
| S3 Frontend      | ✅ Hosted        |
| Media Uploads    | ✅ Functional    |
| Terraform Infra  | ✅ Automated     |
| Ansible Setup    | ✅ Complete      |

---

## 📎 Notes

- Ensure `.env` values are excluded from version control
- If you encounter permission or dependency errors, check Ansible logs or SSH into EC2 for troubleshooting
- Consider using Secrets Manager or SSM Parameter Store for managing credentials in production
