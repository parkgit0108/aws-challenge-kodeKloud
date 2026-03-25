# Managing EC2 Access with S3 Role-based Permissions

---

<aside>

**Purpose:** Enable an EC2 instance to access a private S3 bucket using an IAM policy + IAM role, then verify upload/download from the instance.

</aside>

---

### Procedure

1. Verify existing EC2 instance:
    1. Confirm an instance named `devops-ec2` already exists.
    2. No changes needed yet (it will be attached to an IAM role later).
2. Set up SSH keys (password-less access):
    1. Create SSH key pair on `aws-client`:

```bash
# Generate a new SSH key pair:
ssh-keygen -t rsa -b 4096 -C ""

# Confirm files exist:
ls /root/.ssh/
```

You should see: `id_rsa` and `id_rsa.pub`

1. Add public key to the EC2 instance:
    1. Connect to `devops-ec2` using AWS Console (Instance Connect or Session Manager).
    2. Ensure the EC2 security group allows SSH (port 22) from your source.
    3. Edit the authorized keys file:

```bash
sudo -i
nano /root/.ssh/authorized_keys
```

1. Paste the contents of `/root/.ssh/id_rsa.pub`, then save and exit.
2. Create a private S3 bucket:
    1. Open AWS Console → S3.
    2. Click Create bucket.
    3. Set:
        - Bucket name: `devops-s3-12345`
        - Region: same as the EC2 instance
        - Object Ownership: ACLs disabled
        - Block Public Access: keep all enabled (private)
    4. Click Create bucket.
3. Create an IAM policy for S3 access:
    1. Go to IAM → Policies → Create policy.
    2. Choose the JSON tab and use:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::devops-s3-12345/*"
    },
    {
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::devops-s3-12345"
    }
  ]
}
```

1. Set policy name: `devops-s3-policy`.
2. Create the policy.
3. Create an IAM role and attach the policy:
    1. Go to IAM → Roles → Create role.
    2. Trusted entity: AWS service.
    3. Use case: EC2.
    4. Attach policy: `devops-s3-policy`.
    5. Role name: `devops-role`.
    6. Create the role.
4. Attach the IAM role to the EC2 instance:
    1. Go to EC2 → Instances.
    2. Select `devops-ec2`.
    3. Actions → Security → Modify IAM role.
    4. Choose `devops-role` → Update IAM role.
5. Test S3 access from the EC2 instance:
    1. SSH into the instance from `aws-client`:

```bash
ssh root@<devops-ec2-public-ip>
```

1. Create a test file:

```bash
echo "S3 test" > test.txt
```

1. Upload the file to S3:

```bash
aws s3 cp testfile.txt s3://devops-s3-12345/
```

1. List files in the bucket:

```bash
aws s3 ls s3://devops-s3-12345/
```

### Quick check

- **Expected result:** `aws s3 ls s3://devops-s3-12345/` shows `test.txt` and the upload/download commands succeed.