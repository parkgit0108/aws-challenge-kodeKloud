# Troubleshooting Internet Accessibility for an EC2-Hosted Application

---

<aside>

**Purpose:** Troubleshoot why an EC2-hosted application (e.g., Nginx) is not reachable from the public internet by validating VPC internet access, routing, subnet settings, and instance IP assignment.

</aside>

---

### Procedure

1. Verify Internet Gateway (IGW) is attached to the VPC:
    1. Go to VPC → Internet Gateways.
    2. Confirm an Internet Gateway exists and is attached to `xfusion-vpc`.
    3. If missing, select the IGW → Actions → Attach to VPC → choose `xfusion-vpc`.
    4. Confirm the VPC now has a path to the internet.
2. Verify route table configuration (public subnet routes):
    1. Go to VPC → Route Tables.
    2. Identify the route table associated with the public subnet used by `xfusion-ec2`.
    3. Check Subnet associations.
    4. Open the Routes tab.
    5. Ensure a default route exists:
        - Destination: `0.0.0.0/0`
        - Target: Internet Gateway (`igw-xxxx`)
    6. If the route is incorrect, delete it and add a new route:
        - Destination: `0.0.0.0/0`
        - Target: Internet Gateway
    7. Save changes.
3. Verify subnet auto-assign public IPv4 address:
    1. Go to VPC → Subnets.
    2. Select the subnet where `xfusion-ec2` is running.
    3. Click Edit subnet settings.
    4. Ensure Enable auto-assign public IPv4 address is checked.
    5. Save.
    6. Note: If disabled, new instances won’t get a public IP automatically.
4. Verify the EC2 instance has a public IP:
    1. Go to EC2 → Instances.
    2. Select `xfusion-ec2`.
    3. Check the Public IPv4 address field.
5. Final internet access test:
    1. Open a browser.
    2. Visit `http://<EC2_PUBLIC_IP>`.
    3. Confirm the Nginx Welcome Page loads.

### Quick check

- **Expected result:** The instance is publicly reachable and the application loads in a browser (e.g., Nginx Welcome Page).