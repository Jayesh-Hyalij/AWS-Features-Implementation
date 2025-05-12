  # AWS S3 Features Implementation Documentation

This document provides detailed steps to create and manage an Amazon S3 bucket with various features using both the AWS Management Console and AWS CLI. It covers bucket creation, file uploads, permission settings, versioning, lifecycle rules, static website hosting, cross-region replication, access logging, and monitoring.

---

## 1. Create an S3 Bucket

### Using AWS Console

1. Sign in to the [AWS Management Console](https://aws.amazon.com/console/).
2. Navigate to **Services** > **S3**.
3. Click **Create bucket**.
4. Enter a unique **Bucket name** (e.g., `s3-features-implementation`).
5. Select the **AWS Region** where you want the bucket.

> **Note:** Bucket names must be globally unique across all AWS users and follow DNS naming conventions.

6. Configure options as needed (e.g., versioning, tags).
7. Click **Create bucket**.

*Screenshot: AWS Console Create Bucket page*
![Screenshot](Assets/AWS%20Console%20Create%20Bucket%20page.png)

---

### Using AWS CLI

Run the following command to create a bucket (replace `s3-features-implementation` and `us-east-1` with your values):

```bash
aws s3api create-bucket --bucket s3-features-implementation --region us-east-1 --create-bucket-configuration LocationConstraint=us-east-1
```

*Note:* For `us-east-1` region, omit the `--create-bucket-configuration` parameter.

> **Important:** Ensure your AWS CLI is configured with appropriate credentials and region before running commands.

---

## 2. Upload Files to the Bucket

### Using AWS Console

1. Open your bucket in the S3 console.
2. Click **Upload**.
3. Add files or folders (e.g., `index.html`).
4. Click **Upload**.

*Screenshot: Upload files in AWS Console*
![Screenshot](Assets/Upload%20files%20in%20AWS%20Console.png)

---

### Using AWS CLI

Upload a file to the bucket:

```bash
aws s3 cp index.html s3://my-unique-bucket-name/
```

Upload a folder recursively:

```bash
aws s3 cp ./my-folder s3://my-unique-bucket-name/ --recursive
```

---

## 3. Set Bucket Permissions

### Public Read Policy

To allow public read access to all objects in the bucket, apply the following bucket policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-unique-bucket-name/*"
    }
  ]
}
```

> **Warning:** Making your bucket publicly readable exposes all objects to anyone on the internet. Use this only for public content like static websites.

#### Applying via AWS Console

1. Go to your bucket.
2. Click **Permissions** tab.
3. Click **Bucket Policy**.
4. Paste the above JSON, replacing `my-unique-bucket-name`.
5. Save changes.

*Screenshot: Bucket Policy editor in AWS Console*

---

### IAM Policy for User Access

Example IAM policy to allow a user to list and upload objects to the bucket:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket"
            ],
            "Resource": "arn:aws:s3:::my-unique-bucket-name"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:GetObject",
                "s3:DeleteObject"
            ],
            "Resource": "arn:aws:s3:::my-unique-bucket-name/*"
        }
    ]
}
```

> **Tip:** Use IAM policies to grant fine-grained access control to users and applications interacting with your bucket.

Attach this policy to the IAM user or group as needed.

---
}

## 4. Enable Versioning and Add Lifecycle Rule

### Enable Versioning

#### AWS Console

1. Open your bucket.
2. Go to **Properties** tab.
3. Under **Bucket Versioning**, click **Edit**.
4. Select **Enable**.
5. Save changes.

*Screenshot: Enable versioning in AWS Console*

#### AWS CLI

```bash
aws s3api put-bucket-versioning --bucket my-unique-bucket-name --versioning-configuration Status=Enabled
```

---

### Add Lifecycle Rule

Example lifecycle rule to transition objects to Glacier after 30 days and delete after 365 days:

```json
{
  "Rules": [
    {
      "ID": "TransitionToGlacierAndExpire",
      "Status": "Enabled",
      "Filter": {},
      "Transitions": [
        {
          "Days": 30,
          "StorageClass": "GLACIER"
        }
      ],
      "Expiration": {
        "Days": 365
      }
    }
  ]
}
```

> **Note:** Adjust lifecycle rules based on your data retention and cost optimization requirements.

#### Apply via AWS CLI

Save the above JSON to a file `lifecycle.json` and run:

```bash
aws s3api put-bucket-lifecycle-configuration --bucket my-unique-bucket-name --lifecycle-configuration file://lifecycle.json
```

---
aws s3api put-bucket-lifecycle-configuration --bucket my-unique-bucket-name --lifecycle-configuration file://lifecycle.json
}

## 5. Host a Static Website

### Upload `index.html`

Upload your `index.html` file to the bucket (see upload steps above).

### Enable Website Hosting

#### AWS Console

1. Open your bucket.
2. Go to **Properties** tab.
3. Scroll to **Static website hosting**.
4. Click **Edit**.
5. Select **Enable**.
6. Enter `index.html` as the **Index document**.
7. Save changes.

*Screenshot: Static website hosting settings*

> **Important:** Static website hosting does not support HTTPS. Use CloudFront or other CDN for secure HTTPS hosting.

#### AWS CLI

```bash
aws s3 website s3://my-unique-bucket-name/ --index-document index.html
```

---

## 6. Configure Cross-Region Replication (CRR)

### Prerequisites

- Both source and destination buckets must have versioning enabled.
- The destination bucket must have a bucket policy allowing replication.

> **Note:** Cross-Region Replication may incur additional costs and requires proper IAM roles and permissions.

### Steps

1. Enable versioning on both buckets (see step 4).
2. Create an IAM role for replication with appropriate permissions.
3. Configure replication rule on the source bucket.

Example replication configuration JSON (`replication.json`):

```json
{
  "Role": "arn:aws:iam::123456789012:role/replication-role",
  "Rules": [
    {
      "ID": "ReplicationRule1",
      "Status": "Enabled",
      "Prefix": "",
      "Destination": {
        "Bucket": "arn:aws:s3:::destination-bucket-name"
      }
    }
  ]
}
```

Apply with CLI:

```bash
aws s3api put-bucket-replication --bucket source-bucket-name --replication-configuration file://replication.json
```

*Note:* Replace ARNs and bucket names accordingly.

---
aws s3api put-bucket-replication --bucket source-bucket-name --replication-configuration file://replication.json
}

## 7. Enable Access Logging

### AWS Console

1. Open your source bucket.
2. Go to **Properties** tab.
3. Scroll to **Server access logging**.
4. Click **Edit**.
5. Enable logging and specify a target bucket and optional prefix.
6. Save changes.

*Screenshot: Server access logging settings*

> **Tip:** Use a separate bucket for access logs to avoid recursive logging.

### AWS CLI

```bash
aws s3api put-bucket-logging --bucket source-bucket-name --bucket-logging-status '{
  "LoggingEnabled": {
    "TargetBucket": "log-target-bucket",
    "TargetPrefix": "source-bucket-logs/"
  }
}'
```

---

## 8. Monitor Activity via CloudTrail

1. Open the AWS Management Console.
2. Navigate to **CloudTrail**.
3. Create a new trail or use an existing one.
4. Ensure S3 data events are enabled for your bucket.
5. View logs and events related to S3 bucket activity.

*Screenshot: CloudTrail event history*

---

## 9. View Metrics in CloudWatch

1. Open the AWS Management Console.
2. Navigate to **CloudWatch** > **Metrics**.
3. Select **S3** namespace.
4. View metrics such as NumberOfObjects, BucketSizeBytes, AllRequests, etc.

*Screenshot: CloudWatch S3 metrics*

> **Note:** Some S3 metrics are updated once per day and may have a delay.

---

# Summary

This documentation covered creating an S3 bucket, uploading files, setting permissions, enabling versioning and lifecycle rules, hosting a static website, configuring cross-region replication, enabling access logging, and monitoring via CloudTrail and CloudWatch. Use the AWS Console or CLI commands as per your preference.

---

*Note:* Replace all placeholder bucket names, ARNs, and file names with your actual values. Add screenshots at indicated places for better clarity.
