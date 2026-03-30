# Scaling and Managing Kubernetes Clusters with Amazon EKS

---

<aside>

**Purpose:** Provision an Amazon EKS cluster with a dedicated IAM role and private-only API endpoint, then verify the cluster is Active.

</aside>

---

### Procedure

1. Create IAM role for the EKS cluster (Console → IAM):
    1. Open AWS Console → IAM.
    2. Go to Roles → Create role.
    3. Trusted entity type: AWS service.
    4. Use case: EKS.
    5. Specific use case: EKS – Cluster.
    6. Attach policy: `AmazonEKSClusterPolicy`.
    7. Role name: `eksClusterRole`.
    8. Choose Create role.
2. Create the EKS cluster (Console → EKS):
    1. Go to Services → Amazon EKS.
    2. Choose Clusters → Create cluster.
    3. Custom configuration: Disable “Use EKS Auto Mode”.
    4. Name: `xfusion-eks`.
    5. Cluster service role: `eksClusterRole`.
    6. Kubernetes version: `1.30`.
    7. Choose Next.
3. Configure networking:
    1. VPC: Default VPC.
    2. Subnets: select one subnet per AZ:
        - `us-east-1a`
        - `us-east-1b`
        - `us-east-1c`
    3. Endpoint access:
        - Public access: Disabled
        - Private access: Enabled
    4. Security groups: leave the default security group selected.
    5. Choose Next.
4. Configure logging and add-ons:
    1. Leave control plane logging as default.
    2. Leave add-ons unchanged.
    3. Choose Next.
5. Review and create:
    1. Confirm settings (name `xfusion-eks`, version `1.30`, private endpoint, default VPC/subnets).
    2. Choose Create.
6. Wait for cluster creation:
    1. In EKS → Clusters, select `xfusion-eks`.
    2. Wait until Status shows Active (typically 10–15 minutes).

### Quick check

- **Expected result:** The `xfusion-eks` cluster status is Active, Kubernetes version is `1.30`, and endpoint access is private only.