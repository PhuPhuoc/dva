# Section 1: Getting Started with AWS, IAM & AWS CLI, EC2 Fundamentals

**AWS Certified Developer - Associate**  
*Nguồn: Cloud Mentor Pro - Cập nhật: 15/02/2026*

---

## Mục lục

1. [Getting Started with AWS](#1-getting-started-with-aws)
2. [IAM & AWS CLI](#2-iam--aws-cli)
3. [EC2 Fundamentals](#3-ec2-fundamentals)
4. [Tổng kết](#4-tổng-kết)

---

## 1. Getting Started with AWS

### AWS Global Infrastructure (Hạ tầng Toàn cầu của AWS)

AWS là nhà cung cấp cloud hàng đầu thế giới, với hạ tầng phân tán toàn cầu bao gồm:

- **AWS Regions** (Vùng AWS)
- **AWS Availability Zones** (Vùng Khả dụng)
- **AWS Data Centers** (Trung tâm Dữ liệu)
- **AWS Edge Locations / Points of Presence** (Điểm Hiện diện Biên)

---

### AWS Regions (Vùng AWS)

> A region is a cluster of data centers  
> *Một Region là một cụm các trung tâm dữ liệu*

**Cách chọn AWS Region:**

| Tiêu chí | Giải thích |
|----------|------------|
| **Compliance with data governance and legal requirements** (Tuân thủ quản trị dữ liệu và yêu cầu pháp lý) | Dữ liệu không bao giờ rời khỏi region mà không có sự cho phép rõ ràng của bạn |
| **Proximity to customers** (Độ gần với khách hàng) | Giảm độ trễ (latency) |
| **Available services within a Region** (Dịch vụ khả dụng trong Region) | Dịch vụ mới không có ở mọi Region |
| **Pricing** (Giá cả) | Giá thay đổi theo từng region |

---

### AWS Availability Zones (Vùng Khả dụng)

> Each region has many availability zones (usually 3, min is 3, max is 6). Example:  
> ap-southeast-2a, ap-southeast-2b, ap-southeast-2c  
> *Mỗi region có nhiều Availability Zones (thường là 3, tối thiểu 3, tối đa 6)*

**Đặc điểm của Availability Zones (AZ):**

- Each availability zone (AZ) is one or more discrete data centers with redundant power, networking, connectivity  
  *Mỗi AZ là một hoặc nhiều trung tâm dữ liệu độc lập với nguồn điện dự phòng, kết nối mạng dự phòng*
- They're separate from each other, so that they're isolated from disasters  
  *Chúng tách biệt với nhau để cô lập khỏi các thảm họa*
- They're connected with high bandwidth, ultra-low latency networking  
  *Chúng kết nối với nhau qua mạng băng thông cao, độ trễ cực thấp*

**💡 Ghi chú thực tế:** Khi triển khai ứng dụng production, luôn deploy across multiple AZs (triển khai trên nhiều AZ) để đảm bảo high availability (khả dụng cao).

---

### AWS Edge Locations (Points of Presence) - Điểm Hiện diện Biên

> The global edge network currently including 400+ Edge Locations, and 13 Regional Caches in 90+ cities across 48+ countries  
> *Mạng lưới edge toàn cầu hiện bao gồm 400+ Edge Locations và 13 Regional Caches tại 90+ thành phố thuộc 48+ quốc gia*

- Content is delivered to end users with lower latency  
  *Nội dung được phân phối đến người dùng cuối với độ trễ thấp hơn*

---

## 2. IAM & AWS CLI

### AWS Identity and Access Management (AWS IAM)

---

### IAM: Users & Groups (Người dùng & Nhóm)

> IAM = Identity and Access Management, Global service  
> *IAM = Quản lý Định danh và Truy cập, là dịch vụ Toàn cầu*

| Khái niệm | Mô tả |
|-----------|-------|
| **Root account** (Tài khoản Root) | Được tạo mặc định, không nên sử dụng hoặc chia sẻ |
| **Users** (Người dùng) | Là con người trong tổ chức của bạn |
| **Groups** (Nhóm) | Chỉ chứa users, không chứa groups khác |
| **Users** | Có thể không thuộc group nào, hoặc thuộc nhiều groups |

---

### IAM: Permissions (Quyền hạn)

> Users or Groups can be assigned JSON documents called policies  
> *Users hoặc Groups có thể được gán các tài liệu JSON được gọi là policies*

> These policies define the permissions of the users  
> *Các policies này định nghĩa quyền hạn của users*

> In AWS you apply the least privilege principle: don't give more permissions than a user needs  
> *Trong AWS, bạn áp dụng nguyên tắc least privilege (đặc quyền tối thiểu): không cấp nhiều quyền hơn mức user cần*

**💡 Nguyên tắc Least Privilege (Đặc quyền Tối thiểu):** Đây là nguyên tắc bảo mật quan trọng nhất trong IAM. Chỉ cấp exactly những permissions cần thiết cho task.

---

### IAM Policies Inheritance (Kế thừa Policy)

Khi user thuộc nhiều groups, permissions được kế thừa từ tất cả groups mà user đó tham gia.

---

### IAM Policies Structure (Cấu trúc Policy)

> Consists of:  
> *Policy bao gồm:*

| Thành phần | Bắt buộc? | Mô tả |
|------------|----------|-------|
| **Version** | Có | Policy language version (Phiên bản ngôn ngữ policy) |
| **Id** | Không | Identifier for the policy (Định danh policy) |
| **Statement** | **Có** | Một hoặc nhiều statements (bắt buộc) |

**Statement Structure (Cấu trúc Statement):**

| Thành phần | Mô tả |
|------------|-------|
| **Sid** | Identifier for the statement (Định danh statement) |
| **Effect** | Allow hoặc Deny |
| **Principal** | Account/user/role được áp dụng |
| **Action** | Danh sách actions được phép hoặc từ chối |
| **Resource** | Danh sách resources mà actions được áp dụng |
| **Condition** | Điều kiện khi policy có hiệu lực |

**Ví dụ Policy đơn giản:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3Read",
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:ListBucket"],
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}
```

---

### IAM - Password Policy (Chính sách Mật khẩu)

> Strong passwords = higher security  
> *Mật khẩu mạnh = bảo mật cao hơn*

**Trong AWS, bạn có thể setup password policy với các tùy chọn:**

- Set a minimum password length (Đặt độ dài tối thiểu)
- Require specific character types (Yêu cầu các loại ký tự cụ thể)
- Allow all IAM users to change their own passwords (Cho phép users tự đổi mật khẩu)
- Require users to change their password after some time (Yêu cầu đổi mật khẩu theo chu kỳ)
- Prevent password re-use (Ngăn chặn sử dụng lại mật khẩu cũ)

**💡 Khuyến nghị DVA:** Password policy nên yêu cầu tối thiểu 14 ký tự, bao gồm uppercase, lowercase, numbers, và special characters.

---

### Multi Factor Authentication - MFA (Xác thực Đa yếu tố)

> Protect your Root Accounts and IAM users  
> *Bảo vệ Root Accounts và IAM users*

> MFA = password you know + security device you own  
> *MFA = mật khẩu bạn biết + thiết bị bảo mật bạn sở hữu*

**MFA devices options in AWS (Các tùy chọn thiết bị MFA):**

| Loại thiết bị | Ví dụ |
|--------------|-------|
| **Virtual MFA device** (Thiết bị MFA ảo) | Google Authenticator (điện thoại), Authy |
| **Universal 2nd Factor (U2F) Security Key** | YubiKey by Yubico (thiết bị vật lý thế hệ 3) |

**💡 Quan trọng cho DVA:** Root account **bắt buộc** phải enable MFA. Đây là best practice đầu tiên khi tạo AWS account mới.

---

### How can users access AWS? (Cách người dùng truy cập AWS?)

> To access AWS, you have three options:  
> *Để truy cập AWS, bạn có 3 tùy chọn:*

| Phương thức | Bảo mật | Mục đích sử dụng |
|-------------|---------|------------------|
| **AWS Management Console** | Password + MFA | Quản lý qua giao diện web |
| **AWS Command Line Interface (CLI)** | Access Keys | Tự động hóa, script |
| **AWS Software Developer Kit (SDK)** | Access Keys | Lập trình, tích hợp ứng dụng |

**Về Access Keys (Khóa Truy cập):**
- Access Keys are generated through the AWS Console  
  *Access Keys được tạo qua AWS Console*
- Access Keys are secret, just like a password. Don't share them  
  *Access Keys là bí mật, giống như mật khẩu. Không chia sẻ*
- Access Key ID ~= username  
- Secret Access Key ~= password

---

### What's the AWS CLI? (AWS CLI là gì?)

> A tool that enables you to interact with AWS services using commands in your command-line shell  
> *Tool cho phép bạn tương tác với AWS services bằng các lệnh trong command-line shell*

- Direct access to the public APIs of AWS services  
  *Truy cập trực tiếp đến public APIs của AWS services*
- It's open-source  
  *Là mã nguồn mở*
- Repository: https://github.com/aws/aws-cli

**Cài đặt AWS CLI:**
```bash
# macOS/Linux
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
sudo installer -pkg AWSCLIV2.pkg -target /

# Windows (via MSI installer)
# Hoặc sử dụng pip:
pip install awscli

# Verify
aws --version
```

**Cấu hình AWS CLI:**
```bash
aws configure
# AWS Access Key ID: [your-access-key]
# AWS Secret Access Key: [your-secret-key]
# Default region name: ap-southeast-1
# Default output format: json
```

**💡 Lệnh CLI quan trọng cho DVA:**
```bash
aws iam list-users                    # Liệt kê users
aws ec2 describe-instances             # Mô tả instances
aws s3 ls                             # Liệt kê S3 buckets
aws lambda list-functions             # Liệt kê Lambda functions
```

---

### AWS SDK

> AWS Software Development Kit (AWS SDK)  
> *Bộ công cụ phát triển phần mềm AWS*

> Language-specific APIs (set of libraries)  
> *APIs theo ngôn ngữ (tập hợp thư viện)*

> Enables you to access and manage AWS services programmatically  
> *Cho phép truy cập và quản lý AWS services bằng lập trình*

> Embedded within your application  
> *Được nhúng trong ứng dụng của bạn*

**Supported SDKs (SDKs được hỗ trợ):**

| Ngôn ngữ | Package |
|----------|---------|
| JavaScript | `@aws-sdk/client-*` |
| Python | `boto3` |
| PHP | `aws/aws-sdk-php` |
| .NET | `AWSSDK.*` |
| Ruby | `aws-sdk` |
| Java | `aws-java-sdk` |
| Go | `github.com/aws/aws-sdk-go` |
| C++ | `aws-sdk-cpp` |

**💡 Lưu ý quan trọng DVA 2026:** AWS CLI được build trên AWS SDK for Python (boto3). Hiểu cách sử dụng boto3 là kỹ năng thiết yếu cho developer.

---

### IAM Roles for Services (IAM Roles cho Services)

> Some AWS service will need to perform actions on your behalf  
> *Một số AWS services cần thực hiện actions thay mặt bạn*

> To do so, we will assign permissions to AWS services with IAM Roles  
> *Để làm điều này, chúng ta gán permissions cho AWS services bằng IAM Roles*

**Common roles (Roles phổ biến):**

| Role | Mô tả |
|------|-------|
| **EC2 Instance Roles** | Gán cho EC2 instance để access AWS resources |
| **Lambda Function Roles** | Gán cho Lambda function |
| **Roles for CloudFormation** | Gán cho CloudFormation stack |

**💡 Khác biệt quan trọng:**
- **User/Group Policy**: Gán trực tiếp cho người hoặc nhóm
- **Role**: Gán cho AWS service để asssume (nhận) quyền tạm thời

---

### IAM Security Tools (Công cụ Bảo mật IAM)

| Tool | Phạm vi | Mô tả |
|------|---------|-------|
| **IAM Credentials Report** | Account-level | Liệt kê tất cả users và status của credentials |
| **IAM Access Advisor** | User-level | Hiển thị permissions đã cấp và thời điểm truy cập cuối |

> Access advisor shows the service permissions granted to a user and when those services were last accessed  
> *Access advisor hiển thị service permissions đã cấp cho user và khi nào services được truy cập lần cuối*

> You can use this information to revise your policies  
> *Bạn có thể dùng thông tin này để điều chỉnh policies*

**💡 Sử dụng cho Audit:** Chạy Credentials Report định kỳ để phát hiện unused credentials và Access Advisor để clean up permissions không cần thiết.

---

### IAM Guidelines & Best Practices (Hướng dẫn & Best Practices)

| # | Best Practice | Giải thích |
|---|---------------|-------------|
| 1 | Don't use the root account except for AWS account setup | Chỉ dùng root cho setup ban đầu |
| 2 | One physical user = One AWS user | Mỗi người thật = 1 IAM user |
| 3 | Assign users to groups and assign permissions to groups | Dùng groups thay vì gán trực tiếp |
| 4 | Create a strong password policy | Policy mật khẩu mạnh |
| 5 | Use and enforce Multi Factor Authentication (MFA) | Bắt buộc MFA |
| 6 | Create and use Roles for giving permissions to AWS services | Dùng Roles cho services |
| 7 | Use Access Keys for Programmatic Access (CLI / SDK) | Access Keys cho automation |
| 8 | Audit permissions using IAM Credentials Report & IAM Access Advisor | Audit định kỳ |
| 9 | Never share IAM users & Access Keys | Không chia sẻ credentials |

---

### Shared Responsibility Model for IAM (Mô hình Trách nhiệm Chia sẻ)

**AWS chịu trách nhiệm:**
- Infrastructure (global network security) - Hạ tầng, bảo mật mạng toàn cầu
- Configuration and vulnerability analysis - Cấu hình và phân tích lỗ hổng
- Compliance validation - Xác thực tuân thủ

**Bạn chịu trách nhiệm:**
- Users, Groups, Roles, Policies management and monitoring
- Enable MFA on all accounts
- Rotate all your keys often
- Use IAM tools to apply appropriate permissions
- Analyze access patterns & review permissions

---

### IAM Section - Summary (Tóm tắt IAM)

| Khái niệm | Mô tả |
|-----------|-------|
| **Users** | Ánh xạ đến người dùng thật, có password cho AWS Console |
| **Groups** | Chỉ chứa users |
| **Policies** | Tài liệu JSON định nghĩa permissions |
| **Roles** | Dùng cho EC2 instances hoặc AWS services |
| **Security** | MFA + Password Policy |
| **AWS CLI** | Quản lý AWS services bằng command-line |
| **AWS SDK** | Quản lý AWS services bằng ngôn ngữ lập trình |
| **Access Keys** | Truy cập AWS qua CLI hoặc SDK |
| **Audit** | IAM Credential Reports & IAM Access Advisor |

---

## 3. EC2 Fundamentals

### Amazon EC2 - Basics (Cơ bản)

> EC2 is one of the most popular of AWS' offering  
> *EC2 là một trong những offering phổ biến nhất của AWS*

> EC2 = Elastic Compute Cloud = Infrastructure as a Service  
> *EC2 = Đám mây Điện toán Đàn hồi = Hạ tầng như một Dịch vụ*

**EC2 chủ yếu bao gồm khả năng:**

| Thành phần | Mô tả |
|------------|-------|
| **Renting virtual machines (EC2)** | Thuê máy ảo |
| **Storing data on virtual drives (EBS)** | Lưu trữ dữ liệu trên ổ đĩa ảo |
| **Distributing load across machines (ELB)** | Phân phối tải giữa các máy |
| **Scaling the services using an auto-scaling group (ASG)** | Mở rộng dịch vụ bằng auto-scaling group |

> Knowing EC2 is fundamental to understand how the Cloud works  
> *Hiểu EC2 là nền tảng để hiểu Cloud hoạt động như thế nào*

---

### EC2 - Sizing & Configuration Options (Tùy chọn Cấu hình)

| Tùy chọn | Mô tả |
|----------|-------|
| **Operating System (OS)** | Linux, Windows hoặc Mac OS |
| **Compute power & cores (CPU)** | Số vCPU và loại processor |
| **Random-access memory (RAM)** | Dung lượng RAM |
| **Storage space** | Network-attached (EBS & EFS) hoặc hardware (EC2 Instance Store) |
| **Network card** | Tốc độ card mạng, Public IP address |
| **Firewall rules** | Security Group |
| **Bootstrap script** | EC2 User Data |

---

### EC2 - User Data

> It is possible to bootstrap our instances using an EC2 User data script  
> *Có thể bootstrap instances bằng EC2 User Data script*

> Bootstrapping means launching commands when a machine starts  
> *Bootstrapping nghĩa là chạy commands khi máy khởi động*

> That script is only run once at the instance first start  
> *Script chỉ chạy một lần khi instance khởi động lần đầu*

**EC2 User Data được sử dụng để tự động hóa:**

- Installing updates - Cài cập nhật
- Installing software - Cài phần mềm
- Downloading common files from the internet - Tải files
- Anything you can think of - Bất cứ gì bạn cần

> The EC2 User Data Script runs with the root user  
> *EC2 User Data Script chạy với quyền root user*

**Ví dụ User Data script (Linux):**
```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "Hello from EC2" > /var/www/html/index.html
```

**Ví dụ User Data script (Windows):**
```powershell
<powershell>
Install-WindowsFeature -name Web-Server
Set-Content -Path C:\inetpub\wwwroot\index.html -Value "Hello from EC2"
</powershell>
```

**💡 DVA Exam Tip:** User Data có thể được truy cập bởi root/admin user trên instance. Không bao giờ đặt sensitive data (Access Keys, passwords) trong User Data script!

---

### Hands-On: Launching an EC2 Instance running Linux

Quy trình thực hành:
1. Chọn Amazon Machine Image (AMI)
2. Chọn Instance Type
3. Configure Instance Details (User Data nếu cần)
4. Add Storage
5. Add Tags
6. Configure Security Group
7. Review and Launch
8. Select Key Pair

---

### EC2 - Instance Types

> You can use different types of EC2 instances that are optimised for different use cases  
> *Có thể dùng các loại instance khác nhau được tối ưu cho các use cases khác nhau*

**AWS Instance Naming Convention (Quy ước đặt tên):**

```
m5.2xlarge
│││└── Size (kích thước)
││└── Generation (thế hệ - AWS cải thiện theo thời gian)
│└── Instance class (loại instance)
└── Metric (đơn vị)
```

| Prefix | Ý nghĩa |
|--------|---------|
| **m** | General Purpose |
| **c** | Compute Optimized |
| **r** | Memory Optimized |
| **i** | Storage Optimized |
| **t** | Burstable Performance |

---

### EC2 Instance Types - General Purpose (Loại Đa năng)

> General purpose instances provide a balance of Compute, Memory, Networking  
> *Instances đa năng cung cấp cân bằng giữa Compute, Memory, Networking*

**Great for (Phù hợp cho):**
- Web servers
- Code repositories

**Các instance phổ biến:**
- **T3/T3a**: Burstable, giá rẻ
- **M5/M5a**: Balanced, production-grade

---

### EC2 Instance Types - Compute Optimized (Loại Tối ưu Compute)

> Ideal for compute bound applications that benefit from high performance processors  
> *Lý tưởng cho các ứng dụng giới hạn bởi compute, được hưởng lợi từ processors hiệu năng cao*

**Great for (Phù hợp cho):**
- Compute intensive applications
- Batch processing workloads
- Media transcoding
- High performance web servers, computing (HPC)
- Scientific modeling, machine learning
- Dedicated gaming servers and ad server engines

**Các instance phổ biến:**
- **C5/C5a**: Compute optimized
- **C6i**: Intel-based compute

---

### EC2 Instance Types - Memory Optimized (Loại Tối ưu Memory)

> Fast performance for workloads that process large data sets in memory  
> *Hiệu năng cao cho workload xử lý large datasets trong memory*

**Great for (Phù hợp cho):**
- Relational/non-relational databases
- Distributed web scale cache stores (ví dụ: Redis, Memcached)
- In-memory databases optimized for BI (business intelligence)
- Applications performing real-time processing of big unstructured data

**Các instance phổ biến:**
- **R5/R5a**: Memory optimized
- **X1/X1e**: High memory for SAP, Oracle

---

### EC2 Instance Types - Storage Optimized (Loại Tối ưu Storage)

> High, sequential read and write access to very large data sets on local storage  
> *Truy cập đọc/ghi tuần tự cao đến very large datasets trên local storage*

**Great for (Phù hợp cho):**
- High frequency online transaction processing (OLTP) systems
- Relational & NoSQL databases
- Cache for in-memory databases (Redis)
- Data warehousing applications
- Distributed file systems

**Các instance phổ biến:**
- **I3/I3en**: High I/O
- **D2**: Dense storage

---

### EC2 - Security Groups (Nhóm Bảo mật)

> Control how traffic is allowed into or out of our EC2 (firewall)  
> *Kiểm soát cách traffic được phép vào hoặc ra khỏi EC2 (firewall)*

**Đặc điểm quan trọng:**

| Đặc điểm | Mô tả |
|----------|-------|
| **Can be attached to multiple instances** | Có thể gán cho nhiều instances |
| **All inbound traffic is blocked by default** | Mặc định block toàn bộ inbound |
| **All outbound traffic is authorized by default** | Mặc định cho phép outbound |
| **Security groups only contain allow rules** | Chỉ chứa rules cho phép |
| **Rules can reference by IP or by security group** | Rules có thể tham chiếu IP hoặc security group khác |

**💡 Security Groups là Stateful:**
- Nếu bạn cho phép inbound port 80, outbound response tự động được cho phép
- Khác với Network ACL (Stateless - phải cấu hình cả inbound và outbound)

**Ví dụ Security Group Rules:**
```
Inbound:
- 0.0.0.0/0  -> 80  (HTTP)
- 0.0.0.0/0  -> 443 (HTTPS)
- Your IP    -> 22  (SSH)

Outbound:
- 0.0.0.0/0  -> All (Mặc định)
```

---

### Classic Ports to Know (Các Port Quan trọng)

| Port | Protocol | Mô tả |
|------|---------|-------|
| **22** | SSH (Secure Shell) | Đăng nhập Linux instance |
| **21** | FTP (File Transfer Protocol) | Upload files (không bảo mật) |
| **22** | SFTP (Secure File Transfer Protocol) | Upload files dùng SSH |
| **80** | HTTP | Truy cập website không bảo mật |
| **443** | HTTPS | Truy cập website bảo mật |
| **3389** | RDP (Remote Desktop Protocol) | Đăng nhập Windows instance |

---

### EC2 - Connect to your EC2 instance (Kết nối EC2)

**Các cách kết nối:**

| Phương thức | Mô tả |
|-------------|-------|
| **SSH** | Terminal qua Linux/Mac/Windows (PowerShell/git bash) |
| **Session Manager** | Qua AWS Systems Manager, không cần port 22 open |
| **EC2 Instance Connect** | Kết nối trực tiếp từ browser |

---

### EC2 Instance Connect

> Connect to your EC2 instance within your browser  
> *Kết nối đến EC2 instance ngay trong trình duyệt*

> No need to use your key file  
> *Không cần dùng key file*

> Need to make sure the port 22 is still opened!  
> *Phải đảm bảo port 22 vẫn mở*

> Works only Amazon Linux or Ubuntu  
> *Chỉ hoạt động với Amazon Linux hoặc Ubuntu*

---

### EC2 Instances Purchasing Options (Tùy chọn Mua sắm)

| Tùy chọn | Mô tả | Discount |
|----------|-------|----------|
| **On-Demand Instances** | Ngắn hạn, trả theo giây | 0% |
| **Reserved Instances (1 & 3 years)** | Dài hạn, cam kết | Đến 72% |
| **Convertible Reserved Instances** | Linh hoạt về instance type | Đến 66% |
| **Savings Plans (1 & 3 years)** | Cam kết usage/hour cố định | Đến 72% |
| **Spot Instances** | Ngắn hạn, có thể mất instance | Đến 90% |
| **Dedicated Hosts** | Máy vật lý riêng | 0% |
| **Dedicated Instances** | Hardware riêng trong shared host | 0% |
| **Capacity Reservations** | Reserve capacity trong AZ | 0% |

---

### EC2 On Demand

> Has the highest cost but no upfront payment  
> *Chi phí cao nhất nhưng không trả trước*

> Short-term and un-interrupted workloads  
> *Workload ngắn hạn và không bị gián đoạn*

> Pay for what you use  
> *Trả cho những gì sử dụng*

> No long-term commitment  
> *Không cam kết dài hạn*

**Use cases:**
- Dev/stg/test environment
- Critical batch job
- System that runs only in business hours (ví dụ: 8am - 5pm)
- System that scales frequently

---

### EC2 Reserved Instances

> Up to 72% discount compared to On-demand  
> *Giảm đến 72% so với On-Demand*

**Đặc điểm:**
- Reserve instance configuration (Type, Region, Tenancy, OS)
- Reservation Period: 1 year (+discount) hoặc 3 years (+++discount)
- Payment Options: No Upfront (+), Partial Upfront (++), All Upfront (+++)
- Reserved Instance's Scope: Regional hoặc Zonal

> Recommended for steady-state usage applications (think database)  
> *Khuyến nghị cho ứng dụng usage ổn định (ví dụ: database)*

**Convertible Reserved Instances:**
- Can change the type, family, OS...
- Up to 66% discount

**Use cases:**
- System that runs 24/7 for a long time
- Workload does not change for a long time

---

### EC2 Savings Plans

> Commit to a consistent amount of usage, in USD per hour ($10/hour for 1 or 3 years)  
> *Cam kết một lượng usage nhất quán, theo USD/giờ*

> Up to 72% discount compared to On-demand  
> *Giảm đến 72% so với On-Demand*

**Đặc điểm:**
- Locked to a specific instance family & AWS region (ví dụ: M5 in us-east-1)
- Flexible across:
  - Instance Size (m5.xlarge, m5.2xlarge)
  - OS (Linux, Windows)
  - Tenancy (Host, Dedicated, Default)

**Use cases:**
- Giống Reserved Instances
- System that runs 24/7 for a long time
- Workload cần thay đổi INSTANCE TYPE trong dài hạn

**💡 So sánh Reserved vs Savings Plans:**
- Reserved: Lock vào specific instance type
- Savings Plans: Linh hoạt hơn, có thể chuyển đổi instance family

---

### EC2 Spot Instances

> Discount of up to 90% compared to On-demand  
> *Giảm đến 90% so với On-Demand*

> If the current spot price > your max price you can choose to stop or terminate your instance within a 2 minutes grace period.  
> *Nếu spot price hiện tại > max price của bạn, bạn có thể chọn stop hoặc terminate instance trong 2 phút grace period*

> The MOST cost-efficient instances in AWS  
> *Instance tiết kiệm chi phí nhất trong AWS*

> Spot Block: "block" spot instance during 1 to 6 hours without interruptions  
> *Spot Block: giữ spot instance trong 1-6 giờ mà không bị gián đoạn*

**Useful for workloads that can be interrupted:**
- Batch jobs
- Data analysis
- Workloads with a flexible start and end time

**⚠️ Cảnh báo:** Spot Instances có thể bị terminated bất cứ lúc nào khi EC2 cần capacity. Không dùng cho production workloads quan trọng mà không có proper handling.

---

### EC2 Dedicated Hosts

> A physical server with EC2 instance capacity fully dedicated to your use  
> *Server vật lý với EC2 instance capacity hoàn toàn dành riêng cho bạn*

> Use your existing server-bound software licenses (per-socket, per-core, per-VM software licenses)  
> *Sử dụng software licenses hiện có của bạn*

**Use case:**
- Compliance requirements (yêu cầu tuân thủ)
- On-premises licenses (licenses BYOL - Bring Your Own License)

---

### EC2 Dedicated Instances

> Instances run on hardware that's dedicated to you  
> *Instances chạy trên hardware dành riêng cho bạn*

> May share hardware with other instances in same account  
> *Có thể share hardware với instances khác trong cùng account*

> No control over instance placement (can move hardware after Stop/Start)  
> *Không kiểm soát được instance placement*

**Use case:**
> System that requires strict policy which needs to run in an isolated environment (ví dụ: Bank system, Hospital system)  
> *Hệ thống yêu cầu policy nghiêm ngặt, cần chạy trong môi trường cô lập*

---

### Understanding AWS Tenancy (Hiểu về AWS Tenancy)

| Tenancy | Mô tả |
|---------|-------|
| **Shared** | Shared hardware, mặc định |
| **Dedicated Instance** | Dedicated hardware trong shared host |
| **Dedicated Host** | Toàn bộ physical server riêng |

---

### EC2 Capacity Reservations

> Reserve On-Demand instances capacity in a specific AZ for any duration  
> *Reserve On-Demand capacity trong một AZ cụ thể*

> You always have access to EC2 capacity when you need it  
> *Bạn luôn có quyền truy cập EC2 capacity khi cần*

> No time commitment (create/cancel anytime), no billing discounts  
> *Không cam kết thời gian, không giảm giá*

> You're charged at On-Demand rate whether you run instances or not  
> *Bị tính phí On-Demand rate dù có chạy instance hay không*

**Use case:**
> Suitable for short-term, uninterrupted workloads that needs to be in a specific AZ  
> *Phù hợp cho workload ngắn hạn, không bị gián đoạn, cần ở một AZ cụ thể*

---

### Which purchasing option is right for me? (Tùy chọn nào phù hợp?)

**Analogy (So sánh) - Hotel booking (Đặt khách sạn):**

| Tùy chọn | So sánh Hotel |
|----------|---------------|
| **On Demand** | Đến và ở bất cứ khi nào, trả giá đầy đủ |
| **Reserved** | Đặt trước, ở lâu = được giảm giá tốt |
| **Savings Plans** | Đặt tiêu chuẩn/h giờ, ở bất kỳ loại phòng nào |
| **Spot instances** | Đấu giá phòng trống, có thể bị đuổi |
| **Dedicated Hosts** | Đặt cả tòa nhà |
| **Capacity Reservations** | Đặt phòng trước, dù có ở hay không vẫn trả tiền |

---

### AWS charges for IPv4 addresses (AWS tính phí IPv4 công khai)

> Starting February 1st 2024, there's a charge for all Public IPv4 created in your account  
> *Từ 01/02/2024, có phí cho tất cả Public IPv4 được tạo trong account*

> $0.005 per hour of Public IPv4 (~$3.6 per month)  
> *$0.005 mỗi giờ cho Public IPv4 (~$3.6 mỗi tháng)*

**Free Tier cho Public IPv4:**
| Loại | Free Tier |
|------|-----------|
| Public IP trên EC2 | 750 giờ/tháng trong 12 tháng đầu (cho new accounts) |
| Public IP trên Load Balancer | 1 IP/AZ, không có free tier |
| Public IP trên RDS | Không có free tier |

**💡 Tối ưu chi phí 2026:**
- Sử dụng private IP thay vì public IP khi có thể
- Sử dụng Elastic IP (EIP) chỉ khi cần thiết
- Xóa unused public IPs để tránh phí

---

## 4. Tổng kết

### Kiến thức trọng tâm cho DVA Exam

| Chủ đề | Trọng tâm |
|--------|-----------|
| **AWS Regions & AZs** | Hiểu sự khác biệt, cách chọn Region |
| **IAM Users/Groups/Roles/Policies** | Cấu trúc JSON policy, nguyên tắc least privilege |
| **AWS CLI & SDK** | Cấu hình, sử dụng cơ bản |
| **EC2 Instance Types** | Các loại (m, c, r, i, t) và use cases |
| **Security Groups** | Stateful, default rules, reference by SG |
| **EC2 Purchasing** | On-Demand, Reserved, Spot, Dedicated |
| **User Data** | Bootstrap script |

### Best Practices cho Production (2026)

1. **IAM Security:**
   - Enable MFA cho tất cả users
   - Không dùng root account
   - Áp dụng least privilege
   - Regular audit permissions

2. **EC2:**
   - Luôn deploy across multiple AZs
   - Sử dụng IAM Roles thay vì Access Keys trong instance
   - Không đặt sensitive data trong User Data
   - Chọn đúng instance type cho workload

3. **Cost Optimization:**
   - Reserved Instances/Savings Plans cho steady workloads
   - Spot Instances cho batch jobs có thể interruption
   - Clean up unused resources

---

## Liên kết tham khảo

- [AWS IAM Documentation](https://docs.aws.amazon.com/iam/)
- [AWS EC2 Documentation](https://docs.aws.amazon.com/ec2/)
- [AWS CLI Reference](https://awscli.amazonaws.com/v2/documentation/api/latest/index.html)
- [AWS SDKs and Tools](https://aws.amazon.com/tools/)

---

*Document này được tạo để hỗ trợ học tập cho kỳ thi AWS Certified Developer - Associate*
