# Lab Walkthrough — Secure S3 Architecture

## Overview
**Objective:** Build and harden an S3 environment from scratch following AWS security best practices and HIPAA-aligned controls.
**Time to complete:** ~45 minutes
**AWS Free Tier:** Yes — all services used fall within free tier limits
**Difficulty:** Beginner–Intermediate

---

## Prerequisites
- An AWS account (free tier is sufficient)
- IAM user with AdministratorAccess for lab setup
- Basic familiarity with the AWS console

---

## Step 1 — Create the Primary S3 Bucket

1. Navigate to **S3** in the AWS Console
2. Click **Create bucket**
3. Set the bucket name (e.g. `your-primary-bucket-name`)
4. Select your region
5. Under **Block Public Access settings**, enable all four options:
   - Block all public access ✅
   - Block public ACLs ✅
   - Block public bucket policies ✅
   - Ignore public ACLs ✅
6. Under **Default encryption**, select **SSE-S3 (AES-256)**
7. Click **Create bucket**

**Expected result:** Bucket created with a yellow lock icon confirming public access is blocked.

---

## Step 2 — Create the Logging Bucket

A separate bucket is required to store access logs — storing logs in the same bucket creates a log loop and risks tampering.

1. Repeat Step 1 to create a second bucket (e.g. `your-logging-bucket-name`)
2. Apply the same Block Public Access and encryption settings
3. Do **not** enable logging on this bucket — it is the log destination, not the source

---

## Step 3 — Enable S3 Access Logging

1. Open your **primary bucket**
2. Go to **Properties** tab
3. Scroll to **Server access logging** → click **Edit**
4. Enable logging
5. Set the target bucket to your **logging bucket**
6. Set a log prefix (e.g. `s3-access-logs/`)
7. Save changes

**Expected result:** Any HTTP request to the primary bucket is now logged with requester IP, action, timestamp, and response code.

---

## Step 4 — Apply the Bucket Policy (HTTPS-Only + Encryption Enforcement)

1. Open your **primary bucket**
2. Go to **Permissions** tab
3. Scroll to **Bucket policy** → click **Edit**
4. Paste the contents of `bucket-policy.json` from this repo
5. Replace `your-primary-bucket-name` with your actual bucket name
6. Click **Save changes**

**What this enforces:**
- All HTTP (non-HTTPS) requests are denied
- No object or bucket ACLs can be set to public
- Any uploaded object must use AES-256 encryption or the upload is rejected

---

## Step 5 — Create the IAM Policy

1. Navigate to **IAM** → **Policies** → **Create policy**
2. Select the **JSON** editor tab
3. Paste the contents of `iam-policy.json` from this repo
4. Replace `your-primary-bucket-name` with your actual bucket name
5. Name the policy (e.g. `S3-SecureLab-LeastPrivilege`)
6. Click **Create policy**

**What this grants:**
- `s3:GetObject` and `s3:PutObject` on objects only
- `s3:ListBucket` on the bucket itself
- Explicit Deny on `s3:DeleteObject` and `s3:DeleteBucket` as defense in depth

---

## Step 6 — Create an IAM User and Attach the Policy

1. Navigate to **IAM** → **Users** → **Create user**
2. Name the user (e.g. `s3-lab-user`)
3. Select **Attach policies directly**
4. Search for and attach `S3-SecureLab-LeastPrivilege`
5. Complete user creation

**Expected result:** The user can read and write objects but cannot delete them or access any other AWS service.

---

## Step 7 — Test the Configuration

1. In the primary bucket, click **Upload** and upload any test file
2. Confirm the upload succeeds ✅
3. Click the file → **Object actions** → **Delete**
4. Confirm deletion is denied (if testing with the IAM user's credentials) ✅
5. Try accessing the bucket URL over HTTP — confirm it is blocked ✅

---

## What I Learned / Challenges

- Block Public Access has 4 separate settings — enabling only some still leaves exposure vectors
- IAM Explicit Deny always wins over Allow — even if another policy grants access, Deny takes precedence
- S3 access logs and CloudTrail serve different purposes: access logs capture object-level HTTP requests, CloudTrail captures AWS API management calls — both are needed for a complete audit trail
- SSE-S3 vs SSE-KMS: SSE-KMS provides more control (key rotation, access auditing) but adds cost — the right choice depends on compliance requirements
- Bucket policies and IAM policies work together: bucket policies control resource-level access, IAM policies control identity-level access — both must be configured correctly

---

## CloudTrail Note

CloudTrail's default trail automatically captures S3 management API calls (bucket creation, policy changes, IAM modifications) at the account level. A dedicated trail with S3 data-event logging (GetObject, PutObject, DeleteObject at the object level) was not configured in this lab iteration. In a production or compliance environment (HIPAA, SOC 2, PCI-DSS), a dedicated data-event trail would be required for a complete audit trail and is a planned addition to this lab.
