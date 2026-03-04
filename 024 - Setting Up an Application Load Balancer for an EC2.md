# Setting Up an Application Load Balancer for an EC2 Instance

---

<aside>

**Purpose:** Create an **Application Load Balancer (ALB)**, attach it to an **EC2 target group**, and verify traffic reaches the instance (Nginx example).

</aside>

---

### Procedure

1. Create a security group for the ALB:
    1. Go to **EC2** → **Security Groups**.
    2. Click **Create security group**.
    3. Set:
        - **Security group name:** name is provided
        - **Description:** ALB security group
        - **VPC:** Default VPC
    4. Under **Inbound rules**, click **Add rule**:
        - **Type:** HTTP
        - **Port:** 80
        - **Source:** `0.0.0.0/0`
    5. Leave **Outbound rules** as default.
    6. Click **Create security group**.
        - ✅ This allows public internet traffic to the ALB.
2. Create a target group:
    1. Go to **EC2** → **Target Groups**.
    2. Click **Create target group**.
    3. Configure:
        - **Target type:** Instances
        - **Target group name:** name is provided
        - **Protocol:** HTTP
        - **Port:** 80
        - **VPC:** Default VPC
    4. Under **Health checks**:
        - **Protocol:** HTTP
        - **Path:** `/` (default, works with Nginx)
    5. Click **Next**.
    6. Select `devops-ec2`:
        - **Port:** 80
    7. Click **Include as pending**.
        
        <img width="1420" height="706" alt="image" src="https://github.com/user-attachments/assets/70cf8f2a-927a-49ec-85c4-6806ea6ddebe" />
        
    8. Click **Create target group**.
3. Create an Application Load Balancer (`devops-alb`):
    1. Go to **EC2** → **Load Balancers**.
    2. Click **Create load balancer**.
    3. Choose **Application Load Balancer**.
    4. Under **Basic configuration**:
        - **Name:** name is provided
        - **Scheme:** Internet-facing
        - **IP address type:** IPv4
    5. Under **Network mapping**:
        - **VPC:** Default VPC
        - **Availability Zones:** Select at least 2 subnets
    6. Under **Security groups**:
        - Remove the default security group
        - Select `devops-sg`
    7. Under **Listeners & routing**:
        - **Listener:** HTTP : 80
        - **Forward to:** `target group you created`
    8. Click **Create load balancer**.
4. Update the EC2 instance security group (required for health checks):
    1. Go to **EC2** → **Instances**.
    2. Remove default Security Group and add the group you created.
5. Verify everything works:
    1. Check target group health:
        1. Go to **EC2** → **Target Groups** → `devops-tg`.
        2. Confirm targets show **Healthy**.
    2. Access the ALB:
        1. Go to **EC2** → **Load Balancers** → `devops-alb`.
        2. Copy the **DNS name - can be found in load balancer page**
            
            <img width="1448" height="928" alt="image" src="https://github.com/user-attachments/assets/ec89e1cc-89f6-462e-b1da-6e1d657539e2" />

        3. Open in a browser: `http://<alb-dns-name>`
        4. ✅ You should see the Nginx welcome page.
            
            <img width="1449" height="559" alt="image" src="https://github.com/user-attachments/assets/463265b2-0611-4e96-aa7f-ebb92a3b71cf" />
            

### Quick check

- **Location:** AWS Console (EC2) + a browser
- **Expected result:** Target group shows **Healthy**, and the ALB DNS loads the **Nginx welcome page**
