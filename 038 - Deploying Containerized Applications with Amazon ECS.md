# Deploying Containerized Applications with Amazon ECS

---

<aside>

**Purpose:** Deploy a containerized app to Amazon ECS (Fargate) using an image stored in Amazon ECR, then verify access to the running task.

</aside>

---

### Procedure

1. Create a private ECR repository:
    1. Open the Amazon ECR console: [https://console.aws.amazon.com/ecr/repositories](https://console.aws.amazon.com/ecr/repositories)
    2. Choose Private repositories → Create repository.
    3. For Repository name, enter a name provided from the lab (e.g., `nautilus-ecr`).
    4. Choose Create.
2. Build and push the Docker image:
    1. Authenticate your Docker client to ECR:

```bash
aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.<region>.amazonaws.com

# e.g.
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 442199564561.dkr.ecr.us-east-1.amazonaws.com
```

1. Build the Docker image on your build host (e.g., `aws-client`):

```bash
cd pyapp
docker build -t nautilus-ecr:latest .
```

1. Tag the Docker image:

```bash
docker tag nautilus-ecr:latest <aws_account_id>.dkr.ecr.<region>.amazonaws.com/nautilus-ecr:latest

# e.g.
docker tag nautilus-ecr:latest 442199564561.dkr.ecr.us-east-1.amazonaws.com/nautilus-ecr:latest
```

1. Push the image to ECR:

```bash
docker push <aws_account_id>.dkr.ecr.<region>.amazonaws.com/nautilus-ecr:latest

# e.g.
docker push 442199564561.dkr.ecr.us-east-1.amazonaws.com/nautilus-ecr:latest
```

1. Create and configure the ECS cluster:
    1. Open the Amazon ECS console → Clusters.
    2. Choose Create cluster.
    3. Under Cluster configuration, set the cluster name (e.g., `nautilus-cluster`).
    4. Under Infrastructure, select Fargate only.
    5. Choose Create.
2. Create an ECS task definition:
    1. From the ECS console, go to Task definitions.
    2. Choose Create new task definition.
    3. Under Task definition configuration, set the name (e.g., `nautilus-taskdefinition`).
    4. Under Infrastructure requirements, select AWS Fargate.
    5. Choose OS and task size as needed.
    6. Under Container details, choose Browse ECR images and select the image from the `nautilus-ecr` repository.
    7. Leave other options as default and choose Create.
    8. Review the task definition after creation.
3. Deploy the application using an ECS service:
    1. Go to Clusters → `nautilus-cluster` → Services.
    2. Choose Create.
    3. On Service details, select the task definition and set the service name (e.g., `nautilus-service`).
    4. On Environment, select Fargate as the launch type.
    5. Choose the default VPC and select subnets as needed; leave other options as default.
    6. Choose Create.
4. Verify application access:
    1. Open the service and confirm Status is Active with 1 running task.
    2. In VPC, update the relevant security group inbound rules to allow HTTP from `0.0.0.0/0`.
    3. In ECS: Clusters → `nautilus-cluster` → Services → `nautilus-service` → Tasks.
    4. Open the running task, find the Public IP in the Configuration pane, and open it in a browser.
    5. Confirm the page loads (e.g., `Welcome to KKE AWS cloud labs!`).

### Quick check

- **Expected result:** The ECS service is stable with a running task, and the task’s public IP loads the application in your browser.