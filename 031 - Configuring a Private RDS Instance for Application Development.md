# Configuring a Private RDS Instance for Application Development

---

<aside>

**Purpose:** Create a **private** Amazon RDS **MySQL** database for application development, using the lab-required instance class and storage autoscaling settings.

</aside>

---

### Procedure

1. Open RDS Console:
    1. Go to **AWS Console → RDS**.
    2. Click **Create database**.
2. Choose database creation method:
    1. Select **Full configuration**.
3. Engine configuration:
    1. **Engine type:** MySQL
    2. **Engine version:** MySQL `8.4.x` (choose the latest `8.4` option available)
4. Templates:
    1. Select Free tier.
5. DB instance settings:
    1. **DB instance identifier:** name provided in the lab
    2. **Credentials:**
        - **Username:** keep default (for example: `admin`)
        - **Password:** auto-generate or set manually (either is fine for the lab)
6. Instance configuration:
    1. **DB instance class:** `db.t3.micro`
7. Storage configuration:
    1. Keep the default storage type.
    2. Enable **storage autoscaling**.
    3. Set **Maximum storage threshold:** `50 GB`
        - ⚠️ This value is mandatory.
8. Connectivity (private is required):
    1. **VPC:** Default or existing (keep default unless lab specifies otherwise)
    2. **Public access:** ❌ **No** (must be private)
    3. **Subnet group:** Default
    4. **Security group:** Default (or existing)
9. Additional configuration:
    1. Leave all other settings as default.
    2. No backup or monitoring changes are needed unless the selected template already enabled them.
10. Create database:
    1. Click **Create database**.
    2. Wait for the database status to become **Available**.
        - Go to **RDS → Databases**.
        - Select `nautilus-rds`.
        - Wait until **Status = Available** (typically 5–10 minutes).
        - 🚫 Do not submit while status is **Creating**, **Modifying**, or **Backing-up**.

### Quick check

- **Location:** AWS Console → RDS → Databases
- **Expected result:** `nautilus-rds` exists and shows **Status = Available**, with **Publicly accessible = No**.