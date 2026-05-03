AWS Secure S3 Lab Steps

## Lab Objective
Demonstrate how to architect a secure S3 environment from scratch using IAM least-privilege policies, encryption at rest, access logging, and CloudTrail without relying on default AWS settings which are insufficient for production workloads.

**Time to complete:** ~45 minutes  
**AWS Free Tier:** Yes — all services used fall within free tier limits  
**Difficulty:** Beginner–Intermediate

1. Created zero-spend AWS budget
2. Created private S3 bucket
3. Enabled SSE-S3 encryption
4. Created logging bucket
5. Enabled server access logging
6. Created least-privilege IAM policy
7. Created IAM user
8. Uploaded test object

## What I Learned / Challenges

- Block Public Access has 4 separate settings — enabling only some still leaves exposure vectors
- IAM Explicit Deny always wins over Allow, even if another policy grants access — useful for protecting critical actions
- S3 access logs and CloudTrail serve different purposes: access logs capture object-level HTTP requests, CloudTrail captures AWS API management calls. Both are needed for a complete audit trail
- SSE-S3 vs SSE-KMS: SSE-KMS provides more control (key rotation, access auditing via CloudTrail) but adds cost — the right choice depends on compliance requirements
