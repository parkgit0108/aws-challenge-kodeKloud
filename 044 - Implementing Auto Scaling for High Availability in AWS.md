# Implementing Auto Scaling for High Availability in AWS

---

<aside>

**Purpose:** Create an Auto Scaling Group behind an Application Load Balancer (ALB) for high availability, then verify targets are healthy and the application is reachable via the ALB DNS name.

</aside>

---

### Procedure

1. Create security group for EC2 and ALB (Console → EC2):
    1. Go to EC2 → Security Groups.
    2. Choose Create security group.
    3. Security group name: `devops-sg`.
    4. VPC: Default VPC.
    5. Inbound rules: add HTTP:
        - Type: HTTP
        - Port: 80
        - Source: `0.0.0.0/0`
    6. Leave outbound rules as default.
    7. Choose Create security group.
2. Create EC2 launch template (Console → EC2):
    1. Go to EC2 → Launch Templates → Create launch template.
    2. Launch template name: `devops-launch-template`.
    3. AMI: Amazon Linux 2.
    4. Instance type: `t2.micro`.
    5. Security group: `devops-sg`.
    6. Under Advanced details → User data, paste:
        
        ```
        #!/bin/bash
        dnf update -y
        dnf install -y nginx
        systemctl start nginx
        systemctl enable nginx
        ```
        
    7. Choose Create launch template.
3. Create target group (Console → EC2):
    1. Go to EC2 → Target Groups → Create target group.
    2. Target type: Instances.
    3. Target group name: `devops-tg`.
    4. Protocol: HTTP.
    5. Port: 80.
    6. VPC: Default VPC.
    7. Health check:
        - Protocol: HTTP
        - Path: `/`
    8. Choose Create target group.
4. Create application load balancer (Console → EC2):
    1. Go to EC2 → Load Balancers → Create load balancer.
    2. Choose Application Load Balancer.
    3. Name: `devops-alb`.
    4. Scheme: Internet-facing.
    5. IP address type: IPv4.
    6. Network mapping:
        - VPC: Default VPC
        - Subnets: select at least two Availability Zones
    7. Security group: select `devops-sg`.
    8. Listener and routing:
        - Listener: HTTP : 80
        - Forward to: `devops-tg`
    9. Choose Create load balancer.
5. Create auto scaling group (Console → EC2):
    1. Go to EC2 → Auto Scaling Groups → Create Auto Scaling group.
    2. Choose launch template:
        - Launch template: `devops-launch-template`
        - Version: Latest
        - Choose Next
    3. Network:
        - VPC: Default VPC
        - Subnets: choose the same AZs/subnets used for the ALB
        - Choose Next
    4. Load balancing:
        - Attach to an existing load balancer
        - Choose target group: `devops-tg`
        - Enable ELB health checks
        - Choose Next
    5. Group size and scaling:
        - Minimum capacity: 1
        - Desired capacity: 1
        - Maximum capacity: 2
    6. Scaling policy:
        - Select Target tracking scaling policy
        - Metric: Average CPU utilization
        - Target value: `50`
    7. Choose Next → Create Auto Scaling group.
6. Verify target health (Console → EC2):
    1. Go to EC2 → Target Groups → `devops-tg`.
    2. Open the Targets tab.
    3. Confirm the registered instance status shows Healthy.
7. Verify application access via ALB:
    1. Go to EC2 → Load Balancers → select `devops-alb`.
    2. Copy the DNS name.
    3. Open it in a browser using HTTP: `http://<alb-dns-name>`.

### Quick check

- **Expected result:** At least one target in `devops-tg` is Healthy, and the Nginx default page loads via the `devops-alb` DNS name.