# Centralized Audit Logging with VPC Peering

---

<aside>

**Purpose:** Build a simple centralized audit-log flow using VPC peering: generate a log on a private EC2 instance, copy it to a public EC2 (bastion) via SCP, then upload it to an S3 bucket.

</aside>

---

### Procedure

1. Create the public VPC (Console → VPC)
    1. Create VPC:
        - Name: `nautlius-pub-vpc`
        - IPv4 CIDR: `10.20.0.0/16`
    2. Create subnet:
        - VPC: `nautlius-pub-vpc`
        - Name: `nautlius-pub-subnet`
        - CIDR: `10.20.1.0/24`
        - Enable Auto-assign public IPv4
    3. Create and attach Internet Gateway:
        - Name: `nautlius-pub-igw`
        - Attach to `nautlius-pub-vpc`
    4. Create route table and associate subnet:
        - Name: `nautlius-pub-rt`
        - VPC: `nautlius-pub-vpc`
        - Route: `0.0.0.0/0` → `nautlius-pub-igw`
        - Associate with `nautlius-pub-subnet`
2. Launch the public EC2 (bastion) (Console → EC2)
    1. Launch instance:
        - Name: `nautlius-pub-ec2`
        - AMI: Ubuntu
        - VPC/Subnet: `nautlius-pub-vpc` / `nautlius-pub-subnet`
        - Public IP: Enabled
        - Key pair: `nautlius-key.pem`
    2. Security group inbound:
        - SSH (22) from your client IP (recommended) or as required by the lab
    3. Attach an IAM role that can write to S3 (IAM → Roles):
        - Role name: `nautlius-s3-role`
        - Permissions: S3 write to your bucket (lab may use `AmazonS3FullAccess`; for production, scope to the bucket)
        - Attach this role to `nautlius-pub-ec2`
3. Create the S3 bucket (Console → S3)
    - Bucket name: `nautlius-s3-logs-14184`
    - Block all public access: ON
4. Create VPC peering (Console → VPC → Peering Connections)
    1. Create peering connection:
        - Name: `nautlius-vpc-peering`
        - Requester: `nautlius-priv-vpc`
        - Accepter: `nautlius-pub-vpc`
    2. Accept the peering request
5. Update route tables for peering (Console → VPC → Route Tables)
    1. In the private VPC route table (e.g., `nautlius-priv-rt`), add:
        - Destination: `10.20.0.0/16`
        - Target: `nautlius-vpc-peering`
    2. In `nautlius-pub-rt`, add:
        - Destination: (CIDR of `nautlius-priv-vpc`)
        - Target: `nautlius-vpc-peering`
6. Ensure security groups allow SSH over peering
    - On `nautlius-priv-ec2` security group: allow SSH (22) from the *private IP* or security group of `nautlius-pub-ec2`
    - Confirm NACLs (if used) also allow the traffic
7. SSH access pattern (bastion → private)
    - Correct flow: `aws-client` → `nautlius-pub-ec2` → `nautlius-priv-ec2`
    - Example (from your client):
        
        ```
        ssh -i nautlius-key.pem ubuntu@<PUBLIC_EC2_PUBLIC_IP>
        ```
        
    - Then (from your client, using ProxyJump):
        
        ```
        ssh -i nautlius-key.pem ubuntu@<PRIVATE_EC2_PRIVATE_IP> -J ubuntu@<PUBLIC_EC2_PUBLIC_IP>
        ```
        
8. Log transfer: private EC2 → public EC2 (SCP)
    1. On `nautlius-pub-ec2`, create the destination directory:
        
        ```
        mkdir -p ~/boot
        ```
        
    2. Option A (recommended): use the same key pair and copy from private using bastion routing.
    3. Option B (works, but manage keys carefully): set up SSH keys from private to public.
        - On `nautlius-priv-ec2`:
            
            ```
            ssh-keygen -t rsa
            cat ~/.ssh/rsa.pub
            ```
            
        - On `nautlius-pub-ec2`, append that public key to the `ubuntu` user:
            
            ```
            nano ~/.ssh/authorized_keys
            ```
            
    4. On `nautlius-priv-ec2`, create a cron job to copy the log to the public EC2 (replace `<PUBLIC_EC2_PRIVATE_IP>`):
        
        ```
        crontab -e
        ```
        
        Add:
        
        ```
        * * * * * /usr/bin/scp -o StrictHostKeyChecking=no /var/log/boots.log ubuntu@<PUBLIC_EC2_PRIVATE_IP>:/home/ubuntu/boot/boots.log
        ```
        
9. Upload to S3 from the public EC2
    1. Install AWS CLI v2 (only if it’s not already available):
        
        ```
        curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o awscliv2.zip
        sudo apt-get update
        sudo apt-get install -y unzip
        unzip awscliv2.zip
        sudo ./aws/install
        aws --version
        ```
        
    2. Create a cron job on `nautlius-pub-ec2` to upload the file:
        
        ```
        crontab -e
        ```
        
        Add:
        
        ```
        * * * * * /usr/local/bin/aws s3 cp /home/ubuntu/boot/boots.log s3://nautlius-s3-logs-14184/nautlius-priv-vpc/boot/boots.log
        ```
        

### Quick check

- **Expected result:** After a few minutes, the object exists in S3 at `s3://nautlius-s3-logs-14184/nautlius-priv-vpc/boot/boots.log` and its timestamp/contents update as the log changes.