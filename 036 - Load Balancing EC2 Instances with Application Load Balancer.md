# Load Balancing EC2 Instances with Application Load Balancer

---

<aside>

**Purpose:** Create an Application Load Balancer (ALB) and target group for an EC2 instance, then verify traffic reaches the Nginx web server.

</aside>

---

### Procedure

1. Create / update security groups:
    1. Go to the EC2 dashboard and select Security Groups from the navigation pane.
    2. Create a new security group named nautilus-sg for nautilus-ec2 to use.
    3. Confirm you have two security groups available:
        - default (for the load balancer)
        - nautilus-sg (for the EC2 instance)
2. Adjust default security group (for ALB):
    1. Add an inbound rule:
        - Type: HTTP (80)
        - Source: 0.0.0.0/0
    2. Click Save.
3. Create the EC2 instance:
    1. Go to EC2 Dashboard and click Launch Instance.
    2. Enter name nautilus-ec2.
    3. For AMI, select Ubuntu.
    4. Under Network settings:
        - Select Existing security group
        - Choose nautilus-sg
    5. Under Advanced settings, enter the following in User data:

```bash
#!/bin/bash
apt update -y
apt install -y nginx
systemctl start nginx
systemctl enable nginx
```

1. Click Launch instance and wait for the instance to reach Running.
2. Create the target group:
    1. Go to the EC2 dashboard and select Target Groups from the navigation pane.
    2. Set Target type to Instance, then enter a name and define ports.
    3. Click Next, then select the nautilus-ec2 instance and port.
        - Click Include as pending below (it should appear in the Targets section).
    4. In Review and create, review the settings and click Create target group.
3. Create the Application Load Balancer:
    1. Go to the EC2 dashboard and select Load Balancers from the navigation pane.
    2. Click Create Load Balancer and select Application Load Balancer.
    3. Under Basic configuration:
        - Enter name
        - Scheme: Internet-facing
        - IP address type: IPv4
    4. Under Network mapping:
        - Select the default VPC
        - Select at least two subnets
        - Make sure one subnet matches the subnet your nautilus-ec2 instance is in
    5. For Security groups, select default.
    6. Under Listeners and routing, select your target group (e.g., nautilus-tg).
    7. Click Create load balancer.

### Quick check

- **Expected result:** Opening the load balancer DNS name in a browser shows the Nginx welcome page.