# Snapshot and Restoration of an RDS Instance

---

<aside>

**Purpose:** Create a manual snapshot of an existing Amazon RDS instance and restore it into a **new** RDS instance using lab-required settings.

</aside>

---

### Procedure

1. Ensure the source RDS instance is available:
    1. Go to **AWS Console → RDS**.
    2. Click **Databases**.
    3. Confirm rds instance shows **Status = Available**.
        - ⚠️ If it is not **Available**, wait before proceeding.
2. Create an RDS snapshot:
    1. Select `rds that is provided from the lab`
    2. Click **Actions → Take snapshot**.
    3. Enter:
        - **Snapshot name:** name as lab provided
    4. Click **Take snapshot**.
    5. Wait for the snapshot to become available:
        - Go to **Snapshots**.
        - Confirm rds snapshot shows **Status = Available**.
3. Restore the snapshot to a new RDS instance:
    1. In **Snapshots**, select the rds instance
    2. Click **Actions → Restore snapshot**.
4. Configure the restored RDS instance:
    1. Under **DB instance settings**:
        - **DB instance identifier:** `name as per the lab requests`
    2. Under **Instance configuration**:
        - **DB instance class:** `db.t3.micro`
    3. Under **Connectivity**:
        - **Public access:** **No** (keep private)
    4. Under **Other settings**:
        - Leave all other options as default.
    5. Click **Restore DB instance**.
5. Wait for completion:
    1. Go back to **Databases**.
    2. Select the restored snapshot
    3. Wait until **Status = Available**.
        - ⏳ This may take several minutes.

### Quick check

- **Location:** AWS Console → RDS → Databases
- **Expected result:** `restored snapshot` exists and shows **Status = Available**, with **Publicly accessible = No**.