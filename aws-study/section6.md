# Section 6: ECS, ECR & Fargate - Docker in AWS

**AWS Certified Developer - Associate**  
*Nguồn: Cloud Mentor Pro - Cập nhật: 15/02/2026*

---

## Mục lục

1. [What is Docker?](#1-what-is-docker)
2. [Docker Repositories](#2-docker-repositories)
3. [ECS Overview](#3-ecs-overview)
4. [ECS Task Definitions](#4-ecs-task-definitions)
5. [ECS Launch Types](#5-ecs-launch-types)
6. [ECS Service](#6-ecs-service)
7. [ECS Data Volumes](#7-ecs-data-volumes)
8. [ECS Scaling](#8-ecs-scaling)
9. [ECS Load Balancing](#9-ecs-load-balancing)
10. [ECS Environment Variables](#10-ecs-environment-variables)
11. [Amazon ECR](#11-amazon-ecr)
12. [AWS Copilot](#12-aws-copilot)
13. [Amazon EKS Overview](#13-amazon-eks-overview)
14. [Tổng kết](#14-tổng-kết)

---

## 1. What is Docker?

> Docker is a software development platform to deploy apps  
> *Docker là nền tảng phát triển phần mềm để deploy apps*

> Apps are packaged in containers that can be run on any OS  
> *Apps được đóng gói trong containers có thể chạy trên bất kỳ OS nào*

> Apps run the same, regardless of where they're run  
> *Apps chạy giống nhau, bất kể chạy ở đâu*

**Lợi ích của Docker:**

| Benefit | Mô tả |
|---------|--------|
| **Any machine** | Chạy trên bất kỳ máy nào |
| **No compatibility issues** | Không có vấn đề tương thích |
| **Predictable behavior** | Hành vi có thể dự đoán |
| **Less work** | Ít công việc hơn |
| **Easier to maintain and deploy** | Dễ bảo trì và deploy hơn |
| **Works with any language, any OS, any technology** | Hỗ trợ mọi ngôn ngữ, OS, công nghệ |

**Use cases:**
- Microservices architecture
- Lift-and-shift apps từ on-premises lên AWS cloud

---

## 2. Docker Repositories

> Docker images are stored in Docker Repositories  
> *Docker images được lưu trong Docker Repositories*

**Docker Hub:**
> Public repository  
> *Repository công khai*

> Find base images for many technologies or OS (e.g., Ubuntu, MySQL, ...)  
> *Tìm base images cho nhiều công nghệ/OS*

**Amazon ECR (Elastic Container Registry):**
> Private repository  
> *Repository riêng tư*

> Public repository (Amazon ECR Public Gallery)  
> *Repository công khai*

---

## 3. ECS Overview

### Docker Containers Management on AWS

| Service | Mô tả |
|---------|--------|
| **Amazon ECS** | Amazon's own container platform |
| **Amazon EKS** | Amazon's managed Kubernetes (open source) |
| **AWS Fargate** | Amazon's own Serverless container platform |
| **Amazon ECR** | Store container images |

---

### ECS Cluster

> A Cluster is a group of services and tasks  
> *Cluster là nhóm các services và tasks*

> You MUST create a cluster first!  
> *Phải tạo cluster trước!*

---

### ECS Task

> Basic unit of work in ECS  
> *Đơn vị công việc cơ bản trong ECS*

> One or more containers per Task  
> *Một hoặc nhiều containers mỗi Task*

> Running container with settings specified by Task Definition  
> *Container đang chạy với settings từ Task Definition*

**ECS Task Architecture:**
```
Task Definition
      │
      ├── Container 1: App
      ├── Container 2: Firelens (Sidecar)
      └── Container 3: Logger (Sidecar)
              │
              ▼
         Outputs: S3, CloudWatch Logs
```

---

## 4. ECS Task Definitions

> Task definitions are metadata in JSON form to tell ECS how to run a Docker container  
> *Task definitions là metadata dạng JSON để told ECS cách chạy Docker container*

**Thông tin trong Task Definition:**

| Thông tin | Mô tả |
|----------|--------|
| **Image Name** | Tên Docker image |
| **Port Binding** | Container và Host port |
| **Memory and CPU** | RAM và CPU cần thiết |
| **Environment variables** | Biến môi trường |
| **Networking information** | Thông tin mạng |
| **IAM Role** | Role cho task |
| **Logging configuration** | CloudWatch logs |

> Up to 10 containers in a Task Definition  
> *Lên đến 10 containers trong một Task Definition*

---

## 5. ECS Launch Types

### ECS - EC2 Launch Type

> ECS tasks are launched in your EC2 instances  
> *ECS tasks được launch trong EC2 instances của bạn*

> You must provision & maintain the infrastructure (the EC2 instances)  
> *Phải tự quản lý infrastructure*

> Each EC2 Instance must run the ECS Agent to register in the ECS Cluster  
> *Mỗi EC2 Instance phải chạy ECS Agent*

> AWS takes care of starting / stopping containers  
> *AWS lo việc start/stop containers*

**ECS Agent:**
> ECS agent is installed on each of EC2(Container) Instances  
> *ECS agent được install trên mỗi EC2*

> ECS agent handles incoming requests for container deployment  
> *ECS agent xử lý requests cho container deployment*

> ECS agent handles the lifecycle of container  
> *ECS agent xử lý lifecycle của container*

---

### ECS - IAM Roles for ECS

**EC2 Instance Profile (EC2 Launch Type only):**
> Used by the ECS agent  
> *Dùng bởi ECS agent*

> Makes API calls to ECS service  
> *Gọi API đến ECS service*

> Send container logs to CloudWatch Logs  
> *Gửi logs đến CloudWatch*

> Pull Docker image from ECR  
> *Pull Docker image từ ECR*

> Reference sensitive data in Secrets Manager or SSM Parameter Store  
> *Truy cập Secrets Manager hoặc SSM Parameter Store*

**ECS Task Role:**
> Allows each task to have a specific role  
> *Mỗi task có một role riêng*

> Use different roles for the different ECS Services you run  
> *Dùng roles khác nhau cho các services khác nhau*

> Task Role is defined in the task definition  
> *Task Role được định nghĩa trong task definition*

---

### ECS - Fargate Launch Type

> ECS tasks are launched to Fargate (Managed by AWS)  
> *ECS tasks được launch trên Fargate (AWS quản lý)*

> You do not provision the infrastructure (no EC2 instances to manage)  
> *Không cần quản lý infrastructure*

> It's all Serverless!  
> *Tất cả Serverless!*

> You just create task definitions  
> *Chỉ cần tạo task definitions*

> AWS just runs ECS Tasks for you based on the CPU / RAM you need  
> *AWS chạy ECS Tasks dựa trên CPU/RAM bạn cần*

> To scale, just increase the number of tasks. Simple - no more EC2 instances  
> *Để scale, chỉ cần tăng số lượng tasks*

---

## 6. ECS Service

> Service supervise task. Its job is keep task running  
> *Service giám sát task, đảm bảo task luôn chạy*

> Launch instance to maintain scheduling strategy  
> *Launch instances để duy trì strategy*

> Expose tasks to outside world  
> *Expose tasks ra bên ngoài*

> Direct network traffic to the correct host and port  
> *Điều hướng traffic đến đúng host và port*

---

## 7. ECS Data Volumes

### Amazon ECS - Data Volumes (EFS)

> Mount EFS file systems onto ECS tasks  
> *Mount EFS file systems vào ECS tasks*

> Works for both EC2 and Fargate launch types  
> *Hoạt động cho cả EC2 và Fargate*

> Tasks running in any AZ will share the same data in the EFS file system  
> *Tasks ở mọi AZ chia sẻ cùng data*

> Fargate + EFS = Serverless persistent storage  
> *Fargate + EFS = Serverless persistent storage*

> Use cases: persistent multi-AZ shared storage for your containers  
> *Use cases: persistent storage multi-AZ cho containers*

**⚠️ Lưu ý:** Amazon S3 **cannot be mounted** as a file system.

---

### Amazon ECS - Data Volumes (Bind Mounts)

> Share data between multiple containers in the same Task Definition  
> *Chia sẻ data giữa nhiều containers trong cùng Task Definition*

**EC2 Tasks - using EC2 instance storage:**
> Data are tied to the lifecycle of the EC2 instance  
> *Data gắn liền với lifecycle của EC2 instance*

**Fargate Tasks - using ephemeral storage:**
> Data are tied to the container(s) using them  
> *Data gắn liền với containers*

> 20 GiB - 200 GiB (default 20 GiB)  
> *Ephemeral storage cho Fargate*

**Use cases:**
- Share ephemeral data between containers
- "Sidecar" container pattern (gửi metrics/logs)

---

## 8. ECS Scaling

### ECS Service Auto Scaling

> Automatically increase/decrease the desired number of ECS tasks  
> *Tự động tăng/giảm số lượng ECS tasks*

> Amazon ECS Auto Scaling uses AWS Application Auto Scaling  
> *Dùng AWS Application Auto Scaling*

**Metrics cho scaling:**

| Metric | Mô tả |
|--------|--------|
| **ECS Service Average CPU Utilization** | Scale on CPU |
| **ECS Service Average Memory Utilization** | Scale on RAM |
| **ALB Request Count Per Target** | Metric từ ALB |

**Scaling Policies:**

| Policy | Mô tả |
|--------|--------|
| **Target Tracking** | Scale dựa trên target value cho CloudWatch metric |
| **Step Scaling** | Scale dựa trên CloudWatch Alarm |
| **Scheduled Scaling** | Scale dựa trên date/time (predictable changes) |

**ECS vs EC2 Auto Scaling:**

| Loại | Scale cái gì |
|------|-------------|
| **ECS Service Auto Scaling** | Task level |
| **EC2 Auto Scaling** | EC2 instance level |

> Fargate Auto Scaling is much easier to setup (because Serverless)  
> *Fargate Auto Scaling dễ setup hơn vì Serverless*

---

### EC2 Launch Type - Auto Scaling EC2 Instances

**Auto Scaling Group Scaling:**
> Scale your ASG based on CPU Utilization  
> *Scale ASG dựa trên CPU Utilization*

> Add EC2 instances over time  
> *Thêm EC2 instances theo thời gian*

**ECS Cluster Capacity Provider:**
> Used to automatically provision and scale the infrastructure for your ECS Tasks  
> *Tự động provision và scale infrastructure cho ECS Tasks*

> Capacity Provider paired with an Auto Scaling Group  
> *Capacity Provider kết hợp với ASG*

> Add EC2 Instances when you're missing capacity (CPU, RAM...)  
> *Thêm EC2 Instances khi thiếu capacity*

---

## 9. ECS Load Balancing

### Amazon ECS - Load Balancer Integrations

| Load Balancer | Support | Recommendation |
|--------------|---------|----------------|
| **Application Load Balancer (ALB)** | ✅ Supported | ✅ Recommended for most use cases |
| **Network Load Balancer (NLB)** | ✅ Supported | For high throughput/performance, or AWS PrivateLink |
| **Classic Load Balancer (CLB)** | ✅ Supported | ❌ Not recommended (no advanced features) |

---

### Amazon ECS - Load Balancing (EC2 Launch Type)

> We get a Dynamic Host Port Mapping if you define only the container port in the task definition  
> *Dynamic Host Port Mapping khi chỉ định container port*

> The ALB finds the right port on your EC2 Instances  
> *ALB tìm port đúng trên EC2 Instances*

> You must allow on the EC2 instance's Security Group any port from the ALB's Security Group  
> *Phải allow port từ ALB SG vào EC2 SG*

---

### Amazon ECS - Load Balancing (Fargate)

> Each task has a unique private IP  
> *Mỗi task có một private IP duy nhất*

> Only define the container port (host port is not applicable)  
> *Chỉ định container port (host port không áp dụng)*

**Security Group Configuration:**
```
ALB Security Group → Allow port 80/443 from web
ECS ENI Security Group → Allow port 80 from ALB
```

---

## 10. ECS Environment Variables

**Các nguồn Environment Variables:**

| Source | Use Case |
|--------|---------|
| **Hardcoded** | URLs, non-sensitive values |
| **SSM Parameter Store** | Sensitive variables (API keys, shared configs) |
| **Secrets Manager** | Sensitive variables (DB passwords) |
| **S3** | Environment Files (bulk) |

**Flow:**
```
Task Definition
      │
      ├── SSM Parameter Store ──→ Fetch values
      ├── Secrets Manager ─────→ Fetch values
      └── S3 Bucket ──────────→ Environment File
```

---

## 11. Amazon ECR

### Amazon ECR Overview

> ECR = Elastic Container Registry  
> *Lưu trữ và quản lý Docker images trên AWS*

> Private and Public repository (Amazon ECR Public Gallery)  
> *Repository riêng tư và công khai*

> Fully integrated with ECS, backed by Amazon S3  
> *Tích hợp hoàn toàn với ECS, lưu trữ trên S3*

> Access is controlled through IAM (permission errors => policy)  
> *Access kiểm soát qua IAM*

> Supports image vulnerability scanning, versioning, image tags, image lifecycle  
> *Hỗ trợ scanning, versioning, tags, lifecycle*

---

### Amazon ECR - Using AWS CLI

**Login Command:**
```bash
aws ecr get-login-password --region region | \
  docker login --username AWS --password-stdin aws_account_id.dkr.ecr.region.amazonaws.com
```

**Push Image:**
```bash
docker push aws_account_id.dkr.ecr.region.amazonaws.com/demo:latest
```

**Pull Image:**
```bash
docker pull aws_account_id.dkr.ecr.region.amazonaws.com/demo:latest
```

> In case an EC2 instance (or you) can't pull a Docker image, check IAM permissions  
> *Nếu không pull được image, kiểm tra IAM permissions*

---

## 12. AWS Copilot

> CLI tool to build, release, and operate production-ready containerized apps  
> *CLI tool để build, release và vận hành containerized apps*

> Run your apps on AppRunner, ECS, and Fargate  
> *Chạy apps trên AppRunner, ECS và Fargate*

> Helps you focus on building apps rather than setting up infrastructure  
> *Tập trung vào building apps thay vì setup infrastructure*

> Provisions all required infrastructure for containerized apps (ECS, VPC, ELB, ECR...)  
> *Tự động tạo infrastructure cần thiết*

> Automated deployments with one command using CodePipeline  
> *Deploy tự động với một command*

> Deploy to multiple environments  
> *Deploy đến nhiều environments*

> Troubleshooting, logs, health status...  
> *Troubleshooting, logs, health status*

---

## 13. Amazon EKS Overview

### Amazon EKS Overview

> Amazon EKS = Amazon Elastic Kubernetes Service  
> *Dịch vụ managed Kubernetes trên AWS*

> It is a way to launch managed Kubernetes clusters on AWS  
> *Cách launch managed Kubernetes clusters trên AWS*

> Kubernetes is an open-source system for automatic deployment, scaling and management of containerized (usually Docker) application  
> *Kubernetes là open-source system cho deployment, scaling và management của containerized applications*

> It's an alternative to ECS, similar goal but different API  
> *Là alternative cho ECS, cùng mục tiêu nhưng API khác nhau*

> EKS supports EC2 if you want to deploy worker nodes or Fargate to deploy serverless containers  
> *Hỗ trợ EC2 worker nodes hoặc Fargate*

> Use case: if your company is already using Kubernetes on-premises or in another cloud, and wants to migrate to AWS using Kubernetes  
> *Use case: công ty đã dùng Kubernetes, muốn migrate sang AWS*

> Kubernetes is cloud-agnostic (can be used in any cloud - Azure, GCP...)  
> *Kubernetes có thể dùng trên bất kỳ cloud nào*

> For multiple regions, deploy one EKS cluster per region  
> *Mỗi region cần một EKS cluster*

> Collect logs and metrics using CloudWatch Container Insights  
> *Thu thập logs và metrics bằng CloudWatch Container Insights*

---

### Amazon EKS - Node Types

| Node Type | Mô tả |
|----------|--------|
| **Managed Node Groups** | AWS tạo và quản lý EC2 instances, part of ASG |
| **Self-Managed Nodes** | Bạn tự tạo và register vào cluster |
| **AWS Fargate** | Không có nodes để quản lý |

**Managed Node Groups:**
> Creates and manages Nodes (EC2 instances) for you  
> *AWS tạo và quản lý nodes*

> Nodes are part of an ASG managed by EKS  
> *Nodes nằm trong ASG được quản lý bởi EKS*

> Supports On-Demand or Spot Instances  
> *Hỗ trợ On-Demand và Spot*

**Self-Managed Nodes:**
> Nodes created by you and registered to the EKS cluster  
> *Bạn tự tạo nodes*

> You can use prebuilt AMI - Amazon EKS Optimized AMI  
> *Dùng Amazon EKS Optimized AMI*

---

### Amazon EKS - Data Volumes

> Need to specify Storage Class manifest on your EKS cluster  
> *Cần specify Storage Class manifest*

> Leverages a Container Storage Interface (CSI) compliant driver  
> *Dùng CSI-compliant driver*

**Hỗ trợ:**

| Storage | CSI Driver |
|---------|-----------|
| **Amazon EBS** | ✅ |
| **Amazon EFS** | ✅ (works with Fargate) |
| **Amazon FSx for Lustre** | ✅ |
| **Amazon FSx for NetApp ONTAP** | ✅ |

---

## 14. Tổng kết

### Kiến thức trọng tâm cho DVA Exam

| Chủ đề | Trọng tâm |
|--------|-----------|
| **Docker** | Container concept, benefits |
| **ECS** | Cluster, Task, Task Definition, Service |
| **ECS Launch Types** | EC2 vs Fargate (Serverless) |
| **ECS IAM** | Instance Profile vs Task Role |
| **ECS Data Volumes** | EFS, Bind Mounts |
| **ECS Scaling** | Service Auto Scaling, Capacity Provider |
| **ECS Load Balancing** | ALB recommended, Dynamic Port Mapping |
| **ECS Environment** | SSM, Secrets Manager, S3 |
| **ECR** | Push/Pull images, IAM permissions |
| **EKS** | Managed Kubernetes, Node types |

### So sánh ECS vs EKS

| Feature | ECS | EKS |
|--------|-----|-----|
| **Vendor** | AWS proprietary | Open-source Kubernetes |
| **Learning curve** | Lower | Higher (if already know K8s) |
| **Portability** | AWS only | Cloud-agnostic |
| **Use case** | AWS-native apps | Multi-cloud/Hybrid |

### So sánh EC2 vs Fargate Launch Type

| Feature | EC2 Launch Type | Fargate Launch Type |
|---------|-----------------|---------------------|
| **Infrastructure** | Bạn quản lý EC2 | AWS quản lý |
| **Cost** | Chỉ trả EC2 | Trả theo vCPU/RAM |
| **Scaling** | Phức tạp hơn | Đơn giản hơn |
| **Control** | Full control | Limited |
| **Use case** | Persistent workloads, custom | Serverless, microservices |

### Best Practices 2026

**ECS:**
1. ✅ Dùng Fargate cho Serverless (đơn giản hóa operations)
2. ✅ Dùng Task Role thay vì Instance Profile khi có thể
3. ✅ Dùng ALB với Dynamic Port Mapping
4. ✅ Mount EFS cho persistent shared storage
5. ✅ Dùng Secrets Manager cho sensitive data

**ECR:**
1. ✅ Scan images cho vulnerabilities
2. ✅ Dùng lifecycle policies để clean up old images
3. ✅ Tag images appropriately (latest, v1, v2)
4. ✅ IAM permissions đúng để pull images

**Deployment:**
1. ✅ Rolling updates với additional batches cho production
2. ✅ Immutable deployments cho zero-downtime
3. ✅ Blue/Green deployments cho testing

---

## Liên kết tham khảo

- [Amazon ECS Documentation](https://docs.aws.amazon.com/ecs/)
- [Amazon ECR Documentation](https://docs.aws.amazon.com/ecr/)
- [AWS Fargate Documentation](https://docs.aws.amazon.com/AmazonECS/latest/userguide/what-is-fargate.html)
- [Amazon EKS Documentation](https://docs.aws.amazon.com/eks/)

---

*Document này được tạo để hỗ trợ học tập cho kỳ thi AWS Certified Developer - Associate*
