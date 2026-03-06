# Setting Up an EC2 Instance and CloudWatch Alarm

---

<aside>

**Purpose:** Launch an **EC2 instance** and create a **CloudWatch CPU alarm** (with **SNS notifications**), then verify the alarm and action are configured correctly.

</aside>

---

### Procedure

1. Launch the EC2 instance:
    1. Open **EC2**:
        - AWS Console → **EC2** → **Launch instance**
    2. Configure the instance:
        - **Name:** `datacenter-ec2`
        - **AMI:** Ubuntu Server (20.04 LTS or 22.04 LTS)
        - **Instance type:** `t2.micro` (or any allowed type)
        - Continue with our Key pair
        - Everything else default
    3. Launch:
        - Click **Launch instance**
        - Wait until:
            - **State:** Running
            - **Status checks:** 2/2 passed
2. Create the CloudWatch alarm:
    1. Open **CloudWatch**:
        - AWS Console → **CloudWatch**
        - Left menu → **Alarms**
        - Click **Create alarm**
            
            ![image.png](Setting%20Up%20an%20EC2%20Instance%20and%20CloudWatch%20Alarm/image.png)
            
    2. Select the metric:
        1. Click **Select metric**
        2. Navigate:
            - **EC2** → **Per-Instance Metrics** → `CPUUtilization`
                
                ![image.png](Setting%20Up%20an%20EC2%20Instance%20and%20CloudWatch%20Alarm/image%201.png)
                
        3. Select the metric for `datacenter-ec2`
        4. Click **Select metric**
    3. Configure alarm conditions:
        
        
        | Setting | Value |
        | --- | --- |
        | Statistic | Average |
        | Period | 5 minutes |
        | Threshold type | Static |
        | Condition | Greater than or equal to |
        | Threshold value | 90 |
        | Datapoints to alarm | 1 out of 1 |
        | InstanceId | matches the ec2 instance you created |
        
        ![image.png](Setting%20Up%20an%20EC2%20Instance%20and%20CloudWatch%20Alarm/image%202.png)
        
    4. Configure alarm actions:
        - **Alarm state trigger:** In alarm
        - **Send notification to:** Select existing SNS topic
        - **SNS topic:** `datacenter-sns-topic`
    5. Name the alarm:
        - **Alarm name:** lab provided
        - Click **Create alarm**
3. Verify configuration:
    - ✔ **CloudWatch Alarm**
        - State initially: **OK** (normal)
        - Metric: `CPUUtilization`
        - Threshold: **≥ 90%**
        - Period: **5 minutes**
    - ✔ **SNS Action**
        - Alarm action shows `datacenter-sns-topic`

### Quick check

- **Location:** AWS Console (EC2 + CloudWatch)
- **Expected result:** Instance is **Running**, and the alarm is created with **SNS notifications** configured