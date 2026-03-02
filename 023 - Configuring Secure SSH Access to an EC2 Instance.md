# Configuring Secure SSH Access to an EC2 Instance

---

<aside>

**Purpose:** Configure **secure SSH access** to a new **EC2 instance** using an **SSH key pair** generated on the client.

</aside>

---

### Procedure

1. Create an EC2 instance (no key pair):
    1. Click **Launch instance**.
    2. Set the instance **name** and select settings as required.
    3. For **Key pair**, select **Proceed without a key pair** (No key pair).
    4. Click **Launch instance**.
2. On the AWS client, generate a key pair:
    1. Run `ssh-keygen`.
    2. Confirm where the key is saved.
        
        <img width="1310" height="799" alt="image" src="https://github.com/user-attachments/assets/5b332b2e-1842-4611-add1-3b826fec59f0" />
        
3. Copy the public key content:
    1. Run `cat /root/.ssh/id_rsa.pub`.
    2. Copy the output.
        
4. Add the public key to the EC2 instance `authorized_keys`:
    1. Select the EC2 instance and **connect** to the console.
        
        <img width="1471" height="576" alt="image" src="https://github.com/user-attachments/assets/f83da694-7ff3-4c1a-a7ea-9a000d116792" />
        
    2. Open `/root/.ssh/authorized_keys` in **nano** or **vi**.
    3. Paste the public key output from the previous step.
    4. Save and exit.
5. Test SSH connectivity:
    1. From the AWS client, SSH to the EC2 **public IP**.
        
        <img width="1469" height="613" alt="image" src="https://github.com/user-attachments/assets/c592c38e-ae72-4962-b4e8-dffd122a7f69" />
        

### Quick check

- **Location:** AWS client terminal
- **Expected result:** SSH connection succeeds using the key you generated
