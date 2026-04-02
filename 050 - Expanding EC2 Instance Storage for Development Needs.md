# Expanding EC2 Instance Storage for Development Needs

---

<aside>

**Purpose:** Expand the root EBS volume for the **xfusion-ec2** instance (8 GiB → 12 GiB) and grow the root partition/filesystem so the OS can use the new space.

</aside>

---

### Procedure

1. Identify the volume attached to xfusion-ec2 (from the aws-client host):
    
    ```
    aws ec2 describe-instances \
      --filters "Name=tag:Name,Values=xfusion-ec2" \
      --query "Reservations[].Instances[].BlockDeviceMappings[].Ebs.VolumeId" \
      --output text
    ```
    
    Expected output (example):
    
    ```
    vol-057ed3f876c222e27
    ```
    
2. Expand the EBS volume (8 GiB → 12 GiB):
    
    ```
    aws ec2 modify-volume \
      --volume-id vol-057ed3f876c222e27 \
      --size 12
    ```
    
3. Verify the modification is complete:
    
    ```
    aws ec2 describe-volumes-modifications \
      --volume-ids vol-057ed3f876c222e27
    ```
    
    Confirm the resize finished in AWS (either is fine):
    
    - Console: EC2 → Volumes → select the volume → confirm **Size = 12 GiB** and the volume state is **In-use**.
    - ✅ At this point AWS has expanded the disk.
    - ⚠️ The instance OS does not automatically use the new space until you grow the partition/filesystem.
4. SSH into the EC2 instance:
    
    ```
    ssh -i /root/xfusion-keypair.pem ec2-user@<PUBLIC_IP>
    ```
    
5. Verify the disk/partition layout (inside the EC2 instance):
    
    ```
    lsblk
    ```
    
    Expected example (the disk may show 12G, while the root partition is still 8G until you grow it):
    
    ```
    NAME      MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
    xvda      202:0    0  12G  0 disk
    ├─xvda1   202:1    0   8G  0 part /
    ├─xvda127 259:0    0   1M  0 part
    └─xvda128 259:1    0  10M  0 part /boot/efi
    ```
    
6. Grow the root partition and filesystem:
    1. Identify the root disk device (common examples):
        - /dev/xvda
        - /dev/nvme0n1
    2. Grow partition 1 (example for /dev/xvda):
        
        ```
        sudo growpart /dev/xvda 1
        ```
        
    3. Grow the filesystem:
        - If the root filesystem is XFS (common on Amazon Linux):
            
            ```
            sudo xfs_growfs /
            ```
            
7. Verify the final result:
    
    ```
    df -h /
    ```
    
    Expected: the root filesystem reflects ~12G.
    

### Quick check

- **Expected result:** Root disk was expanded in AWS and the OS now sees the increased space (root filesystem ~12 GiB). No reboot required.