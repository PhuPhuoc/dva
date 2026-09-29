# Section 5: Amazon VPC (Virtual Private Cloud)

**AWS Certified Developer - Associate**  
*Nguồn: Cloud Mentor Pro - Cập nhật: 15/02/2026*

---

## Mục lục

1. [VPC Overview & Components](#1-vpc-overview--components)
2. [CIDR & Subnets](#2-cidr--subnets)
3. [Internet Gateway & Route Tables](#3-internet-gateway--route-tables)
4. [NAT Gateway & Bastion Hosts](#4-nat-gateway--bastion-hosts)
5. [NACLs vs Security Groups](#5-nacls-vs-security-groups)
6. [VPC Peering](#6-vpc-peering)
7. [VPC Endpoints](#7-vpc-endpoints)
8. [VPC Flow Logs](#8-vpc-flow-logs)
9. [Site-to-Site VPN & Direct Connect](#9-site-to-site-vpn--direct-connect)
10. [Tổng kết](#10-tổng-kết)

---

## 1. VPC Overview & Components

### VPC Components Overview

**VPC = Virtual Private Cloud**  
*Là một mạng ảo riêng trong AWS*

**Các thành phần chính:**

| Component | Mô tả |
|----------|--------|
| **Internet Gateway** | Kết nối VPC với Internet |
| **Route Tables** | Định tuyến traffic |
| **Subnets** | Chia mạng con trong VPC |
| **NAT Gateway** | Cho phép private instances ra Internet |
| **Security Groups** | Firewall ở instance level |
| **NACLs** | Firewall ở subnet level |
| **VPC Peering** | Kết nối 2 VPCs |
| **VPC Endpoints** | Truy cập AWS services riêng tư |
| **Bastion Host** | Jump server để SSH vào private instances |

**Architecture Diagram:**
```
Internet
    │
    ▼
Internet Gateway
    │
    ▼
┌─────────────────────────────────────────┐
│              VPC                          │
│                                          │
│  ┌────────────────┐  ┌────────────────┐  │
│  │ Public Subnet  │  │ Private Subnet │  │
│  │                │  │                │  │
│  │ • Bastion Host │  │ • EC2 (Private)│  │
│  │ • NAT Gateway │  │                │  │
│  └────────────────┘  └────────────────┘  │
│                                          │
└─────────────────────────────────────────┘
```

---

### Default VPC Walkthrough

> All new AWS accounts have a default VPC  
> *Tất cả accounts mới đều có default VPC*

> New EC2 instances are launched into the default VPC if no subnet is specified  
> *EC2 instances mới được launch vào default VPC nếu không chỉ định subnet*

> Default VPC has Internet connectivity and all EC2 instances inside it have public IPv4 addresses  
> *Default VPC có Internet connectivity và EC2 có public IPv4 addresses*

**💡 Default VPC bị xóa?** Có thể tạo lại default VPC qua AWS Console hoặc CLI.

---

## 2. CIDR & Subnets

### Understanding CIDR - IPv4

> CIDR: Classless Inter-Domain Routing - IP Range  
> *CIDR: Định nghĩa dải IP*

**Ví dụ VPC:**
```
172.22.241.128/25
│    │    │    │   │
│    │    │    │   └── Prefix (số bits network)
│    │    │    └──────── Octet thứ 4
│    │    └───────────── Octet thứ 3
│    └────────────────── Octet thứ 2
└─────────────────────── Octet thứ 1
```

**Tính số IPs:**
```
/24 = 256 IPs (2^(32-24) = 256)
/25 = 128 IPs
/26 = 64 IPs
/27 = 32 IPs
/28 = 16 IPs
/29 = 8 IPs
/30 = 4 IPs
```

---

### VPC in AWS - IPv4

> VPC = Virtual Private Cloud  
> *Virtual Private Cloud*

> You can have multiple VPCs in an AWS region (max. 5 per region - soft limit)  
> *Có thể tạo nhiều VPCs trong region (tối đa 5, có thể tăng)*

> Max. CIDR per VPC is 5, for each CIDR:
> *Tối đa 5 CIDR blocks mỗi VPC*

> Min. size is /28 (16 IP addresses)  
> *Kích thước tối thiểu: /28 (16 IPs)*

> Max. size is /16 (65536 IP addresses)  
> *Kích thước tối đa: /16 (65536 IPs)*

> We recommend that you specify a CIDR block from the private IPv4 address ranges as specified in RFC 1918  
> *Nên dùng private IP ranges theo RFC 1918*

**RFC 1918 Private IP Ranges:**
| Range | CIDR | Số IPs |
|-------|------|--------|
| 10.0.0.0 - 10.255.255.255 | 10.0.0.0/8 | 16,777,216 |
| 172.16.0.0 - 172.31.255.255 | 172.16.0.0/12 | 1,048,576 |
| 192.168.0.0 - 192.168.255.255 | 192.168.0.0/16 | 65,536 |

---

### VPC - Subnet (IPv4)

> AWS reserves 5 IP addresses (first 4 & last 1) in each subnet  
> *AWS reserved 5 IPs trong mỗi subnet*

> These 5 IP addresses are not available for use and can't be assigned to an EC2 instance  
> *5 IPs này không thể dùng cho EC2*

**Ví dụ subnet 10.0.0.0/24:**
| IP | Reserved cho |
|----|--------------|
| 10.0.0.0 | Network Address |
| 10.0.0.1 | VPC Router |
| 10.0.0.2 | Amazon DNS |
| 10.0.0.3 | Reserved for future use |
| 10.0.0.255 | Network Broadcast |

**💡 Exam Tip:**
```
Subnet /27 = 32 IPs - 5 = 27 IPs usable
Subnet /28 = 16 IPs - 5 = 11 IPs usable
Subnet /29 = 8 IPs - 5 = 3 IPs usable

Cần 29 EC2 instances?
• /27: 32 - 5 = 27 ❌ Không đủ
• /26: 64 - 5 = 59 ✅ Đủ
```

---

## 3. Internet Gateway & Route Tables

### Internet Gateway (IGW)

> Allows resources (e.g., EC2 instances) in a VPC connect to the Internet  
> *Cho phép resources trong VPC kết nối Internet*

> Must be created separately from a VPC  
> *Phải tạo riêng biệt với VPC*

> One VPC can only be attached to one IGW and vice versa  
> *Một VPC chỉ gắn được một IGW và ngược lại*

> Internet Gateways on their own do not allow Internet access...  
> *IGW một mình không cho phép Internet access*

> Route tables must also be edited!  
> *Phải edit Route Tables!*

**Tạo IGW:**
```
1. Tạo Internet Gateway
2. Attach vào VPC
3. Tạo Route Table cho Public Subnet
4. Thêm route: Destination 0.0.0.0/0 → Target IGW
```

---

### Editing Route Tables

**Public Subnet Route Table:**
```
Destination          Target
─────────────────   ───────────────────
10.0.0.0/16         Local           (traffic trong VPC)
0.0.0.0/0          igw-xxxxx       (traffic ra Internet)
```

**Private Subnet Route Table:**
```
Destination          Target
─────────────────   ───────────────────
10.0.0.0/16         Local           (traffic trong VPC)
0.0.0.0/0          nat-xxxxx       (qua NAT Gateway)
```

---

## 4. NAT Gateway & Bastion Hosts

### NAT Gateway

> AWS-managed NAT, higher bandwidth, high availability, no administration  
> *NAT Gateway được quản lý bởi AWS, băng thông cao, HA*

> Pay per hour for usage and bandwidth  
> *Trả phí theo giờ và bandwidth*

> NATGW is created in a specific Availability Zone, uses an Elastic IP  
> *NATGW được tạo trong một AZ cụ thể, dùng Elastic IP*

> Requires an IGW (Private Subnet => NATGW => IGW)  
> *Cần có IGW*

> 5 Gbps of bandwidth with automatic scaling up to 100 Gbps  
> *5 Gbps, auto scale đến 100 Gbps*

> No Security Groups to manage / required  
> *Không cần Security Groups*

**Flow của NAT Gateway:**
```
Private EC2 (10.0.1.x)
       │
       ▼
Private Subnet Route: 0.0.0.0/0 → NAT Gateway
       │
       ▼
NAT Gateway (trong Public Subnet)
       │
       ▼
Internet Gateway
       │
       ▼
Internet
```

**💡 Multi-AZ:** Tạo NAT Gateway trong mỗi AZ để ensure high availability.

---

### Bastion Hosts

> We can use a Bastion Host to SSH into our private EC2 instances  
> *Dùng Bastion Host để SSH vào private EC2 instances*

> The bastion is in the public subnet which is then connected to all other private subnets  
> *Bastion nằm trong public subnet, kết nối đến private subnets*

**Cấu hình Bastion Host:**

| Component | Requirement |
|----------|-------------|
| Bastion SG | Allow inbound port 22 từ corporate IP |
| Private EC2 SG | Allow inbound port 22 từ Bastion SG hoặc IP |

**Flow:**
```
User → Bastion Host (Public) → Private EC2 (Private)
```

**Security Best Practice:**
- Limit SSH access đến bastion từ known IP ranges
- Sử dụng Session Manager thay vì SSH khi có thể

---

## 5. NACLs vs Security Groups

### Network Access Control List (NACL)

> NACL are like a firewall which control traffic from and to subnets  
> *NACL như firewall kiểm soát traffic vào/ra subnets*

> One NACL per subnet, new subnets are assigned the Default NACL  
> *Một NACL mỗi subnet*

> Can have ALLOW and DENY rules  
> *Có thể có ALLOW và DENY rules*

> Rules only include IP addresses  
> *Rules chỉ bao gồm IP addresses*

> NACL are a great way of blocking a specific IP address at the subnet level  
> *Dùng để block IP cụ thể ở subnet level*

### Security Group vs NACLs

| Feature | Security Group | NACL |
|---------|---------------|------|
| **Level** | Instance level | Subnet level |
| **Rules** | Allow rules only | Allow AND Deny rules |
| **State** | **Stateful** | **Stateless** |
| **Evaluation** | All rules evaluated | Evaluated in order (lowest to highest) |
| **Apply** | To specific EC2 instance | To ALL instances in subnet |
| **Return traffic** | Auto allowed | Must explicitly allow |

**Stateful vs Stateless:**

| | Stateful | Stateless |
|--|---------|-----------|
| **Outbound request** | Response auto allowed | Must add rule |
| **Security Group** | ✅ | ❌ |
| **NACL** | ❌ | ✅ |

**Default NACL:**
> Accepts everything inbound/outbound with the subnets it's associated with  
> *Chấp nhận tất cả traffic*

> Do NOT modify the Default NACL, instead create custom NACLs  
> *Không nên modify Default NACL, tạo custom NACLs*

**💡 Most cases:** Chỉ cần Security Groups. NACL chỉ cần khi có strict security requirements.

---

### NACLs - Ephemeral Ports

**Stateless = Phải handle ephemeral ports:**

| OS | Ephemeral Ports |
|----|-----------------|
| Linux | 32768-60999 |
| Windows | 49152-65535 |

**Ví dụ NACL Rule Order:**
```
Inbound:
100  | 0.0.0.0/0  | TCP  | 443  | ALLOW  (HTTPS from anywhere)
110  | 0.0.0.0/0  | TCP  | 80   | ALLOW  (HTTP from anywhere)
120  | 0.0.0.0/0  | TCP  | 1024-65535 | ALLOW  (Ephemeral ports - RESPONSE)
*    | 0.0.0.0/0  | TCP  | ALL  | DENY   (Block all)

Outbound:
100  | 0.0.0.0/0  | TCP  | 443  | ALLOW  (HTTPS out)
110  | 0.0.0.0/0  | TCP  | 80   | ALLOW  (HTTP out)
120  | 0.0.0.0/0  | TCP  | 1024-65535 | ALLOW  (Ephemeral ports - RESPONSE)
*    | 0.0.0.0/0  | TCP  | ALL  | DENY   (Block all)
```

---

## 6. VPC Peering

> Privately connect two VPCs using AWS' network  
> *Kết nối riêng tư 2 VPCs qua mạng AWS*

> Make them behave as if they were in the same network  
> *Làm cho chúng như ở trong cùng một mạng*

> Must not have overlapping CIDRs  
> *Không được trùng CIDRs*

> VPC Peering connection is NOT transitive  
> *VPC Peering KHÔNG có tính transitive*

**Ví dụ Transitive:**
```
VPC-A ↔ VPC-B (peered)
VPC-B ↔ VPC-C (peered)
❌ VPC-A → VPC-C (không tự động)
✅ Phải tạo VPC-A ↔ VPC-C peering riêng
```

> You must update route tables in each VPC's subnets to ensure EC2 instances can communicate with each other  
> *Phải update route tables trong mỗi VPC*

**Route Table cho VPC Peering:**
```
# Trong VPC-A Route Table
Destination        Target
────────────────   ─────────────────────
10.0.0.0/16       Local
10.1.0.0/16       pcx-xxxxx (VPC Peering)
```

---

## 7. VPC Endpoints

> Every AWS service is publicly exposed (public URL)  
> *Mọi AWS service đều có public URL*

> VPC Endpoints (powered by AWS PrivateLink) allows you to connect to AWS services using a private network instead of using the public Internet  
> *VPC Endpoints cho phép kết nối AWS services qua private network*

> They're redundant and scale horizontally  
> *Redundant và scale ngang*

> They remove the need of IGW, NATGW, ... to access AWS Services  
> *Loại bỏ nhu cầu IGW, NATGW để truy cập AWS Services*

**Use cases:**
- Private EC2 instances cần truy cập S3, DynamoDB
- Khi không muốn traffic ra Internet

---

### Types of Endpoints

| Feature | Interface Endpoints | Gateway Endpoints |
|---------|-------------------|-------------------|
| **Powered by** | PrivateLink | PrivateLink |
| **Connection** | ENI với private IP | Gateway trong Route Table |
| **Security Group** | Cần attach SG | ❌ Không cần |
| **Cost** | $ per hour + $ per GB | Miễn phí |
| **Services** | Most AWS services (SQS, SNS, etc.) | **Chỉ S3 và DynamoDB** |
| **Access** | From VPC hoặc on-premises | Từ VPC |

**Interface Endpoints:**
```
EC2 Instance → VPC Interface Endpoint (ENI) → AWS Service (SQS, SNS, etc.)
```

**Gateway Endpoints:**
```
EC2 Instance → Route Table → VPC Gateway Endpoint → S3 / DynamoDB
```

**Route Table cho Gateway Endpoint:**
```
Destination        Target
────────────────   ─────────────────────
10.0.0.0/16       Local
pl-xxxxxx         vpce-xxxxx (S3 Gateway)
```

---

## 8. VPC Flow Logs

> Capture information about IP traffic going into your interfaces:  
> *Bắt thông tin IP traffic vào interfaces*

| Level | Captures |
|-------|----------|
| **VPC Flow Logs** | Tất cả traffic trong VPC |
| **Subnet Flow Logs** | Traffic trong subnet |
| **ENI Flow Logs** | Traffic của một instance cụ thể |

> Helps to monitor & troubleshoot connectivity issues  
> *Giúp monitor và troubleshoot connectivity issues*

> Flow logs data can go to S3, CloudWatch Logs, and Kinesis Data Firehose  
> *Flow logs có thể gửi đến S3, CloudWatch Logs, Kinesis Data Firehose*

> Captures network information from AWS managed interfaces too: ELB, RDS, ElastiCache, Redshift, WorkSpaces, NATGW, Transit Gateway...  
> *Bắt traffic từ cả AWS managed interfaces*

---

### VPC Flow Logs Syntax

| Field | Mô tả |
|-------|--------|
| **srcaddr** | Source IP address |
| **dstaddr** | Destination IP address |
| **srcport** | Source port |
| **dstport** | Destination port |
| **Action** | ACCEPT hoặc REJECT |

**Query Flow Logs:**
- Amazon Athena (trên S3)
- CloudWatch Logs Insights

**💡 Troubleshooting:**
```
ACCEPT trong Flow Log → SG cho phép, nhưng application không respond
→ Kiểm tra application, security groups của destination
REJECT trong Flow Log → Bị blocked bởi SG hoặc NACL
→ Kiểm tra inbound rules
```

---

## 9. Site-to-Site VPN & Direct Connect

### Site-to-Site VPN

> Connect an on-premises VPN to AWS  
> *Kết nối on-premises VPN đến AWS*

> The connection is automatically encrypted  
> *Connection được tự động mã hóa*

> Goes over the public internet  
> *Đi qua public internet*

**Components:**
- Customer Gateway (CGW) - thiết bị VPN ở on-premises
- VPN Gateway (VGW) - Virtual Private Gateway ở AWS
- Site-to-Site VPN Connection

---

### Direct Connect (DX)

> Establish a physical connection between on-premises and AWS  
> *Thiết lập physical connection giữa on-premises và AWS*

> The connection is private, secure and fast  
> *Connection riêng tư, bảo mật, nhanh*

> Goes over a private network  
> *Đi qua private network*

> Takes at least a month to establish  
> *Mất ít nhất 1 tháng để thiết lập*

**So sánh:**

| Feature | Site-to-Site VPN | Direct Connect |
|---------|-----------------|---------------|
| Speed | Lên đến 1.25 Gbps | 1 Gbps - 400 Gbps |
| Latency | Qua Internet | Qua private network |
| Setup | Vài phút | Ít nhất 1 tháng |
| Cost | Per hour + data transfer | Dedicated connection + data transfer |
| Encryption | Mã hóa tự động | Có thể không mã hóa |

**Backup:**
> In case Direct Connect fails, you can set up a backup Direct Connect connection (expensive), or a Site-to-Site VPN connection  
> *VPN có thể làm backup cho Direct Connect*

---

## 10. Tổng kết

### Kiến thức trọng tâm cho DVA Exam

| Chủ đề | Trọng tâm |
|--------|-----------|
| **CIDR** | Tính số IPs, reserved IPs (5 IPs/subnet) |
| **VPC** | /28 to /16, private IP ranges (RFC 1918) |
| **Subnets** | Mỗi subnet gắn với 1 AZ |
| **Internet Gateway** | Cần tạo riêng, phải edit route table |
| **NAT Gateway** | Trong public subnet, cho phép private instances ra Internet |
| **Bastion Host** | Jump server vào private instances |
| **Security Groups** | Stateful, instance level, allow only |
| **NACLs** | Stateless, subnet level, allow + deny, ephemeral ports |
| **VPC Peering** | Non-transitive, no overlapping CIDRs |
| **VPC Endpoints** | Interface ($$) vs Gateway (free) - S3/DynamoDB |
| **VPC Flow Logs** | S3, CloudWatch, Athena |

### So sánh Security Groups vs NACLs

| | Security Group | NACL |
|--|---------------|------|
| **Level** | Instance | Subnet |
| **State** | Stateful | Stateless |
| **Rules** | Allow only | Allow + Deny |
| **Evaluation** | All rules | By order |
| **Default** | Deny all inbound | Allow all |

### CIDR Cheat Sheet

| CIDR | Total IPs | Usable IPs |
|------|-----------|------------|
| /24 | 256 | 251 |
| /25 | 128 | 123 |
| /26 | 64 | 59 |
| /27 | 32 | 27 |
| /28 | 16 | 11 |
| /29 | 8 | 3 |
| /30 | 4 | 1 |

### Best Practices 2026

**VPC Design:**
1. ✅ Mỗi environment (dev, staging, prod) có VPC riêng
2. ✅ Public subnets cho bastion hosts và NAT gateways
3. ✅ Private subnets cho application servers và databases
4. ✅ Tách biệt tiers (Web, App, DB) vào các subnets khác nhau
5. ✅ Enable VPC Flow Logs để monitor traffic

**Security:**
1. ✅ Sử dụng Security Groups làm primary firewall
2. ✅ Dùng NACLs chỉ khi cần block specific IPs
3. ✅ Bastion hosts với restricted SSH access
4. ✅ VPC Endpoints thay vì IGW/NATGW cho AWS services
5. ✅ Không để traffic không cần thiết qua NAT Gateway

**High Availability:**
1. ✅ NAT Gateway trong mỗi AZ
2. ✅ Bastion hosts across multiple AZs
3. ✅ Tạo Multi-AZ architecture

---

## Liên kết tham khảo

- [Amazon VPC Documentation](https://docs.aws.amazon.com/vpc/)
- [VPC CIDR Blocks](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-cidr-blocks.html)
- [VPC Security](https://docs.aws.amazon.com/vpc/latest/userguide/security.html)
- [VPC Endpoints](https://docs.aws.amazon.com/vpc/latest/privatelink/concepts.html)

---

*Document này được tạo để hỗ trợ học tập cho kỳ thi AWS Certified Developer - Associate*
