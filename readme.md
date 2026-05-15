# React Frontend CI/CD Pipeline using GitHub Actions, AWS S3, CloudFront & OIDC

This project demonstrates a production-level CI/CD pipeline for a React frontend application using:

- React (Vite)
- GitHub Actions
- AWS S3
- AWS CloudFront
- OpenID Connect (OIDC)
- GitHub Flow Strategy

The application is automatically tested and deployed to AWS after every successful merge into the `main` branch.

---

# Project Architecture

```text
Developer
   ↓
Feature Branch
   ↓
Pull Request
   ↓
GitHub Actions CI Pipeline
   ↓
Code Review + Approval
   ↓
Merge into main
   ↓
GitHub Actions CD Pipeline
   ↓
OIDC Authentication
   ↓
Deploy React Build to S3
   ↓
CloudFront Cache Invalidation
   ↓
Production Deployment
```

---

# GitHub Flow Strategy

This project follows the GitHub Flow workflow.

## Workflow Steps

1. Create feature branch from `main`
2. Develop feature locally
3. Push branch to GitHub
4. Open Pull Request
5. CI pipeline runs automatically
6. Review and approve PR
7. Merge PR into `main`
8. CD pipeline deploys application automatically

Example:

```bash
git checkout -b feature/navbar
```

---

# Technologies Used

| Technology | Purpose |
|---|---|
| React + Vite | Frontend Application |
| GitHub Actions | CI/CD Automation |
| AWS S3 | Static Website Hosting |
| AWS CloudFront | CDN |
| AWS IAM | Secure Access Management |
| OIDC | Secure AWS Authentication |

---

# Step 1 — Create React Application

Create React app using Vite:

```bash
npm create vite@latest .
```

Select:

```text
Framework: React
Variant: JavaScript
```

Install dependencies:

```bash
npm install
```

Run locally:

```bash
npm run dev
```

Application runs at:

```text
http://localhost:5173
```

Build application:

```bash
npm run build
```

Production build files are generated inside:

```text
dist/
```

---

# Step 2 — Initialize Git Repository

```bash
git init
git add .
git commit -m "Initial React frontend setup"
```

Create GitHub repository and push code:

```bash
git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPOSITORY.git

git branch -M main

git push -u origin main
```

---

# Step 3 — Create AWS S3 Bucket

Create an S3 bucket for frontend hosting.

Example:

```text
my-react-frontend-prod
```

Enable:

- Static website hosting
- Public access (if needed)
- CloudFront integration

---

# Step 4 — Create CloudFront Distribution

Create a CloudFront distribution using the S3 bucket as origin.

Benefits:

- Global CDN
- HTTPS support
- Faster performance
- Better caching

---

# Step 5 — Create OIDC Provider in AWS

Go to:

```text
AWS Console
→ IAM
→ Identity Providers
→ Add Provider
```

Configure:

| Setting | Value |
|---|---|
| Provider Type | OpenID Connect |
| Provider URL | https://token.actions.githubusercontent.com |
| Audience | sts.amazonaws.com |

OIDC allows GitHub Actions to authenticate securely with AWS without storing AWS access keys.

---

# Step 6 — Create IAM Role for GitHub Actions

Create an IAM role for GitHub Actions deployment.

Example role name:

```text
frontend-deployment-role
```

---

# Step 7 — Configure IAM Trust Policy

Attach the following trust relationship to the IAM role:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::<ACCOUNT_ID>:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
        },
        "StringLike": {
          "token.actions.githubusercontent.com:sub": "repo:YOUR_GITHUB_USERNAME/YOUR_REPOSITORY:ref:refs/heads/main"
        }
      }
    }
  ]
}
```

This ensures that only GitHub Actions from the `main` branch of this repository can assume the AWS role.

---

# Step 8 — Attach IAM Permissions

Attach permissions required for deployment.

Example policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "S3Deployment",
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:DeleteObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::YOUR_BUCKET_NAME",
        "arn:aws:s3:::YOUR_BUCKET_NAME/*"
      ]
    },
    {
      "Sid": "CloudFrontInvalidation",
      "Effect": "Allow",
      "Action": [
        "cloudfront:CreateInvalidation"
      ],
      "Resource": "*"
    }
  ]
}
```

---

# Step 9 — Configure GitHub Repository Secrets

Go to:

```text
GitHub Repository
→ Settings
→ Secrets and variables
→ Actions
```

Create the following secrets:

| Secret Name | Description |
|---|---|
| AWS_ROLE_ARN | IAM Role ARN |
| AWS_REGION | AWS deployment region |
| S3_BUCKET_NAME | S3 bucket name |
| CLOUDFRONT_DISTRIBUTION_ID | CloudFront distribution ID |

Example:

```text
AWS_ROLE_ARN=arn:aws:iam::123456789012:role/frontend-deployment-role
```

---

# Step 10 — Create CI Workflow

Create file:

```text
.github/workflows/ci.yml
```

The CI pipeline runs on Pull Requests.

## CI Responsibilities

- Install dependencies
- Run lint checks
- Run tests
- Validate production build

# Step 11 — Create CD Workflow

Create file:

```text
.github/workflows/cd.yml
```

The CD pipeline runs automatically after merging into the `main` branch.

## CD Responsibilities

- Build React application
- Authenticate with AWS using OIDC
- Upload frontend files to S3
- Invalidate CloudFront cache

# Why Use OIDC Instead of AWS Access Keys?

OIDC is more secure because:

- No long-term AWS secrets
- No manual credential rotation
- Temporary credentials are generated dynamically
- Better security for production systems

---

# Why Use `npm ci` Instead of `npm install`?

`npm ci` is preferred in CI/CD because:

- Faster installation
- Uses exact versions from package-lock.json
- More reliable and deterministic builds

---

# Why Use CloudFront Cache Invalidation?

CloudFront caches frontend assets.

After deployment, cache invalidation ensures users receive the latest frontend version.

Command used:

```bash
aws cloudfront create-invalidation \
  --distribution-id DISTRIBUTION_ID \
  --paths "/*"
```

---

# Recommended Branch Protection Rules

Protect the `main` branch with:

- Require Pull Request before merge
- Require CI checks to pass
- Require approvals
- Prevent force pushes
- Prevent branch deletion

---

# Final Result

After successful merge into `main`:

- CI validates the frontend
- CD deploys application automatically
- React build files are uploaded to S3
- CloudFront serves the frontend globally
- Users receive the latest production version automatically

---