# AWS Secure S3 Lab

Hardened S3 architecture with least privilege IAM, encryption at rest, an HTTPS only bucket policy, access logging and CloudTrail, with the reasoning behind each control.

> Personal lab project, built by hand in the console and CLI. A Terraform version is planned.

## Why this matters

S3 misconfiguration is one of the most common causes of cloud data exposure. This lab implements the controls that prevent it and documents why each one is there.

## Architecture

```
IAM policy (least privilege)
  s3:GetObject and s3:PutObject only
  scoped to one bucket ARN
  explicit deny on s3:DeleteObject
        |
        v
Primary S3 bucket
  SSE-S3 encryption
  Block Public Access (all four settings)
  Versioning
  HTTPS only bucket policy
        |
        | access logs
        v
Logging S3 bucket
  separate bucket for access logs

CloudTrail (account level)
  management API calls
```

## Controls

| Control | What it does | Why |
| --- | --- | --- |
| Least privilege IAM policy | Grants Get and Put on one bucket, denies Delete, no bucket listing | Wildcard S3 permissions are a top misconfiguration. An explicit deny beats any allow. |
| SSE-S3 encryption | Encrypts objects at rest by default | No key management cost. A regulated workload would use SSE-KMS with a customer managed key. |
| Block Public Access | All four settings on | Overrides object ACLs and stops accidental public exposure. |
| Access logging | Sends request logs to a separate bucket | Audit trail of who accessed what, kept apart from the source bucket. |
| HTTPS only policy | Denies requests made over HTTP | Protects data in transit. |
| CloudTrail | Records management API calls | Shows policy and configuration changes. |

## Files

| File | Description |
| --- | --- |
| `iam-policy.json` | Least privilege IAM policy |
| `bucket-policy.json` | HTTPS only and restrictive bucket policy |
| `steps.md` | Full walkthrough with CLI commands |
| `screenshot/` | Console screenshots of each configuration |

## Limitations

* CloudTrail here covers management events only. Object level data events (GetObject, PutObject, DeleteObject) are not configured.
* SSE-S3 is used, not SSE-KMS.
* CloudTrail log file validation is not covered, so the trail is not tamper evident.
* Built manually, so it is not repeatable as code yet.

## What I would do next

- [ ] Rebuild the whole lab in Terraform
- [ ] Add Checkov scanning in GitHub Actions
- [ ] Switch to SSE-KMS with a customer managed key and rotation
- [ ] Add data event logging and log file validation

## Key takeaways

* A private ACL without Block Public Access can still be made public. Turn on all four settings.
* SSE-S3 versus SSE-KMS is a cost and compliance tradeoff, not a default.
* Access logs and CloudTrail answer different questions. You want both.
* An explicit Deny always overrides an Allow in IAM.

## Author

Mark Schwinn · [Website](https://markschwinn.com) · [LinkedIn](https://www.linkedin.com/in/mark-schwinn-994625362/)
