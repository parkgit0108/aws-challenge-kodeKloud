# Configure NAT Gateway for Internet Access in a Private VPC

---

<aside>

**Purpose:** Provide outbound internet access for instances in a private subnet by routing egress traffic through a NAT Gateway in a public subnet.

</aside>

---

### Procedure

1. Create a public subnet (Console → VPC):
    1. Go to VPC → Subnets → Create subnet.
    2. VPC: devops-priv-vpc.
    3. Subnet name: devops-pub-subnet.
    4. Choose a valid CIDR block.
    5. Enable auto-assign public IPv4 address for the public subnet (Subnet → Edit subnet settings → Auto-assign IP settings).
2. Create an Internet Gateway (IGW) and attach it to the VPC (Console → VPC):
    1. Go to VPC → Internet Gateways → Create internet gateway.
    2. Name: devops-ig.
    3. Select the new IGW → Actions → Attach to VPC → choose devops-priv-vpc.
3. Create a public route table and associate it with the public subnet (Console → VPC):
    1. Go to VPC → Route tables → Create route table.
    2. Name: devops-pub-rt.
    3. VPC: devops-priv-vpc.
    4. Open devops-pub-rt → Routes → Edit routes → Add route:
        - Destination: 0.0.0.0/0
        - Target: Internet Gateway (devops-ig)
    5. Open devops-pub-rt → Subnet associations → Edit subnet associations → select devops-pub-subnet.
4. Create a NAT Gateway in the public subnet (Console → VPC):
    1. Go to VPC → NAT Gateways → Create NAT gateway.
    2. Name: devops-nat.
    3. Subnet: devops-pub-subnet.
    4. Connectivity type: Public.
    5. Elastic IP allocation: Allocate a new Elastic IP.
    6. Create NAT gateway and wait until Status = Available.
5. Update the private route table to use the NAT Gateway for internet-bound traffic (Console → VPC):
    1. Go to VPC → Route tables and identify the route table associated with devops-priv-subnet.
    2. If the private subnet is not associated yet: Subnet associations → Edit subnet associations → select devops-priv-subnet.
    3. Routes → Edit routes → Add/Update route:
        - Destination: 0.0.0.0/0
        - Target: NAT Gateway (devops-nat)
6. Verify outbound access from the private instance:
    1. SSH to the EC2 instance in devops-priv-subnet (via a bastion host or Session Manager).
    2. Verify internet egress (example): curl [https://aws.amazon.com](https://aws.amazon.com)
    3. If the lab includes an S3 access test, confirm the expected file/object upload from the instance is present in the target bucket.

### Quick check

- **Expected result:** Instances in the private subnet can reach the internet (outbound) via the NAT Gateway, while still not being directly reachable from the public internet.