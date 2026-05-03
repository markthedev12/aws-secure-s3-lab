# AWS Secure S3 Lab

**Skills demonstrated:** IAM least-privilege, S3 encryption, access logging, CloudTrail, bucket policy hardening

---

## Overview

This lab demonstrates how to architect and secure an Amazon S3 environment following AWS security best practices. The focus is on applying the principle of least privilege through IAM, enabling encryption at rest, and establishing a full audit trail via access logging and CloudTrail.

This project reflects real-world cloud security requirements found in healthcare, financial services, and regulated industries.

---

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                    IAM Layer                        │
│  ┌──────────────────────────────────────────────┐   │
│  │  IAM Policy (Least Privilege)                │   │
│  │  • s3:GetObject, s3:PutObject only           │   │
│  │  • Scoped to specific bucket ARN             │   │
│  │  • Denies s3:DeleteObject explicitly         │   │
│  └──────────────────┬───────────────────────────┘   │
└─────────────────────┼───────────────────────────────┘
                      │
          ┌───────────▼────────────┐
          │   Primary S3 Bucket    │
          │  • SSE-S3 Encryption   │
          │  • Block Public Access │
          │  • Versioning Enabled  │
          │  • HTTPS-only Policy   │
          └───────────┬────────────┘
                      │ Access Logs
          ┌───────────▼────────────┐
          │   Logging S3 Bucket    │
          │  • Stores access logs  │
          │  • Separate IAM policy │
          └────────────────────────┘
                      │
          ┌───────────▼────────────┐
          │      CloudTrail        │
          │  • API call logging    │
          │  • Immutable log trail │
          └────────────────────────┘
```

---

## Security Controls Implemented

### 1. IAM Least-Privilege Policy
**Why:** Granting `s3:*` to any user or role violates the principle of least privilege and is a top misconfiguration in cloud breaches. This policy grants only the minimum permissions required.

**Decisions made:**
- Explicitly scoped to a single bucket ARN (not `*`)
- `s3:DeleteObject` denied explicitly even though it isn't granted — defense in depth
- No `s3:ListAllMyBuckets` — the user cannot enumerate other buckets in the account

### 2. Server-Side Encryption (SSE-S3)
**Why:** Encrypts all objects at rest by default. SSE-S3 was chosen over SSE-KMS for this lab because it requires no additional KMS key management cost, making it appropriate for non-regulated workloads. In a HIPAA or PCI-DSS environment, SSE-KMS with a customer-managed key (CMK) and key rotation would be required.

### 3. Block Public Access (All Four Settings)
**Why:** Even with a private bucket policy, a misconfigured ACL can inadvertently expose objects. Enabling all four Block Public Access settings provides a hard override that prevents any public exposure regardless of object-level ACL settings.

### 4. S3 Access Logging
**Why:** Logs every request made to the bucket (requester, IP, action, timestamp). Stored in a dedicated logging bucket to prevent log tampering. This satisfies audit trail requirements in ISO 27001, SOC 2, and HIPAA.

### 5. HTTPS-Only Bucket Policy
**Why:** Denies any request made over HTTP (unencrypted). This prevents man-in-the-middle attacks on data in transit — required by HIPAA Security Rule §164.312(e)(1).

### 6. CloudTrail Integration

> **Note:** CloudTrail's default trail automatically captures S3 management 
> API calls (bucket creation, policy changes, IAM modifications) at the account 
> level. A dedicated trail with S3 data-event logging (GetObject, PutObject, 
> DeleteObject at the object level) was not configured in this lab iteration. 
> In a production or compliance environment (HIPAA, SOC 2, PCI-DSS), a dedicated 
> data-event trail would be required for a complete audit trail and is a planned 
> addition to this lab.

---

## Files in This Repo

| File | Description |
|------|-------------|
| `README.md` | Project overview and security rationale |
| `iam-policy.json` | Least-privilege IAM policy with inline comments |
| `steps.md` | Full lab walkthrough with CLI commands |
| `screenshot/` | Console screenshots proving each configuration |

---

## Key Takeaways

- A bucket with private ACL but no Block Public Access enabled can still be made public — always enable all four BPA settings
- SSE-S3 vs SSE-KMS is an architectural decision driven by compliance requirements and cost
- Access logging and CloudTrail serve different purposes and both are needed for full auditability
- Explicit Deny in IAM always overrides an Allow — use it for critical actions like Delete

---

## Author

Mark Schwinn | [LinkedIn](https://www.linkedin.com/in/mark-schwinn-994625362/) | CompTIA Security+ | AWS SAA (In Progress)
