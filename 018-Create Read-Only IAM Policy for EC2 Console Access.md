# Create Read-Only IAM Policy for EC2 Console Access

---

<aside>

**Purpose:** Create an **IAM policy** that grants **read-only access to EC2** (all EC2 resources).

</aside>

---

### Policy details

- **Policy name:** `iampolicy_ravi`  (name provided each lab)
- **Service:** EC2
- **Access level:** Read
- **Resources:** All

### Procedure

1. Sign in to the AWS Console using the provided credentials and URL.
2. Open **IAM**:
    1. In the AWS search bar, type **IAM**.
    2. Select **IAM** from the results.
3. In the left sidebar, go to **Policies**.
4. Click **Create policy**.
5. Choose the policy editor:
    - Select **JSON**.
6. Paste the following policy (EC2 read-only, all resources):
    
    ```json
    {
    	"Version": "2012-10-17",
    	"Statement": [
    		{
    			"Sid": "EC2ReadyOnly",
    			"Effect": "Allow",
    			"Action": [
    				"ec2:DescribeInstances",
    				"ec2:DescribeImages",
    				"ec2:DescribeSnapshots"
    			],
    			"Resource": "*"
    		}
    	]
    }
    ```
    
    - Sid - set appropriate name.
    - Action
        - "ec2:DescribeInstances" - view all instances
        - "ec2:DescribeImages" - view all AMIs
        - "ec2:DescribeSnapshots" - view all Snapshots
    
        <img width="1462" height="907" alt="image" src="https://github.com/user-attachments/assets/013b96ce-39db-4cc8-aee3-2113e611f46a" />
    
7. Click **Next**.
8. Enter the policy name: `iampolicy_ravi`.
9. (Optional) Add a description, for example: "EC2 read-only (describe/get), all resources".
10. Click **Create policy**.
11. Confirm the policy appears in the list.
    
    <img width="1467" height="736" alt="image" src="https://github.com/user-attachments/assets/6344d7af-8b90-4f24-a0f7-c1050bc4c407" />
    

### Quick check

- **Location:** IAM → Policies
- **Expected result:** Policy `iampolicy_ravi` exists and can be attached to a group, role, or user
