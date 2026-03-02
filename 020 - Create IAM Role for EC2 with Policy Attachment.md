# Create IAM Role for EC2 with Policy Attachment

---

<aside>

**Purpose:** Create an **IAM role** for **EC2** and attach the required **IAM policy**.

</aside>

---

### Procedure

1. Sign in to the AWS Console using the provided credentials and URL.
2. Open **IAM**:
    1. In the AWS search bar, type **IAM**.
    2. Select **IAM** from the results.
3. In the left sidebar, go to **Roles**.
4. Click **Create role**.

    <img width="1463" height="618" alt="image" src="https://github.com/user-attachments/assets/575c0dc9-cff8-47e5-afbe-e391cf366e24" />
    
5. Select the **trusted entity type** and **use case** as provided.
    
    <img width="1466" height="988" alt="image" src="https://github.com/user-attachments/assets/92cfa055-cff0-4ed2-9579-6fe3e251b578" />
    
6. Search for the policy name provided, then select it to attach.
    
7. Set the **role name** as provided, review, then click **Create role**.

    <img width="1460" height="978" alt="image" src="https://github.com/user-attachments/assets/7f95247d-1cc1-4e02-b28f-734e6171033c" />
    
8. Confirm the role exists:
    1. Go back to **Roles**.
    2. Search for the role name.
    3. Confirm it appears in the list.
    

### Quick check

- **Location:** IAM → Roles
- **Expected result:** New role appears in the list and shows the intended policy under **Permissions policies**
