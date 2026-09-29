# Section 2: EC2 Instance Storage, High Availability & Scalability (ELB & ASG)

**AWS Certified Developer - Associate**  
*Nguồn: Cloud Mentor Pro - Cập nhật: 15/02/2026*

---

## Mục lục

1. [EC2 Instance Storage](#1-ec2-instance-storage)
2. [High Availability & Scalability](#2-high-availability--scalability)
3. [ELB - Elastic Load Balancer](#3-elb---elastic-load-balancer)
4. [ASG - Auto Scaling Group](#4-asg---auto-scaling-group)
5. [Tổng kết](#5-tổng-kết)

---

## 1. EC2 Instance Storage

### EBS Volume, EC2 Instance Store, Elastic File System (EFS)

---

### What's an EBS Volume? (EBS Volume là gì?)

> An EBS (Elastic Block Store) Volume is a network drive you can attach to your instances while they run  
> *EBS (Elastic Block Store) Volume là một ổ đĩa mạng mà bạn có thể gắn vào instances khi chúng đang chạy*

> It allows your instances to persist data, even after their termination  
> *Nó cho phép instances lưu trữ dữ liệu, ngay cả sau khi terminate*

> They can only be mounted to one instance at a time (except: io1/io2)  
> *Chúng chỉ có thể gắn vào một instance tại một thời điểm (ngoại trừ: io1/io2)*

> It's locked to an Availability Zone (AZ)  
> *Nó bị khóa vào một Availability Zone (AZ)*

> Analogy: Think of them as a "network USB stick"  
> *Tương tự: Hãy nghĩ chúng như một "USB stick mạng"*

**💡 Đặc điểm quan trọng:**
- EBS là network-attached storage → có độ trễ thấp hơn so với instance store nhưng chậm hơn local disk
- Data persist (tồn tại) sau khi EC2 terminate → KHÁC với instance store
- Chỉ gắn được vào 1 instance trong cùng AZ (trừ io1/io2 với EBS Multi-Attach)

---

### Root EBS Volume (Volume Root)

> By default, when you create an EC2 Instance, it will automatically attach an EBS Volume, this is called Root EBS Volume.  
> *Mặc định, khi tạo EC2 Instance, nó sẽ tự động gắn một EBS Volume, gọi là Root EBS Volume*

> This volume contains OS and system files.  
> *Volume này chứa OS và system files*

**Root EBS Volume** chứa:
- Operating System (Linux/Windows)
- System files
- Boot files

---

### EBS Volume Types (Các loại EBS Volume)

> EBS Volumes come in 6 types  
> *EBS Volumes có 6 loại*

| Loại | Mô tả | Use Case |
|------|--------|----------|
| **gp2/gp3** | General Purpose SSD | Boot volumes, dev/test, applications |
| **io1/io2 Block Express** | Provisioned IOPS SSD | Mission-critical, databases |
| **st1** | Throughput Optimized HDD | Big data, data warehouses, log processing |
| **sc1** | Cold HDD | Infrequently accessed data, lowest cost |

**⚠️ Chỉ gp2/gp3 và io1/io2 Block Express có thể dùng làm boot volumes**

---

### General Purpose SSD (gp2/gp3)

> General purpose SSD volume that balances price and performance  
> *SSD đa năng cân bằng giá và hiệu năng*

> Cost effective storage, low-latency  
> *Lưu trữ tiết kiệm chi phí, độ trễ thấp*

**gp3:**
- Baseline: 3,000 IOPS, 125 MiB/s throughput
- Có thể tăng IOPS lên 16,000 và throughput lên 1,000 MiB/s independently
- Giá rẻ hơn gp2

**gp2:**
- Small volumes có thể burst lên 3,000 IOPS
- IOPS và size liên kết (3 IOPS per GB)
- Max IOPS: 16,000

**💡 So sánh gp2 vs gp3:**
```
gp2:   Size ↔ IOPS (tỷ lệ 3:1)
gp3:   Size và IOPS độc lập (có thể tune riêng)
```

---

### Provisioned IOPS SSD (io1/io2 Block Express)

> Highest-performance SSD volume for mission-critical low-latency or high-throughput workloads  
> *SSD hiệu năng cao nhất cho workload mission-critical, độ trễ thấp hoặc throughput cao*

> Or applications that need more than 16,000 IOPS  
> *Hoặc ứng dụng cần hơn 16,000 IOPS*

> Great for databases workloads  
> *Phù hợp cho database workloads*

**io1 (4 GiB - 16 TiB):**
- Max PIOPS: 64,000 (Nitro EC2) hoặc 32,000 (khác)
- Có thể tăng PIOPS độc lập với storage size

**io2 Block Express (4 GiB - 64 TiB):**
- Sub-millisecond latency
- Max PIOPS: 256,000 (IOPS:GiB ratio = 1,000:1)
- Hỗ trợ EBS Multi-attach

---

### Hard Disk Drives (HDD)

**Throughput Optimized HDD (st1):**
> Big Data, Data Warehouses, Log Processing  
> *Big Data, Data Warehouses, Xử lý Log*

- Max throughput: 500 MiB/s
- Max IOPS: 500
- **Không dùng làm boot volume**

**Cold HDD (sc1):**
> For data that is infrequently accessed  
> *Cho dữ liệu truy cập không thường xuyên*

> Scenarios where the lowest cost is important  
> *Trường hợp cần chi phí thấp nhất*

- Max throughput: 250 MiB/s
- Max IOPS: 250

**⚠️ Lưu ý:** st1 và sc1 không thể dùng làm boot volumes!

---

### EBS Volume Types Summary (Tóm tắt các loại EBS)

| Volume Type | Size | Max IOPS | Max Throughput | Boot Volume | Multi-Attach |
|-------------|------|----------|-----------------|-------------|---------------|
| **gp3** | 1 GiB - 16 TiB | 16,000 | 1,000 MiB/s | ✅ | ❌ |
| **gp2** | 1 GiB - 16 TiB | 16,000 | 250 MiB/s | ✅ | ❌ |
| **io2 Block Express** | 4 GiB - 64 TiB | 256,000 | 4,000 MiB/s | ✅ | ✅ |
| **io1** | 4 GiB - 16 TiB | 64,000 | 1,000 MiB/s | ✅ | ✅ |
| **st1** | 125 GiB - 16 TiB | 500 | 500 MiB/s | ❌ | ❌ |
| **sc1** | 125 GiB - 16 TiB | 250 | 250 MiB/s | ❌ | ❌ |

**Độ bền (Durability):**
- gp2/gp3/io1: 99.8% - 99.9% (0.1% - 0.2% annual failure rate)
- io2 Block Express: **99.999%** (0.001% annual failure rate)

---

### EBS Multi-Attach (Chỉ io1/io2)

> Attach the same EBS volume to multiple EC2 instances in the same AZ  
> *Gắn cùng một EBS volume vào nhiều EC2 instances trong cùng AZ*

**Use cases:**
> Achieve higher application availability in clustered Linux applications (ex: Teradata)  
> *Đạt availability cao hơn trong clustered Linux applications*

> Applications must manage concurrent write operations  
> *Ứng dụng phải quản lý các thao tác ghi đồng thời*

> Up to 16 EC2 Instances at a time  
> *Lên đến 16 EC2 Instances cùng lúc*

**⚠️ Yêu cầu:** Tất cả instances phải cùng AZ và phải quản lý concurrent writes

---

### EBS Snapshots (Ảnh chụp EBS)

> Make a backup snapshot of your EBS volume at a point in time  
> *Tạo backup snapshot của EBS volume tại một thời điểm*

> Not necessary to detach volume to do snapshot, but recommended  
> *Không bắt buộc detach volume trước khi snapshot, nhưng khuyến nghị*

> Can copy snapshots across AZ or Region  
> *Có thể copy snapshots giữa các AZ hoặc Region*

**Sử dụng AWS CLI:**
```bash
# Tạo snapshot
aws ec2 create-snapshot --volume-id vol-1234567890abcdef0 --description "My snapshot"

# Copy snapshot sang region khác
aws ec2 copy-snapshot --source-region us-east-1 --source-snapshot-id snap-1234567890abcdef0 --destination-region ap-southeast-1
```

---

### EBS Snapshots Features (Tính năng Snapshot)

**EBS Snapshot Archive:**
> Move a Snapshot to an "archive tier" that is 75% cheaper  
> *Chuyển Snapshot sang "archive tier" giảm 75% chi phí*

> Takes within 24 to 72 hours for restoring the archive  
> *Mất 24-72 giờ để restore từ archive*

**Recycle Bin for EBS Snapshots:**
> Setup rules to retain deleted snapshots so you can recover them after an accidental deletion  
> *Thiết lập rules để giữ lại deleted snapshots để khôi phục sau khi vô tình xóa*

> Specify retention (from 1 day to 1 year)  
> *Chỉ định retention (từ 1 ngày đến 1 năm)*

**Fast Snapshot Restore (FSR):**
> Force full initialization of snapshot to have no latency on the first use ($$$)  
> *Buộc khởi tạo đầy đủ snapshot để không có latency khi sử dụng lần đầu (tốn phí)*

---

### EBS Snapshots vs AMI (So sánh)

| | EBS Snapshot | AMI |
|--|--------------|-----|
| **Là gì?** | Copy của một volume cụ thể | Backup toàn bộ EC2 instance |
| **Launch được không?** | ❌ Không | ✅ Có |
| **Use case** | Backup data của D Drive, E Drive... | Backup entire instance, scale ra nhiều instances |

**Snapshot:**
> You cannot launch an EC2 instance from an EBS Snapshot  
> *Không thể launch EC2 instance từ EBS Snapshot*

**AMI:**
> You can launch an EC2 instance from an AMI  
> *Có thể launch EC2 instance từ AMI*

---

### AMI Overview (Tổng quan AMI)

> AMI = Amazon Machine Image

> AMI are a customization of an EC2 instance  
> *AMI là sự tùy chỉnh của EC2 instance*

> You add your own software, configuration, operating system, monitoring...  
> *Bạn thêm software, configuration, OS, monitoring của riêng*

> Faster boot / configuration time because all your software is pre-packaged  
> *Boot và configure nhanh hơn vì tất cả software đã được đóng gói sẵn*

> AMI are built for a specific region (and can be copied across regions)  
> *AMI được tạo cho một region cụ thể (có thể copy sang regions khác)*

**Nguồn AMI:**
| Loại | Mô tả |
|------|-------|
| **Public AMI** | AWS cung cấp sẵn |
| **Your own AMI** | Bạn tự tạo và bảo trì |
| **AWS Marketplace AMI** | Người khác tạo (có thể bán) |

---

### AMI Process (Quy trình tạo AMI)

> Start an EC2 instance and customize it  
> *Khởi tạo EC2 instance và tùy chỉnh*

> Stop the instance (for data integrity)  
> *Stop instance (để đảm bảo data integrity)*

> Build an AMI - this will also create EBS snapshots  
> *Build AMI - đồng thời tạo EBS snapshots*

> Launch instances from other AMIs  
> *Launch instances từ các AMIs*

---

### EBS - Delete on Termination Attribute

> By default, the root EBS volume is deleted (attribute enabled)  
> *Mặc định, root EBS volume bị xóa (attribute enabled)*

> By default, Non-root EBS volume is not deleted (attribute disabled)  
> *Mặc định, non-root EBS volume không bị xóa (attribute disabled)*

> Use case: preserve root volume when instance is terminated  
> *Use case: giữ lại root volume khi instance bị terminate*

**💡 Mẹo:** Disable "Delete on Termination" trên root volume nếu muốn preserve data sau khi instance terminate.

---

### EBS Encryption (Mã hóa EBS)

> When you create an encrypted EBS volume, you get the following:  
> *Khi tạo encrypted EBS volume, bạn có được:*

| Loại mã hóa | Mô tả |
|-------------|-------|
| **Data at rest** | Data trong volume được mã hóa |
| **Data in flight** | Data di chuyển giữa instance và volume được mã hóa |
| **Snapshots** | Tất cả snapshots đều được mã hóa |
| **Volumes from snapshot** | Tất cả volumes tạo từ snapshot đều được mã hóa |

> Encryption has a minimal impact on latency  
> *Mã hóa có impact tối thiểu đến latency*

> EBS Encryption leverages keys from KMS (AES-256)  
> *EBS Encryption sử dụng keys từ KMS (AES-256)*

**Mã hóa volume chưa mã hóa:**
```
1. Tạo EBS snapshot của volume
2. Encrypt snapshot (dùng copy)
3. Tạo volume mới từ snapshot (volume sẽ được encrypt)
4. Attach encrypted volume vào instance
```

---

### EC2 Instance Store

> EC2 Instance Store is high-performance hardware disk  
> *EC2 Instance Store là hardware disk hiệu năng cao*

> Better I/O performance  
> *Hiệu năng I/O tốt hơn*

> Risk of data loss when EC2 are stopped (ephemeral)  
> *Có nguy cơ mất dữ liệu khi EC2 bị stop (ephemeral/tạm thời)*

> Good for buffer / cache / scratch data / temporary content  
> *Phù hợp cho buffer/cache/scratch data/temporary content*

> Some instance types do not support instance store volumes  
> *Một số instance types không hỗ trợ instance store volumes*

**⚠️ Cảnh báo:** Instance Store là **ephemeral** - data bị mất khi:
- Instance stop/start
- Instance terminate
- Hardware failure

**Use cases phù hợp:**
- Buffer/Cache data
- Scratch data
- Temporary content
- Write-heavy databases với replication

---

### Amazon EFS - Elastic File System

> Managed NFS (network file system)  
> *Hệ thống file mạng được quản lý*

> Can be mounted on many EC2  
> *Có thể mount trên nhiều EC2*

> EFS works with EC2 instances in Multi-AZ  
> *EFS hoạt động với EC2 instances across Multi-AZ*

> Highly available, scalable, expensive (3x gp2)  
> *Khả dụng cao, scalable, đắt hơn (3x gp2)*

**Use cases:**
- Content management
- Web serving
- Data sharing

> Uses NFSv4.1 protocol  
> *Sử dụng NFSv4.1 protocol*

> Compatible with Linux based AMI (not Windows)  
> *Tương thích với Linux AMI (không hỗ trợ Windows)*

> POSIX file system (~Linux)  
> *POSIX file system*

---

### Amazon EFS - Storage Classes

| Storage Class | Designed for | Latency | Durability | Min billing |
|--------------|-------------|---------|------------|-------------|
| **EFS Standard** | Active data cần sub-millisecond latency | Sub-ms | 99.999999999% | N/A |
| **EFS IA** (Infrequent Access) | Data truy cập vài lần mỗi quarter | Tens of ms | 99.999999999% | 128 KiB |
| **EFS Archive** | Data truy cập vài lần/năm | Tens of ms | 99.999999999% | 128 KiB, 90 days |

---

### EBS vs EFS vs Instance Store (So sánh)

| Đặc điểm | EBS | EFS | Instance Store |
|----------|-----|-----|---------------|
| **Kiểu** | Block storage | File storage (NFS) | Local storage |
| **Share được?** | Không (trừ io1/io2) | ✅ Nhiều EC2 | ✅ Nhiều EC2 |
| **Multi-AZ** | ❌ Chỉ 1 AZ | ✅ Cross AZ | ❌ |
| **Giá** | Medium | Cao (3x gp2) | Miễn phí (có sẵn) |
| **Persistence** | ✅ Data tồn tại | ✅ Always available | ❌ Mất khi stop |
| **Use case** | Database, boot | Shared file system | Cache, temp |

---

## 2. High Availability & Scalability

### Scalability & High Availability (Khả năng mở rộng & Khả dụng cao)

> Scalability means that an application / system can handle greater loads by adapting.  
> *Scalability nghĩa là application/system có thể xử lý loads lớn hơn bằng cách thích nghi*

> There are two kinds of scalability:  
> *Có 2 loại scalability:*

| Loại | Mô tả |
|------|-------|
| **Vertical Scalability** | Tăng kích thước instance (scale up/down) |
| **Horizontal Scalability** (= elasticity) | Tăng số lượng instances (scale out/in) |

> Scalability is linked but different to High Availability  
> *Scalability liên quan nhưng khác với High Availability*

---

### Vertical Scalability (Mở rộng theo chiều dọc)

> Vertically scalability (= scale up / down) means increasing the size of the instance  
> *Vertical scalability nghĩa là tăng kích thước của instance*

> Vertical scalability is very common for non distributed systems, such as a database.  
> *Vertical scalability rất phổ biến cho các hệ thống không phân tán, như database*

> RDS, ElastiCache are services that can scale vertically.  
> *RDS, ElastiCache là các services có thể scale vertically*

**Ví dụ:**
```
t2.micro (1 vCPU, 1 GB RAM) → t2.large (2 vCPU, 8 GB RAM)
M5.large (2 vCPU, 8 GB RAM) → M5.4xlarge (16 vCPU, 64 GB RAM)
```

---

### Horizontal Scalability (Mở rộng theo chiều ngang)

> Horizontal Scalability (= scale out / in) means increasing the number of instances / systems for your application  
> *Horizontal scalability nghĩa là tăng số lượng instances/systems cho application*

> Horizontal scaling implies distributed systems.  
> *Horizontal scaling ngụ ý các hệ thống phân tán*

> This is very common for web applications / modern applications  
> *Điều này rất phổ biến cho web applications/modern applications*

**Auto Scaling Group:**
```
[EC2] [EC2] [EC2] [EC2]
  ↕        ↕        ↕        ↕
Scale Out ← Load Balancer → Scale In
```

---

### High Availability (Khả dụng cao)

> High Availability usually goes hand in hand with horizontal scaling  
> *High Availability thường đi kèm với horizontal scaling*

> High availability means running your application / system in at least 2 data centers (== Availability Zones)  
> *High availability nghĩa là chạy application/system ở ít nhất 2 data centers (== Availability Zones)*

> The goal of high availability is to survive a data center loss  
> *Mục tiêu của high availability là sống sót qua việc mất data center*

| Loại HA | Ví dụ |
|---------|-------|
| **Passive HA** | RDS Multi-AZ |
| **Active HA** | Horizontal scaling với Load Balancer |

---

## 3. ELB - Elastic Load Balancer

### What is Load Balancing? (Load Balancer là gì?)

> Load Balancers are servers that forward traffic to multiple servers (e.g., EC2 instances) downstream  
> *Load Balancers là servers forward traffic đến nhiều servers (ví dụ: EC2 instances)*

---

### Why use a Load Balancer? (Tại sao dùng Load Balancer?)

| Lợi ích | Mô tả |
|---------|-------|
| **Spread load** | Phân phối load across multiple instances |
| **Single point of access** | Một DNS endpoint duy nhất |
| **Handle failures** | Tự động handle downstream failures |
| **Health checks** | Kiểm tra health của instances |
| **SSL Termination** | HTTPS termination tại LB |
| **Stickiness** | Duy trì session với cookies |
| **High availability** | Across zones |
| **Separate traffic** | Tách public/private traffic |

---

### Why use an Elastic Load Balancer? (Tại sao dùng ELB?)

> An Elastic Load Balancer is a managed load balancer  
> *ELB là một managed load balancer*

> AWS guarantees that it will be working  
> *AWS guarantee nó sẽ hoạt động*

> AWS takes care of upgrades, maintenance, high availability  
> *AWS lo việc upgrades, maintenance, HA*

> AWS provides only a few configuration knobs  
> *AWS chỉ cung cấp một vài configuration options*

**ELB tích hợp với:**
- EC2, EC2 Auto Scaling Groups, Amazon ECS
- AWS Certificate Manager (ACM), CloudWatch
- Route 53, AWS WAF, AWS Global Accelerator

---

### Health Checks (Kiểm tra sức khỏe)

> Health Checks are crucial for Load Balancers  
> *Health Checks rất quan trọng cho Load Balancers*

> They enable the load balancer to know if instances it forwards traffic to are available to reply to requests  
> *Chúng cho phép LB biết instances có sẵn sàng trả lời requests không*

> The health check is done on a port and a route (/health is common)  
> *Health check được thực hiện trên port và route (/health là phổ biến)*

> If the response is not 200 (OK), then the instance is unhealthy  
> *Nếu response không phải 200 (OK), instance được coi là unhealthy*

**Ví dụ Health Check:**
```
Protocol: HTTP
Port: 80
Path: /health
Healthy threshold: 2 consecutive successes
Unhealthy threshold: 2 consecutive failures
Timeout: 5 seconds
Interval: 30 seconds
```

---

### Types of Load Balancer on AWS (Các loại LB)

| Feature | ALB | NLB | GWLB | CLB |
|---------|-----|-----|------|-----|
| **Layer** | Layer 7 | Layer 4 | Layer 3 + Layer 4 | Layer 4/7 |
| **Target type** | IP, Instance, Lambda | IP, Instance, ALB | IP, Instance | Instance |
| **Protocol** | HTTP, HTTPS, gRPC | TCP, UDP, TLS | GENEVE (6081) | TCP, SSL/TLS, HTTP |
| **Use case** | Microservices, Containers | Extreme performance | Firewalls, IDS/IPS | Legacy |

---

### Application Load Balancer (ALB) - Layer 7

> Application load balancers is Layer 7 (HTTP) of OSI model (support HTTP/HTTPS)  
> *ALB là Layer 7 (HTTP) của OSI model*

> Load balancing to multiple HTTP applications across machines (target groups)  
> *Cân bằng tải đến nhiều HTTP applications across machines*

> Load balancing to multiple applications on the same machine (ex: containers)  
> *Cân bằng tải đến nhiều applications trên cùng một machine (ví dụ: containers)*

> Support for HTTP/2 and WebSocket  
> *Hỗ trợ HTTP/2 và WebSocket*

> Support redirects (from HTTP to HTTPS for example)  
> *Hỗ trợ redirects*

**ALB Routing:**
- Path-based: `/users`, `/posts`
- Host-based: `one.example.com`, `other.example.com`
- Query String: `/users?id=123&order=false`
- Headers-based

**💡 ALB là lựa chọn phổ biến nhất cho web applications hiện đại**

---

### Application Load Balancer - Target Groups

> EC2 instances (can be managed by an Auto Scaling Group) - HTTP  
> *EC2 instances (có thể quản lý bởi ASG)*

> ECS tasks (managed by ECS itself) - HTTP  
> *ECS tasks*

> Lambda functions - HTTP request is translated into a JSON event  
> *Lambda functions - HTTP request được chuyển thành JSON event*

> IP Addresses - must be private IPs  
> *IP Addresses - phải là private IPs*

> ALB can route to multiple target groups  
> *ALB có thể route đến nhiều target groups*

> Health checks are at the target group level  
> *Health checks ở target group level*

---

### Application Load Balancer - Good to Know

> Fixed hostname (XXX.region.elb.amazonaws.com)  
> *Hostname cố định*

> The application servers don't see the IP of the client directly  
> *Application servers không thấy trực tiếp IP của client*

> The true IP of the client is inserted in the header X-Forwarded-For  
> *IP thật của client được insert vào header X-Forwarded-For*

**💡 Để lấy IP thật của client trong application:**
```python
client_ip = request.headers.get('X-Forwarded-For', '').split(',')[0].strip()
```

---

### Network Load Balancer (NLB) - Layer 4

> Network load balancers (Layer 4) allow to:  
> *NLB cho phép:*

> Forward TCP & UDP traffic to your instances  
> *Forward TCP & UDP traffic*

> Handle millions of request per seconds  
> *Xử lý hàng triệu requests/giây*

> Less latency ~100 ms (vs 400 ms for ALB)  
> *Độ trễ thấp hơn ~100ms (so với ALB ~400ms)*

> NLB has one static IP per AZ, and supports assigning Elastic IP (helpful for whitelisting specific IP)  
> *NLB có một static IP mỗi AZ, hỗ trợ Elastic IP*

> NLB are used for extreme performance, TCP or UDP traffic  
> *NLB dùng cho extreme performance*

> Not included in the AWS free tier  
> *Không có trong free tier*

---

### Gateway Load Balancer (GWLB)

> Deploy, scale, and manage a fleet of 3rd party network virtual appliances in AWS  
> *Triển khai, scale và quản lý 3rd party network virtual appliances*

**Use cases:**
- Firewalls
- Intrusion Detection and Prevention Systems (IDS/IPS)
- Deep Packet Inspection Systems
- Payload manipulation

> Operates at Layer 3 (Network Layer) - IP Packets  
> *Hoạt động ở Layer 3*

> Uses the GENEVE protocol on port 6081  
> *Sử dụng GENEVE protocol trên port 6081*

---

### Sticky Sessions (Session Affinity)

> It is possible to implement stickiness so that the same client is always redirected to the same instance behind a load balancer  
> *Có thể implement stickiness để cùng một client luôn được redirect đến cùng một instance*

> For both CLB & ALB, the "cookie" used for stickiness has an expiration date you control  
> *Cookie có expiration date mà bạn kiểm soát*

> Use case: make sure the user doesn't lose his session data  
> *Use case: đảm bảo user không mất session data*

> Enabling stickiness may bring imbalance to the load over the backend EC2 instances  
> *Stickiness có thể gây mất cân bằng load*

**⚠️ Cân nhắc:** Stickiness có thể gây uneven distribution. Cân nhắc dùng session store (ElastiCache) thay thế.

---

### Cross-Zone Load Balancing

**With Cross Zone Load Balancing:**
> Each load balancer instance distributes evenly across all registered instances in all AZ  
> *Mỗi LB instance phân phối đều across all instances trong tất cả AZ*

**Without Cross Zone Load Balancing:**
> Requests are distributed in the instances of the node of the Elastic Load Balancer  
> *Requests chỉ phân phối trong instances của LB node đó*

**Mặc định theo loại LB:**

| LB Type | Cross-Zone Default | Charges |
|---------|-------------------|---------|
| ALB | ✅ Enabled (default) | Không tính phí |
| NLB | ❌ Disabled (default) | Có tính phí |
| GWLB | ❌ Disabled (default) | Có tính phí |

---

### SSL/TLS - Basics

> An SSL Certificate allows traffic between your clients and your load balancer to be encrypted in transit (in-flight encryption)  
> *SSL Certificate mã hóa traffic giữa clients và LB*

> SSL refers to Secure Sockets Layer, used to encrypt connections  
> *SSL - Secure Sockets Layer, dùng để mã hóa connections*

> TLS refers to Transport Layer Security, which is a newer version  
> *TLS - Transport Layer Security, phiên bản mới hơn*

> Public SSL certificates are issued by Certificate Authorities (CA)  
> *Public SSL certificates được issue bởi Certificate Authorities*

> SSL certificates have an expiration date (you set) and must be renewed  
> *SSL certificates có expiration date và phải renewed*

---

### Load Balancer - SSL Certificates

> The load balancer uses an X.509 certificate (SSL/TLS server certificate)  
> *LB sử dụng X.509 certificate*

> You can manage certificates using ACM (AWS Certificate Manager)  
> *Quản lý certificates bằng ACM*

> HTTPS listener:  
> *HTTPS listener:*

> You must specify a default certificate  
> *Phải chỉ định default certificate*

> You can add an optional list of certs to support multiple domains  
> *Có thể thêm list of certs để hỗ trợ nhiều domains*

> Clients can use SNI (Server Name Indication) to specify the hostname they reach  
> *Clients có thể dùng SNI để chỉ định hostname*

---

### SSL - Server Name Indication (SNI)

> SNI solves the problem of loading multiple SSL certificates onto one web server (to serve multiple websites)  
> *SNI giải quyết vấn đề load nhiều SSL certificates trên một web server*

> Only works for ALB & NLB (newer generation), CloudFront  
> *Chỉ hoạt động với ALB & NLB (thế hệ mới), CloudFront*

> Does not work for CLB (older gen)  
> *Không hoạt động với CLB*

**💡 SNI cho phép một LB serve multiple domains với multiple SSL certificates**

---

### Connection Draining

| Tên gọi | Load Balancer |
|---------|--------------|
| **Connection Draining** | CLB |
| **Deregistration Delay** | ALB & NLB |

> Time to complete "in-flight requests" while the instance is de-registering or unhealthy  
> *Thời gian hoàn thành "in-flight requests" khi instance deregistering hoặc unhealthy*

> Stops sending new requests to the EC2 instance which is de-registering  
> *Ngừng gửi requests mới đến instance đang deregister*

> Between 1 to 3600 seconds (default: 300 seconds)  
> *1-3600 giây (mặc định: 300 giây)*

> Can be disabled (set value to 0)  
> *Có thể disable (set = 0)*

> Set to a low value if your requests are short  
> *Set giá trị thấp nếu requests ngắn*

---

## 4. ASG - Auto Scaling Group

### What's an Auto Scaling Group? (ASG là gì?)

> In real-life, the load on your websites and application can change  
> *Trong thực tế, load trên websites và application có thể thay đổi*

**Goal của ASG:**
> Scale out (add EC2 instances) to match an increased load  
> *Scale out (thêm EC2 instances) để match increased load*

> Scale in (remove EC2 instances) to match a decreased load  
> *Scale in (xóa EC2 instances) để match decreased load*

> Ensure we have a minimum and a maximum number of EC2 instances running  
> *Đảm bảo có min và max number of EC2 instances*

> Automatically register new instances to a load balancer  
> *Tự động register new instances với load balancer*

> Re-create an EC2 instance in case a previous one is terminated (ex: if unhealthy)  
> *Tái tạo EC2 instance nếu instance trước bị terminate (ví dụ: unhealthy)*

> ASG are free (you only pay for the underlying EC2 instances)  
> *ASG miễn phí (chỉ trả tiền cho EC2 instances)*

---

### Auto Scaling Group Attributes (Thuộc tính ASG)

**Launch Template (thay thế Launch Configurations - đã deprecated):**

| Thuộc tính | Mô tả |
|-----------|-------|
| AMI + Instance Type | Image và loại instance |
| EC2 User Data | Bootstrap script |
| EBS Volumes | Volumes |
| Security Groups | Firewall rules |
| SSH Key Pair | Để SSH |
| IAM Roles | Cho EC2 instances |
| Network + Subnets | Network settings |
| Load Balancer | Target groups |
| Min/Max/Initial | Số lượng instances |

---

### Auto Scaling - CloudWatch Alarms & Scaling

> It is possible to scale an ASG based on CloudWatch alarms  
> *Có thể scale ASG dựa trên CloudWatch alarms*

> An alarm monitors a metric (such as Average CPU, or a custom metric)  
> *Alarm giám sát một metric*

> Metrics such as Average CPU are computed for the overall ASG instances  
> *Metrics được tính toán cho toàn bộ ASG instances*

**Scaling Policies:**

| Policy | Mô tả |
|--------|-------|
| **Target Tracking** | Giữ metric ở một target value |
| **Simple/Step** | Thêm/bớt instances khi alarm trigger |
| **Scheduled** | Scale theo lịch (known patterns) |
| **Predictive** | ML predict future capacity |

---

### Auto Scaling Groups - Scaling Policies

**Target Tracking Scaling (strongly recommend):**
> Scale a resource based on a target value for a specific CloudWatch metric  
> *Scale dựa trên target value cho một CloudWatch metric*

> Example: I want the average ASG CPU to stay at around 40%  
> *Ví dụ: Giữ CPU trung bình ở ~40%*

**Simple / Step Scaling:**
> When a CloudWatch alarm is triggered (example CPU > 70%), then add 2 units  
> *Khi alarm trigger (CPU > 70%), thêm 2 units*

> When a CloudWatch alarm is triggered (example CPU < 30%), then remove 1  
> *Khi alarm trigger (CPU < 30%), xóa 1 unit*

**Scheduled Scaling:**
> Scale based on known usage patterns  
> *Scale dựa trên known usage patterns*

> Example: increase the min capacity to 10 at 5 pm on Fridays  
> *Ví dụ: Tăng min capacity lên 10 vào 5pm thứ 6*

**Predictive Scaling:**
> Uses machine learning to predict capacity requirements based on historical data from CloudWatch  
> *Sử dụng ML để predict capacity requirements*

---

### Good Metrics to Scale On (Metrics tốt để scale)

| Metric | Mô tả |
|--------|-------|
| **CPUUtilization** | CPU utilization trung bình across instances |
| **RequestCountPerTarget** | Số requests mỗi instance ổn định |
| **Average Network In / Out** | Khi application bị bound bởi network |
| **Custom Metric** | Metric tùy chỉnh push bằng CloudWatch |

---

### Auto Scaling Groups - Scaling Cooldowns

> After a scaling activity happens, you are in the cooldown period (default 300 seconds)  
> *Sau khi scaling activity xảy ra, có cooldown period (mặc định 300 giây)*

> During the cooldown period, the ASG will not launch or terminate additional instances (to allow for metrics to stabilize)  
> *Trong cooldown, ASG không launch/terminate thêm instances*

> Advice: Use a ready-to-use AMI to reduce configuration time in order to be serving requests faster and reduce the cooldown period  
> *Khuyến nghị: Dùng pre-configured AMI để giảm thời gian configure*

**💡 Cooldown giúp metrics ổn định trước khi scale tiếp**

---

### Auto Scaling - Instance Refresh

> Goal: update launch template and then re-creating all EC2 instances  
> *Mục tiêu: Update launch template và tái tạo tất cả EC2 instances*

> Setting of minimum healthy percentage  
> *Thiết lập minimum healthy percentage*

> Specify warm-up time (how long until the instance is ready to use)  
> *Chỉ định warm-up time*

---

## 5. Tổng kết

### Kiến thức trọng tâm cho DVA Exam

| Chủ đề | Trọng tâm |
|--------|-----------|
| **EBS Volume Types** | gp2/gp3, io1/io2, st1/sc1 - use cases và specs |
| **EBS Snapshots** | Backup, cross-region copy, lifecycle policies |
| **AMI** | Tạo từ EC2, sử dụng làm template |
| **EBS Encryption** | KMS (AES-256), encrypt unencrypted volumes |
| **EFS vs EBS vs Instance Store** | So sánh use cases |
| **Scalability Types** | Vertical vs Horizontal vs HA |
| **Load Balancers** | ALB (Layer 7), NLB (Layer 4), CLB, GWLB |
| **ALB Features** | Routing, target groups, health checks, sticky sessions |
| **ASG** | Launch template, scaling policies, cooldown |
| **Cross-Zone LB** | Default settings và charges |

### So sánh nhanh Load Balancers

| Feature | ALB | NLB | CLB |
|---------|-----|-----|-----|
| Layer | 7 (HTTP) | 4 (TCP/UDP) | 4/7 |
| Target Types | IP, Instance, Lambda | IP, Instance, ALB | Instance |
| Static IP | ❌ | ✅ | ❌ |
| SNI | ✅ | ✅ | ❌ |
| Cross-Zone | Default ✅ | Default ❌ | Default ❌ |

### Best Practices 2026

1. **EBS:**
   - Dùng gp3 thay vì gp2 (giá tốt hơn)
   - Enable encryption by default
   - Regular backups với lifecycle policies
   - Dùng EFS cho shared storage cần multi-instance

2. **Load Balancer:**
   - ALB cho hầu hết web applications
   - NLB cho extreme performance, game servers
   - Always enable health checks
   - Dùng ACM cho SSL certificates

3. **ASG:**
   - Always set min/max appropriately
   - Dùng Target Tracking policy (recommend by AWS)
   - Pre-bake AMIs để giảm bootstrap time
   - Dùng multiple AZs

---

## Liên kết tham khảo

- [Amazon EBS Documentation](https://docs.aws.amazon.com/ebs/)
- [Elastic Load Balancing Documentation](https://docs.aws.amazon.com/elasticloadbalancing/)
- [Amazon EC2 Auto Scaling Documentation](https://docs.aws.amazon.com/autoscaling/)

---

*Document này được tạo để hỗ trợ học tập cho kỳ thi AWS Certified Developer - Associate*
