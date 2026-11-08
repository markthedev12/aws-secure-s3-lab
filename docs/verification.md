# Verify the example S3 policies

This repository documents a manual lab. Policies use placeholder bucket names;
replace all ARNs in a local copy before applying them to a bucket you control.
Do not use production data. See `steps.md` for setup and logging configuration.

## Understand the exact semantics

- The IAM policy permits GetObject/PutObject and ListBucket. It explicitly denies DeleteObject/DeleteBucket; this is a deliberate lab restriction, not a complete enterprise policy.
- The bucket policy denies non-TLS requests, every object/bucket ACL update, and uploads whose encryption request header is not AES256.
- Consequently a normal upload with no explicit encryption header can be denied even when the bucket has default encryption. Use `aws s3 cp demo.txt s3://YOUR_BUCKET/demo.txt --sse AES256` for an intended permitted upload.
- The header rule also rejects SSE-KMS uploads. Changing encryption strategy requires reviewing the policy, not merely the bucket default.
- CloudTrail management events do not include object data events unless separately configured. Server access logs are best-effort and delayed.

## Manual acceptance checks

Use an appropriately scoped test identity. Confirm listing, explicit AES256 upload,
and download succeed; confirm deletion and uploads missing the required header
are denied. Review Block Public Access in the bucket configuration, the separate
logging destination permissions, and CloudTrail's enabled event selectors.
Do not disable Block Public Access to test this lab.

Validate JSON syntax locally:

```sh
python -m json.tool iam-policy.json
python -m json.tool bucket-policy.json
```

JSON parsing proves syntax only, not policy correctness or effective access.
Use IAM Access Analyzer policy validation and controlled AWS tests before relying
on a modified policy. Record the identity, expected outcome, actual result, and
timestamp using sanitized evidence.

## Cleanup

Review the example explicit deletion deny before cleanup: use the authorized
setup identity and adjust/remove the lab policy when necessary. Empty test
objects, versions and delete markers as applicable; remove test buckets, logging
configuration and any lab-only trail after reviewing dependencies. Confirm
remaining storage/logging charges. Never delete shared infrastructure as cleanup.
