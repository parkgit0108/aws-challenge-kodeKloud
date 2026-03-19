# Establishing Secure Communication Between Public and Private VPCs via VPC Peering

---

<aside>

**Purpose:** Establish **secure communication** between a **public VPC** and a **private VPC** using **VPC peering**, then validate connectivity (ICMP) and access (SSH).

</aside>

---

### Procedure

1. Create the VPC peering connection:
    1. Open **AWS Console → VPC**.
    2. In the navigation pane, choose **Peering connections**.
    3. Choose **Create peering connection**.
    4. Set **Name tag** (for example: `xfusion-vpc-peering`).
    5. Set **VPC ID (Requester)** to the public VPC (same account).
    6. Under **Select another VPC to peer with**:
        1. **Account:** choose **My account** (or **Another account** and enter the account ID).
        2. **Region:** choose **This Region** (or **Another Region** and select the region).
        3. **VPC ID (Accepter):** select the private VPC.
    7. Choose **Create peering connection**.
2. Accept the peering request:
    1. In **Peering connections**, select the pending request.
    2. Choose **Actions → Accept request**.
3. Update route tables (both sides):
    1. In **VPC → Route tables**, update the route table associated with the public VPC subnets (for example: `default`).
        1. Add a route where **Destination** is the private VPC CIDR.
        2. Set **Target** to the peering connection (for example: `xfusion-vpc-peering`).
    2. Update the route table associated with the private VPC subnets (for example: `xfusion-private-vpc`).
        1. Add a route where **Destination** is the public VPC CIDR.
        2. Set **Target** to the same peering connection.
4. Allow ICMP between instances (security groups):
    1. Update the security group for `xfusion-public-ec2`:
        1. Add an **Inbound rule**: **ICMP** from the **private VPC CIDR**.
        2. Add an **Inbound rule**: **SSH (22)** from `0.0.0.0/0` (Any IPv4).
    2. Update the security group for `xfusion-private-ec2`:
        1. Add an **Inbound rule**: **ICMP** from the **public VPC CIDR**.
5. Enable SSH access to the public instance (from the AWS client):
    1. Connect to `xfusion-public-ec2` using **EC2 Instance Connect** in the AWS Console.
    2. Add the **AWS client** public key (`id_rsa.pub`) to `~/.ssh/authorized_keys` on `xfusion-public-ec2`.
    3. Verify you can SSH from the client:
        
        ```
        ssh ec2-user@<public-ip-of-xfusion-public-ec2>
        ```
        
6. Verify connectivity to the private instance through peering:
    1. From `xfusion-public-ec2`, ping the private instance using its **private IP**.
        
        ```
        ping <private-ip-of-xfusion-private-ec2>
        ```
        

### Quick check

- **Location:** AWS Console (VPC, Route Tables, Security Groups) + EC2 Instance Connect/SSH terminal
- **Expected result:** Peering is **Active**, both route tables have the correct CIDR routes via the peering connection, ICMP rules allow traffic, and the public instance can **ping** the private instance using its private IP