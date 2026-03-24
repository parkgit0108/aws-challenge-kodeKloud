# Deploying and Managing Applications on AWS

---

<aside>

**Purpose:** Deploy and configure an EC2-hosted web app on AWS by enabling SSH access, creating a private MySQL RDS instance, and deploying an `index.php` connectivity check page.

</aside>

---

### Procedure

#### Task 1: Update EC2 security group inbound rules

1. Go to the EC2 dashboard and select Security groups under Network & Security from the navigation pane.
2. Select the available default security group.
3. Edit Inbound rules as required.
    * Allow SSH, HTTP from anywhere.

#### Task 2: Enable SSH access to nautilus-ec2 from aws-client

1. Generate SSH keys in aws-client.

```
ssh-keygen -t rsa
```

1. Connect to nautilus-ec2 using EC2 Instance Connect.
2. Add the aws-client public key id_[rsa.pub](http://rsa.pub) to the root user’s authorized_keys on nautilus-ec2.
    1. After SSH into nautilus-ec2, switch to root user.

```
sudo -i
```

1. Move to the root directory.

```
cd ~
```

1. Add the aws-client public key to .ssh/authorized_keys.

```
nano .ssh/authorized_keys
```

1. SSH into nautilus-ec2 from aws-client as root.

```
ssh root@<ec2-pub-ip>
```

#### Task 3: Create a private RDS instance

1. Sign in to the AWS Management Console and open the Amazon RDS console.
2. Click Create a database.
3. In Choose a database creation method, choose Full configuration.
4. In Engine Options, select MySQL.
5. Set Engine version to MySQL v8.4.5.
6. In Templates, select Free tier.
7. In Availability and durability, choose Single-AZ DB instance deployment (1 instance).
8. In Settings, enter the instance name and master username.
9. Enter an appropriate master password.
10. In Instance configuration, set:
    - DB instance class: Burstable classes (includes t classes)
    - Size: db.t3.micro
11. In Storage, set:
    - Storage type: gp2
    - Allocate storage: 5
12. In Connectivity → Compute resource:
    - Select Connect to an EC2 compute resource
    - Choose nautilius-ec2
    - Leave all other choices as default
13. In Additional configuration, set Initial database name (under Database option) to nautilus_db.
14. Click Create database.

#### Task 4: Copy index.php to nautilius-ec2

1. On the aws-client, edit index.php values to match what was created.

```
<?php
 $dbname = 'nautilus_db';
 $dbuser = 'nautilus_admin';
 $dbpass = 'nautilus_db';
 $dbhost = 'nautilus-rds.cgmyyysfambv.us-east-1.rds.amazonaws.com';

 $link = mysqli_connect($dbhost, $dbuser, $dbpass) or die("Unable to Connect to '$dbhost'");
 mysqli_select_db($link, $dbname) or die("Could not open the db '$dbname'");

 $test_query = "SHOW TABLES FROM$dbname";
 $result = mysqli_query($link, $test_query);

 $tblCnt = 0;
 while($tbl = mysqli_fetch_array($result)) {
   $tblCnt++;
 }

 if (!$tblCnt) {
   echo "Connected successfully<br />\n";
 } else {
   echo "Connected successfully<br />\n";
 }
?>
```

1. Copy the index.php file to nautilius-ec2 → /var/www/html/.
    
    `scp index.php root@<ec2-public-IP>:/var/www/html/`
    
2. Browse the ec2 instance public IP.
    - Expected output: Connected successfully

### Quick check

- **Expected result:** Browsing the instance public IP shows: Connected successfully
