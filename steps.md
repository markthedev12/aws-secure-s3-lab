# Lab Walkthrough — Secure S3 Architecture

## Overview
**Objective:** Build and harden an S3 environment from scratch following AWS security best practices and HIPAA-aligned controls.
**Time to complete:** ~45 minutes
**Costs:** S3 storage/requests, log delivery and CloudTrail configuration may incur charges. Check current pricing and account eligibility.
**Difficulty:** Beginner–Intermediate

---

## Prerequisites
- An AWS account (free tier is sufficient)
- A dedicated setup role scoped to the S3 and logging actions required by this lab; do not use root credentials
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

**Expected result:** Server access logs arrive in the destination bucket on a best-effort basis. Delivery is delayed and is not a complete or immediate request ledger.

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
- All PutBucketAcl and PutObjectAcl requests are denied, including private ACL updates
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

**Expected result:** This policy permits listing the target bucket and reading/writing its objects, and denies deletion. Other attached policies may still grant access to other services; inspect effective permissions rather than assuming this policy removes those grants.

---

## Step 7 — Test the Configuration

1. Using the scoped test identity, upload a fictional file with an explicit AES256 encryption header: `aws s3 cp demo.txt s3://YOUR_BUCKET/demo.txt --sse AES256`
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

CloudTrail Event history and configured trails have different retention and delivery behavior; inspect the actual trail and event selectors rather than assuming a trail exists by default. S3 object data events (GetObject, PutObject, DeleteObject) were not configured in this lab. Define event coverage and retention from the workload's audit requirements before relying on the logs.
