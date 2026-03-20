# Enable Internet Access for Private EC2 using NAT Instance

---

<aside>

**Purpose:** Enable **internet access** for instances in a **private subnet** by routing outbound traffic through a **NAT instance** in a **public subnet**.

</aside>

---

### Procedure

1. Create a public subnet:
    1. Go to **VPC → Subnets → Create subnet**.
    2. Configure:
        - **VPC:** `nautilus-priv-vpc`
        - **Subnet name:** `nautilus-pub-subnet`
        - **AZ:** any AZ in the same region
        - **CIDR:** for example `10.1.2.0/24` (must not overlap)
    3. Choose **Create subnet**.
    4. Enable public IP auto-assign:
        1. Select `nautilus-pub-subnet`.
        2. Choose **Actions → Edit subnet settings**.
        3. Enable **Auto-assign public IPv4 address**.
        4. Choose **Save**.
2. Ensure an Internet Gateway exists and is attached:
    1. Go to **VPC → Internet Gateways**.
    2. If none exists:
        1. Create one (for example: `nautilus-igw`).
        2. Attach it to `nautilus-priv-vpc`.
3. Create a route table for the public subnet:
    1. Go to **VPC → Route Tables → Create route table**.
    2. Configure:
        - **Name:** `nautilus-pub-rt`
        - **VPC:** `nautilus-priv-vpc`
    3. Choose **Create**.
    4. Add a default route to the internet:
        1. Choose **Edit routes → Add route**.
        2. Set **Destination:** `0.0.0.0/0`.
        3. Set **Target:** your **Internet Gateway** (for example: `nautilus-igw`).
        4. Choose **Save changes**.
    5. Associate the route table with the public subnet:
        1. Go to **Subnet associations → Edit subnet associations**.
        2. Select `nautilus-pub-subnet`.
        3. Choose **Save associations**.
4. Create a security group for the NAT instance:
    1. Go to **EC2 → Security Groups → Create security group**.
    2. Configure:
        - **Name:** `nautilus-nat-sg`
        - **VPC:** `nautilus-priv-vpc`
    3. Add inbound rules:
        - **All traffic** from the **private subnet CIDR** (for example: `10.1.0.0/16`).
        - **SSH (22)** from your admin IP or trusted source.
    4. Choose **Create security group**.
5. Launch the NAT instance in the public subnet:
    1. Go to **EC2 → Launch instance**.
    2. Configure:
        - **Name:** `nautilus-nat-instance`
        - **AMI:** Amazon Linux 2
        - **Instance type:** `t2.micro`
        - **VPC:** `nautilus-priv-vpc`
        - **Subnet:** `nautilus-pub-subnet`
        - **Auto-assign public IP:** Enabled
        - **Security group:** `nautilus-nat-sg`
    3. Choose **Launch instance**.
6. Disable source/destination check (critical):
    1. Select `nautilus-nat-instance`.
    2. Choose **Actions → Networking → Change source/destination check**.
    3. Disable it, then **Save**.
        - ⚠️ If you skip this, NAT will not work.
7. Enable IP forwarding and configure NAT on the instance:
    1. Connect to the NAT instance (SSH or **EC2 Instance Connect**).
    2. Run:
        
        ```
        sudo dnf install -y iptables-services
        
        # Enable IP forwarding
        sudo sysctl -w net.ipv4.ip_forward=1
        echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
        
        # Flush old NAT rules
        sudo iptables -t nat -F
        sudo iptables -F
        
        # NAT for the private subnet
        sudo iptables -t nat -A POSTROUTING -s 10.1.0.0/16 -o enX0 -j MASQUERADE
        
        # Allow forwarding
        sudo iptables -A FORWARD -i enX0 -m state --state RELATED,ESTABLISHED -j ACCEPT
        sudo iptables -A FORWARD -o enX0 -s 10.1.0.0/16 -j ACCEPT
        
        # Make iptables persistent
        sudo systemctl enable iptables
        sudo systemctl restart iptables
        ```
        
        - Note: Replace `enX0` with the NAT instance’s actual outbound network interface.
8. Update the private subnet route table to send internet-bound traffic to the NAT instance:
    1. Go to **VPC → Route Tables**.
    2. Select the route table associated with `nautilus-priv-subnet`.
    3. Choose **Edit routes → Add route**.
    4. Set:
        - **Destination:** `0.0.0.0/0`
        - **Target:** `nautilus-nat-instance` (instance ID)
    5. Choose **Save changes**.

### Quick check

- **Location:** AWS Console (VPC, Route Tables, Security Groups) + EC2 terminal
- **Expected result:** The private subnet route table has `0.0.0.0/0 → nautilus-nat-instance`, and instances in the private subnet can reach the internet.
- **Verification:** Wait 1–2 minutes, then run:
    
    ```
    aws s3 ls s3://nautilus-nat-28334
    ```