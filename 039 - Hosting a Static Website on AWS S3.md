# Hosting a Static Website on AWS S3

---

<aside>

**Purpose:** Host a static website on Amazon S3 using S3 static website hosting, then verify the site is publicly accessible.

</aside>

---

### Procedure

1. Create an S3 bucket:
    1. Go to S3 and choose Create bucket.
    2. Enter the bucket name (e.g., `nautilus-web-12345`) and choose a region.
    3. Under Block Public Access settings for this bucket, uncheck Block all public access.
    4. Choose Create.
2. Enable static website hosting:
    1. Open the S3 console: [https://console.aws.amazon.com/s3](https://console.aws.amazon.com/s3)
    2. In the left navigation pane, choose General purpose buckets.
    3. Select your bucket.
    4. Choose Properties.
    5. Under Static website hosting, choose Edit.
    6. Under Static website hosting, choose Enable.
    7. In Index document, enter `index.html`.
    8. Choose Save changes.
3. Set a bucket policy (public read for website objects):
    1. Go to the Permissions tab for the bucket.
    2. Edit Bucket policy and use:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::nautilus-web-12345/*"
    }
  ]
}
```

1. Upload `index.html` to the bucket:
    1. Upload the file from your client host:

```bash
aws s3 cp index.html s3://nautilus-web-12345/
```

1. Verify the object is in the bucket.
2. Access the application:
    1. In the bucket, open Properties.
    2. In the Static website hosting section, open the Bucket website endpoint.
    3. Confirm the page loads (e.g., `Welcome to KKE labs!`).

### Quick check

- **Expected result:** The S3 website endpoint loads your `index.html` successfully in a browser.