# Configuring a Public VPC with an EC2 Instance for Internet Access

---

<aside>

**Purpose:** Create a **public VPC** with a **public subnet**, attach an **Internet Gateway**, configure **route tables**, then launch an **EC2 instance** that receives a **public IP** and has **internet access**.

</aside>

---

### Procedure

1. Create the public VPC:
    1. Open **VPC**:
        - AWS Console → **VPC**
    2. Create VPC:
        - Click **Create VPC**
        - Select **VPC only**
    3. Configure:
        - **Name:** `xfusion-pub-vpc`
        - **IPv4 CIDR block:** `10.0.0.0/16`
        - **Tenancy:** Default
    4. Create:
        - Click **Create VPC**
2. Create the public subnet:
    1. Open **Subnets**:
        - VPC → **Subnets**
    2. Create subnet:
        - Click **Create subnet**
    3. Configure:
        - **VPC:** `xfusion-pub-vpc`
        - **Subnet name:** `xfusion-pub-subnet`
        - **Availability Zone:** any (e.g. `us-east-1a`)
        - **IPv4 CIDR:** `10.0.1.0/24`
    4. Create:
        - Click **Create subnet**
3. Enable auto-assign public IPv4 on the subnet:
    1. Select **`xfusion-pub-subnet`**
    2. Click **Actions** → **Edit subnet settings**
    3. Enable:
        - **Auto-assign public IPv4 address**
    4. Save:
        - Click **Save**
4. Create and attach an Internet Gateway:
    1. Create Internet Gateway:
        1. Open **Internet Gateways**:
            - VPC → **Internet Gateways**
        2. Click **Create internet gateway**
        3. Configure:
            - **Name:** `xfusion-pub-igw`
        4. Create:
            - Click **Create**
    2. Attach Internet Gateway to VPC:
        1. Select **`xfusion-pub-igw`**
        2. Click **Actions** → **Attach to VPC**
        3. Select **`xfusion-pub-vpc`**
        4. Click **Attach**
5. Configure the route table for internet access:
    1. Open **Route tables**:
        - VPC → **Route Tables**
    2. Select the route table associated with **`xfusion-pub-vpc`**
    3. Add a default route:
        1. Click **Edit routes**
        2. Add route:
            - **Destination:** `0.0.0.0/0`
            - **Target:** Internet Gateway (**`xfusion-pub-igw`**)
        3. Click **Save changes**
    4. Associate the route table with the subnet:
        1. Go to **Subnet associations**
        2. Click **Edit subnet associations**
        3. Select **`xfusion-pub-subnet`**
        4. Click **Save**
6. Launch an EC2 instance in the public subnet:
    1. Open **EC2**:
        - AWS Console → **EC2** → **Launch instance**
    2. Configure:
        - **Name:** `xfusion-pub-ec2`
        - **AMI:** Amazon Linux or Ubuntu
        - **Instance type:** `t2.micro`
    3. Network settings:
        - **VPC:** `xfusion-pub-vpc`
        - **Subnet:** `xfusion-pub-subnet`
        - **Auto-assign public IP:** Enabled
    4. Security group:
        1. Create a new security group:
            - **Name:** `xfusion-pub-sg`
        2. Inbound rules:
            - **SSH:** 22 from `0.0.0.0/0`
        3. Outbound rules:
            - Allow all
    5. Launch:
        - Click **Launch instance**
7. Verify internet and SSH access:
    1. Wait until:
        - **State:** Running
        - **Status checks:** 2/2 passed
    2. Confirm the instance has a public IP:
        - **Public IPv4 address** is present on the instance details
    3. (Optional) Test connectivity:
        - SSH to the instance using the public IP and confirm it can reach the internet (for example, `curl https://example.com`)

### Quick check

- **Location:** AWS Console (VPC, EC2)
- **Expected result:** VPC has a **public subnet** with **auto-assign public IPv4 enabled**, route table has `0.0.0.0/0 → Internet Gateway`, and the EC2 instance is **Running** with a **Public IPv4 address**