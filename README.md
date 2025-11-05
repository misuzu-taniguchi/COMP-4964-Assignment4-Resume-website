# 📘 Assignment 4 – CI/CD Pipeline Documentation
**Author:** Misuzu Taniguchi  
**Course:** COMP 4964 – DevOps  
**Project:** Resume Website Deployment on AWS

---

## 🧩 1. Architecture Overview

### Diagram
GitHub → CodePipeline → CodeBuild → S3 (Static Website Hosting)


**Workflow Summary:**
1. Source code (HTML/CSS/PDF) is stored in a GitHub repository.  
2. CodePipeline automatically detects any commit or push to the `main` branch.  
3. CodePipeline triggers a build job in CodeBuild.  
4. CodeBuild executes the commands defined in `buildspec.yml` to synchronize files to the S3 bucket.  
5. The S3 bucket hosts the updated website as a static site.

### AWS Services Used
| Service | Purpose |
|----------|----------|
| GitHub | Version control and source trigger |
| AWS CodePipeline | CI/CD orchestration (detects changes and triggers builds) |
| AWS CodeBuild | Executes the buildspec file and deploys to S3 |
| Amazon S3 | Hosts the static website |
| IAM | Manages permissions for CodeBuild and CodePipeline roles |

---

## ⚙️ 2. Setup Steps

### Step 1 – Prepare the Website Repository
Files included:
- index.html
- style.css
- buildspec.yml
- assets/Resume_Misuzu Taniguchi.pdf

Push all files to GitHub (public or private).

---

### Step 2 – Create an S3 Bucket
1. Create a bucket (e.g., `<your-s3-bucket>`).  
2. Enable **Static Website Hosting** → Index document: `index.html`  
3. Disable *Block All Public Access* and attach this policy:
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Sid": "PublicReadGetObject",
         "Effect": "Allow",
         "Principal": "*",
         "Action": "s3:GetObject",
         "Resource": "arn:aws:s3:::<your-s3-bucket>/*"
       }
     ]
   }


### Step 3 – Create the CodeBuild Project

1.	Project name: resume-site-build
2.	Source: GitHub
3.	Environment:
	- OS: Amazon Linux 2
    - Runtime: Standard
    - Image: aws/codebuild/amazonlinux2-x86_64-standard:5.0
4.	Buildspec: use buildspec.yml
5.	Service role: create new → attach minimal S3 permissions:
    ```json
    {
    "Effect": "Allow",
    "Action": ["s3:ListBucket"],
    "Resource": "arn:aws:s3:::<your-s3-bucket>"
    },
    {
    "Effect": "Allow",
    "Action": ["s3:PutObject","s3:DeleteObject","s3:GetObject"],
    "Resource": "arn:aws:s3:::<your-s3-bucket>/*"
    }
    ```


### Step 4 – Create CodePipeline
1.	Pipeline name: resume-site-pipeline
2.	Source provider: GitHub (Version 2)
    - Repo: misuzu-taniguchi/COMP-4964-Assignment4-Resume-website
    - Branch: main
3.	Build provider: AWS CodeBuild (resume-site-build)
4.	Deploy stage: Skipped (buildspec handles S3 sync)
5.	Review and create pipeline.


### Step5 - Test the Pipeline

1. Edit index.html
2. Commit and push
3. CodePipeline runs automatically → triggers CodeBuild → updates S3.
4.  Refresh the static website URL to confirm update.


## 🩺 3. Troubleshooting Guide

| Issue | Cause | Solution |
|-------|--------|-----------|
| **AccessDenied** | Missing S3 permissions | Attach correct S3 bucket policy and update CodeBuild role permissions |
| **Cannot save bucket policy** | Block Public Access still enabled | Disable “Block Public Access” at bucket or account level |
| **403 Error on website** | Static hosting disabled or policy missing | Enable static website hosting and verify bucket policy |
| **Changes not detected** | GitHub webhook not connected | Reconnect GitHub (Version 2) in CodePipeline |
| **Sync fails** | Syntax error in `buildspec.yml` (line breaks or backslashes) | Use a single-line `aws s3 sync` command |

---

