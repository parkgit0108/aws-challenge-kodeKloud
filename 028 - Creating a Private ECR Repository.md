# Creating a Private ECR Repository

---

<aside>

**Purpose:** Create a **private Amazon ECR repository**, authenticate Docker to ECR, then **build, tag, and push** a Docker image so it appears in the repository.

</aside>

---

### Procedure

1. Verify Docker and AWS CLI:
    1. Check Docker is available:
        
        ```
        docker --version
        ```
        
    2. Confirm AWS CLI credentials are configured:
        
        ```
        aws sts get-caller-identity
        ```
        
    3. (Optional) Confirm the expected AWS region:
        
        ```
        aws configure get region
        ```
        
2. Create the ECR repository:
    1. Create the repository:
        
        ```
        aws ecr create-repository \
          --repository-name xfusion-ecr \
          --region us-east-1
        ```
        
    2. Copy the `repositoryUri` from the output (you will use it for login and tagging).
3. Authenticate Docker to ECR:
    1. Log in to ECR using an auth token:
        
        ```
        aws ecr get-login-password --region us-east-1 \
        | docker login --username AWS --password-stdin \
        022390610537.dkr.ecr.us-east-1.amazonaws.com/xfusion-ecr
        ```
        
    2. Expected output:
        - `Login Succeeded`
4. Build the Docker image:
    1. Go to the directory with the Dockerfile:
        
        ```
        cd /root/pyapp
        ```
        
    2. Build the image:
        
        ```
        docker build -t xfusion-ecr:latest .
        ```
        
    3. Verify the image exists locally:
        
        ```
        docker images
        ```
        
5. Tag the image for ECR:
    1. Tag the local image with the repository URI:
        
        ```
        docker tag xfusion-ecr:latest \
        022390610537.dkr.ecr.us-east-1.amazonaws.com/xfusion-ecr:latest
        ```
        
6. Push the image to ECR:
    1. Push the tagged image:
        
        ```
        docker push 022390610537.dkr.ecr.us-east-1.amazonaws.com/xfusion-ecr:latest
        ```
        
    2. Expected result:
        - Layer upload progress
        - Push completes successfully
7. Verify the image exists in ECR:
    1. List images in the repository:
        
        ```
        aws ecr list-images \
          --repository-name xfusion-ecr \
          --region us-east-1
        ```
        
    2. Confirm the `latest` image tag (or image digest) is returned.

### Quick check

- **Location:** AWS CLI terminal + AWS Console (ECR)
- **Expected result:** ECR repository exists, Docker login succeeds, the image is pushed successfully, and `aws ecr list-images` returns the image