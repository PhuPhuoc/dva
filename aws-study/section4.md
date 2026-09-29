# Section 4: Amazon S3, S3 Advanced, S3 Security

**AWS Certified Developer - Associate**  
*Nguồn: Cloud Mentor Pro - Cập nhật: 15/02/2026*

---

## Mục lục

1. [Amazon S3 Overview](#1-amazon-s3-overview)
2. [Amazon S3 Buckets & Objects](#2-amazon-s3-buckets--objects)
3. [Amazon S3 Security](#3-amazon-s3-security)
4. [Amazon S3 Storage Classes](#4-amazon-s3-storage-classes)
5. [Amazon S3 Advanced](#5-amazon-s3-advanced)
6. [Tổng kết](#6-tổng-kết)

---

## 1. Amazon S3 Overview

### Amazon S3 Introduction

> Amazon S3 is one of the main building blocks of AWS  
> *S3 là một trong những building blocks chính của AWS*

> It's advertised as "infinitely scaling" storage  
> *Nó được quảng cáo là storage "infinitely scaling"*

**Benefits:**

| Benefit | Mô tả |
|---------|--------|
| **Scalability** | Khả năng mở rộng vô hạn |
| **Durability** | Độ bền cao nhất trong cloud - 99.999999999% (11 9's) |
| **Availability** | Cung cấp availability hàng đầu trong ngành |
| **Security** | Secure, private, và encrypted by default |
| **Storage Classes** | Nhiều storage classes với best price performance |
| **Performance** | Không giới hạn performance |

---

### Amazon S3 Use cases

| Use case | Mô tả |
|----------|--------|
| **Backup and storage** | Backup và lưu trữ |
| **Disaster Recovery** | Khôi phục thảm họa |
| **Archive** | Lưu trữ lâu dài |
| **Hybrid Cloud storage** | Storage hybrid cloud |
| **Application hosting** | Hosting ứng dụng |
| **Media hosting** | Hosting media files |
| **Data lakes & big data analytics** | Data lakes và phân tích big data |
| **Software delivery** | Phân phối phần mềm |
| **Static website** | Static website hosting |

---

## 2. Amazon S3 Buckets & Objects

### Amazon S3 - Buckets

> Amazon S3 allows people to store objects (files) in "buckets" (directories)  
> *S3 cho phép lưu trữ objects (files) trong "buckets"*

> Buckets must have a globally unique name (across all regions all accounts)  
> *Buckets phải có tên globally unique (across all regions và all accounts)*

> Buckets are defined at the region level  
> *Buckets được định nghĩa ở region level*

> S3 looks like a global service but buckets are created in a region  
> *S3 trông như global service nhưng buckets được tạo trong một region*

**Naming convention:**
- ❌ No uppercase
- ❌ No underscore
- ✅ 3-63 characters long
- ❌ Not an IP (ví dụ: 192.168.1.1)
- ✅ Must start with lowercase letter hoặc number
- ❌ Must NOT start with prefix `xn-`
- ❌ Must NOT end with suffix `-s3alias`

**💡 Example valid bucket names:**
```
my-bucket
s3-bucket-123
aws-logs-us-east-1
```

---

### Amazon S3 - Objects

> Objects (files) have a Key  
> *Objects có một Key*

> The key is the FULL path:  
> *Key là FULL path:*

```
s3://my-bucket/my_file.txt
s3://my-bucket/my_folder1/another_folder/my_file.txt
```

> The key is composed of prefix + object name  
> *Key bao gồm prefix + object name*

```
s3://my-bucket/my_folder1/another_folder/my_file.txt
│         │        │           │              │
Prefix:   │        │           │              Object name
          └────────┴───────────┘
          (prefix = my_folder1/another_folder/)
```

**Object characteristics:**

| Thành phần | Mô tả |
|-----------|--------|
| **Max Object Size** | 5TB (5000GB) |
| **Metadata** | Set of name-value pairs, thông tin về object |
| **Tags** | Unicode key/value pair - up to 10 tags |
| **Version ID** | If versioning is enabled |

---

## 3. Amazon S3 Security

### Amazon S3 - Security Overview

**Có 3 loại security:**

| Type | Mô tả |
|------|--------|
| **User-Based** | IAM Policies - cho specific user từ IAM |
| **Resource-Based** | Bucket Policies, Object ACL, Bucket ACL |
| **Encryption** | Mã hóa objects sử dụng encryption keys |

> Note: an IAM principal can access an S3 object if  
> *Note: IAM principal có thể access S3 object nếu:*

> The user IAM permissions ALLOW it OR the resource policy ALLOWS it  
> *User IAM permissions ALLOW OR resource policy ALLOWS*

> AND there's no explicit DENY  
> *VÀ không có explicit DENY*

**💡 Security Model:**
```
ALLOW (IAM) OR ALLOW (Resource Policy)
          ↓
    NO EXPLICIT DENY
          ↓
      ✓ ACCESS GRANTED
```

---

### S3 Bucket Policies

> JSON based policies  
> *Policies dựa trên JSON*

**Policy elements:**

| Element | Mô tả |
|---------|--------|
| **Effect** | Allow / Deny |
| **Principal** | Account hoặc user để apply policy |
| **Actions** | Set of API to Allow or Deny |
| **Resources** | Buckets và objects |

**Use S3 bucket for policy to:**

| Use case | Mô tả |
|----------|--------|
| **Grant public access** | Cấp quyền public access |
| **Force encryption** | Buộc objects được encrypt khi upload |
| **Cross-account access** | Cấp quyền cho account khác |

**Ví dụ Bucket Policy - Public Read:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}
```

**Ví dụ Bucket Policy - Force Encryption:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ForceEncryption",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::my-bucket/*",
      "Condition": {
        "StringNotEquals": {
          "s3:x-amz-server-side-encryption": "AES256"
        }
      }
    }
  ]
}
```

---

### S3 Security - Access Control

**Resource-Based Policies:**

| Type | Mức độ | Có thể disable? |
|------|--------|----------------|
| **Bucket Policy** | Bucket-wide | Không |
| **Object Access Control List (ACL)** | Object-level | ✅ Có |
| **Bucket Access Control List (ACL)** | Bucket-level | ✅ Có |

> Object ACL - finer grain (can be disabled)  
> *Object ACL - kiểm soát chi tiết hơn*

> Bucket ACL - less common (can be disabled)  
> *Bucket ACL - ít phổ biến hơn*

**💡 Best Practice:** Sử dụng Bucket Policies thay vì ACLs - simpler và linh hoạt hơn.

---

### Bucket settings for Block Public Access

> These settings were created to prevent company data leaks  
> *Settings này được tạo để ngăn chặn data leaks*

> If you know your bucket should never be public, leave these on  
> *Nếu bucket không bao giờ nên public, hãy để ON*

> Can be set at the account level  
> *Có thể set ở account level*

**4 settings:**
1. Block public access to buckets and objects granted through new access control lists (ACLs)
2. Block public access to buckets and objects granted through any access control lists (ACLs)
3. Block public access to buckets and objects granted through new public bucket or access point policies
4. Block public access to buckets and objects granted through any public bucket or access point policies

**💡 Khuyến nghị:** Bật Block Public Access cho tất cả accounts trừ khi cần public access cụ thể.

---

### Amazon S3 - Static Website Hosting

> S3 can host static websites and have them accessible on the Internet  
> *S3 có thể host static websites*

> If you get a 403 Forbidden error, make sure the bucket policy allows public reads!  
> *Nếu nhận 403, đảm bảo bucket policy cho phép public reads*

**Cần thiết lập:**
1. Block all Public Access: **Off**
2. Bucket policy allows public reads
3. Enable Static website hosting

**URL formats:**
```
http://bucket-name.s3-website-aws-region.amazonaws.com
http://bucket-name.s3-website.aws-region.amazonaws.com
```

**⚠️ Lưu ý:** Không dùng được với regional endpoint - phải dùng website endpoint.

---

## 4. Amazon S3 Storage Classes

### S3 Storage Classes Overview

| Storage Class | Mô tả |
|--------------|--------|
| **S3 Standard** | General Purpose - frequently accessed |
| **S3 Standard-IA** | Infrequent Access - rapid access when needed |
| **S3 One Zone-IA** | Single AZ - recreatable, infrequent |
| **S3 Glacier Instant Retrieval** | Archive - millisecond retrieval |
| **S3 Glacier Flexible Retrieval** | Archive - minutes to hours retrieval |
| **S3 Glacier Deep Archive** | Long-term archive - hours retrieval |
| **S3 Intelligent-Tiering** | Auto-optimize based on access |

> Can move between classes manually or using S3 Lifecycle configurations  
> *Có thể di chuyển giữa các classes thủ công hoặc dùng Lifecycle*

---

### S3 Durability and Availability

**Durability (Độ bền):**
> High durability (99.999999999%, 11 9's) of objects across multiple AZ  
> *Độ bền cao across multiple AZ*

> If you store 10,000,000 objects with Amazon S3, you can on average expect to incur a loss of a single object once every 10,000 years  
> *Nếu lưu 10 triệu objects, trung bình mất 1 object sau 10,000 năm*

> Same for all storage classes  
> *Giống nhau cho tất cả storage classes*

**Availability (Khả dụng):**
> Measures how readily available a service is  
> *Đo lường độ sẵn sàng của service*

> Varies depending on storage class  
> *Khác nhau tùy storage class*

> Example: S3 standard has 99.99% availability = not available 53 minutes a year  
> *Ví dụ: Standard 99.99% = downtime ~53 phút/năm*

---

### S3 Standard - General Purpose

> 99.99% Availability  
> *99.99% Availability*

> Used for frequently accessed data  
> *Dùng cho data được access thường xuyên*

> Low latency and high throughput  
> *Độ trễ thấp và throughput cao*

> Sustain 2 concurrent facility failures  
> *Chịu được 2 facility failures đồng thời*

**Use Cases:**
- Big Data analytics
- Mobile & gaming applications
- Content distribution
- Data lakes

---

### S3 Storage Classes - Infrequent Access

> For data that is less frequently accessed, but requires rapid access when needed  
> *Cho data ít access nhưng cần rapid access khi cần*

> Lower cost than S3 Standard  
> *Chi phí thấp hơn Standard*

**S3 Standard-Infrequent Access (S3 Standard-IA):**
> 99.9% Availability  
> *99.9% Availability*

> Use cases: Disaster Recovery, backups  
> *Use cases: DR, backups*

**S3 One Zone-Infrequent Access (S3 One Zone-IA):**
> High durability (99.999999999%) in a single AZ  
> *Độ bền cao trong một AZ*

> Data lost when AZ is destroyed  
> *Data bị mất khi AZ bị phá hủy*

> 99.5% Availability  
> *99.5% Availability*

> Use Cases: Storing secondary backup copies of on-premises data, or data you can recreate  
> *Use cases: Backup copies có thể recreate*

---

### Amazon S3 Glacier Storage Classes

> Low-cost object storage meant for archiving / backup  
> *Storage chi phí thấp cho archiving/backup*

> Pricing: price for storage + object retrieval cost  
> *Giá = storage + retrieval cost*

**S3 Glacier Instant Retrieval:**
> Millisecond retrieval, great for data accessed once a quarter  
> *Retrieval milliseconds, tốt cho data access 1 lần/quý*

> Minimum storage duration of 90 days  
> *Minimum 90 ngày*

**S3 Glacier Flexible Retrieval:**
> Formerly Amazon S3 Glacier  
> *Tên cũ: Amazon S3 Glacier*

| Retrieval Type | Thời gian | Phí |
|---------------|-----------|-----|
| **Expedited** | 1-5 minutes | Có phí |
| **Standard** | 3-5 hours | Miễn phí |
| **Bulk** | 5-12 hours | Miễn phí |

> Minimum storage duration of 90 days  
> *Minimum 90 ngày*

**S3 Glacier Deep Archive:**
> For long term storage  
> *Cho lưu trữ dài hạn*

| Retrieval Type | Thời gian |
|---------------|-----------|
| **Standard** | 12 hours |
| **Bulk** | 48 hours |

> Minimum storage duration of 180 days  
> *Minimum 180 ngày*

---

### S3 Intelligent-Tiering

> Small monthly monitoring and auto-tiering fee  
> *Phí monitoring nhỏ hàng tháng*

> Moves objects automatically between Access Tiers based on usage  
> *Di chuyển objects tự động giữa các tiers dựa trên usage*

> There are no retrieval charges in S3 Intelligent-Tiering  
> *Không có retrieval charges*

**Access Tiers:**

| Tier | Kích hoạt | Mô tả |
|------|-----------|--------|
| **Frequent Access** | Mặc định | Access thường xuyên |
| **Infrequent Access** | 30 days không access | Objects ít access |
| **Archive Instant Access** | 90 days không access | Objects cần archive |
| **Archive Access** | 90-700+ days (configurable) | Objects archive (optional) |
| **Deep Archive Access** | 180-700+ days (configurable) | Objects deep archive (optional) |

---

### S3 Storage Classes - Comparison Table

| Storage Class | Designed for | Min Duration | Min AZs | Key Feature |
|--------------|-------------|--------------|---------|------------|
| **S3 Standard** | Frequently accessed (>1x/month) | None | 3 | Low latency, high throughput |
| **S3 Standard-IA** | Infrequent (>1x/month) | 30 days | 3 | Lower cost, retrieval fees |
| **S3 One Zone-IA** | Recreatable, infrequent | 30 days | 1 | 40% cheaper, risk of AZ loss |
| **S3 Intelligent-Tiering** | Unknown/unpredictable patterns | None | 3 | Auto-tiering, monitoring fee |
| **S3 Glacier Instant** | Archive (quarterly access) | 90 days | 3 | Millisecond retrieval |
| **S3 Glacier Flexible** | Archive (annual access) | 90 days | 3 | Expedited/Standard/Bulk |
| **S3 Glacier Deep Archive** | Long-term archive | 180 days | 3 | Cheapest, 12-48hr retrieval |

---

## 5. Amazon S3 Advanced

### Amazon S3 - Versioning

> You can version your files in Amazon S3  
> *Có thể version files trong S3*

> It is enabled at the bucket level  
> *Được enable ở bucket level*

> Same key overwrite will change the "version": 1, 2, 3....  
> *Cùng key overwrite sẽ tạo version mới: 1, 2, 3...*

> It is best practice to version your buckets  
> *Best practice là nên version buckets*

> Preserve, retrieve, and restore every version of every object  
> *Bảo tồn, lấy, và khôi phục mọi version*

> Help you recover objects from accidental deletion or overwrite  
> *Giúp khôi phục từ accidental deletion hoặc overwrite*

**Notes:**
> Any file that is not versioned prior to enabling versioning will have version "null"  
> *File chưa versioned trước khi enable sẽ có version "null"*

> Suspending versioning does not delete the previous versions  
> *Suspending versioning không xóa các versions trước đó*

**💡 Versioning + Lifecycle = Powerful backup strategy**

---

### Amazon S3 - Replication (CRR & SRR)

> Must enable Versioning in source and destination buckets  
> *Phải enable Versioning ở cả source và destination buckets*

**Hai loại Replication:**

| Loại | Viết tắt | Mô tả |
|------|----------|--------|
| **Cross-Region Replication** | CRR | Replication across regions |
| **Same-Region Replication** | SRR | Replication trong cùng region |

> Buckets can be in different AWS accounts  
> *Buckets có thể ở different accounts*

> Copying is asynchronous  
> *Copying là asynchronous*

> Must give proper IAM permissions to S3  
> *Phải cấp proper IAM permissions cho S3*

**Use cases:**

| Loại | Use cases |
|------|-----------|
| **CRR** | Compliance, lower latency access, cross-account replication |
| **SRR** | Log aggregation, live replication between production and test accounts |

---

### Amazon S3 - Replication Notes

> After you enable Replication, only new objects are replicated  
> *Sau khi enable, chỉ objects mới được replicate*

> Optionally, you can replicate existing objects using S3 Batch Replication  
> *Có thể replicate existing objects dùng S3 Batch Replication*

> Replicates existing objects and objects that failed replication  
> *Replicates existing và objects failed replication*

**DELETE operations:**
> Can replicate delete markers from source to target (optional setting)  
> *Có thể replicate delete markers (optional)*

> Deletions with a version ID are not replicated (to avoid malicious deletes)  
> *Deletions với version ID không được replicate*

**⚠️ Important:**
> There is no "chaining" of replication  
> *Không có "chaining" của replication*

> If bucket 1 has replication into bucket 2, which has replication into bucket 3  
> *Nếu bucket 1 → bucket 2 → bucket 3*

> Then objects created in bucket 1 are not replicated to bucket 3  
> *Objects trong bucket 1 không được replicate đến bucket 3*

---

### Amazon S3 - Lifecycle Rules

**Hai loại Lifecycle Actions:**

| Action | Mô tả | Ví dụ |
|--------|--------|-------|
| **Transition Actions** | Configure objects to transition to another storage class | Move to Standard IA sau 30 days |
| **Expiration Actions** | Configure objects to expire (delete) after some time | Delete log files sau 365 days |

**Transition Actions examples:**
> Move objects to Standard IA class 60 days after creation  
> *Di chuyển đến Standard IA 60 ngày sau khi tạo*

> Move to Glacier for archiving after 6 months  
> *Di chuyển đến Glacier sau 6 tháng*

**Expiration Actions examples:**
> Access log files can be set to delete after a 365 days  
> *Log files được xóa sau 365 ngày*

> Can be used to delete old versions of files (if versioning is enabled)  
> *Xóa old versions (nếu versioning enabled)*

> Can be used to delete incomplete Multi-Part uploads  
> *Xóa incomplete multipart uploads*

**Targeting:**
> Rules can be created for a certain prefix (example: s3://mybucket/mp3/*)  
> *Rules cho một prefix cụ thể*

> Rules can be created for certain objects Tags (example: Department: Finance)  
> *Rules cho certain tags*

---

### Amazon S3 - Lifecycle Rules Scenarios

**Scenario 1: Image Thumbnails**

> Your application on EC2 creates images thumbnails after profile photos are uploaded to Amazon S3. These thumbnails can be easily recreated, and only need to be kept for 60 days. The source images should be able to be immediately retrieved for these 60 days, and afterwards, the user can wait up to 6 hours.

**Solution:**
```
Source images: S3 Standard → Lifecycle → S3 Glacier (after 60 days)
Thumbnails: S3 One Zone-IA → Lifecycle → Expire (after 60 days)
```

**Scenario 2: Object Recovery**

> A rule in your company states that you should be able to recover your deleted S3 objects immediately for 30 days, although this may happen rarely. After this time, and for up to 365 days, deleted objects should be recoverable within 48 hours.

**Solution:**
```
1. Enable S3 Versioning (deleted objects = delete markers)
2. Transition noncurrent versions → S3 Standard-IA (after 30 days)
3. Transition noncurrent versions → S3 Glacier Deep Archive (after 365 days)
```

---

### S3 Analytics - Storage Class Analysis

> Help you decide when to transition objects to the right storage class  
> *Giúp quyết định khi nào transition objects*

> Recommendations for Standard and Standard IA  
> *Recommendations cho Standard và Standard IA*

> Does NOT work for One-Zone IA or Glacier  
> *Không hoạt động với One-Zone IA hoặc Glacier*

> Report is updated daily  
> *Report được update hàng ngày*

> 24 to 48 hours to start seeing data analysis  
> *24-48 giờ để bắt đầu thấy data*

> Good first step to put together Lifecycle Rules  
> *Bước đầu tiên tốt để tạo Lifecycle Rules*

**💡 Sử dụng S3 Analytics trước khi tạo Lifecycle Rules để đưa ra quyết định dựa trên data.**

---

### S3 Event Notifications

> S3:ObjectCreated, S3:ObjectRemoved, S3:ObjectRestore, S3:Replication...  
> *Các loại events có thể notify*

> Object name filtering possible (*.jpg)  
> *Có thể filter theo object name*

> Use case: generate thumbnails of images uploaded to S3  
> *Use case: tạo thumbnails khi upload images*

> Can create as many "S3 events" as desired  
> *Có thể tạo nhiều S3 events*

> S3 event notifications typically deliver events in seconds but can sometimes take a minute or longer  
> *Thường deliver trong seconds, đôi khi lâu hơn*

**Destinations:**
- SNS Topic
- SQS Queue
- Lambda Function

**💡 Event Notifications architecture:**
```
Upload File → S3 Event → SNS/SQS/Lambda → Process (e.g., thumbnail)
```

---

### S3 Event Notifications with Amazon EventBridge

> Advanced filtering options with JSON rules (metadata, object size, name...)  
> *Advanced filtering với JSON rules*

> Multiple Destinations - ex Step Functions, Kinesis Streams / Firehose...  
> *Nhiều destinations hơn*

> EventBridge Capabilities - Archive, Replay Events, Reliable delivery  
> *EventBridge capabilities*

**Khi nào dùng EventBridge thay vì S3 Event Notifications:**

| Feature | S3 Events | EventBridge |
|---------|----------|-------------|
| Filtering | Prefix/Suffix | Metadata, size, custom |
| Destinations | SNS, SQS, Lambda | Many more (Step Functions, Kinesis, etc.) |
| Archive/Replay | ❌ | ✅ |
| Multiple rules | Limited | ✅ |

---

## 6. Tổng kết

### Kiến thức trọng tâm cho DVA Exam

| Chủ đề | Trọng tâm |
|--------|-----------|
| **S3 Buckets** | Naming rules, global unique name |
| **S3 Objects** | Key = prefix + object name, max 5TB |
| **S3 Security** | IAM, Bucket Policies, ACLs, Block Public Access |
| **S3 Versioning** | Versions, null version, MFA Delete |
| **S3 Replication** | CRR, SRR, batch replication, no chaining |
| **S3 Storage Classes** | Use cases, costs, retrieval times |
| **S3 Lifecycle** | Transition, Expiration, prefix/tags |
| **S3 Event Notifications** | Events, destinations, EventBridge |

### So sánh Storage Classes

| Class | Cost | Retrieval | Use Case |
|-------|------|----------|----------|
| Standard | Highest | Instant | Frequent access |
| Standard-IA | Medium | Instant | Infrequent, need rapid |
| One Zone-IA | Low | Instant | Recreatable, infrequent |
| Intelligent-Tiering | Variable | Instant | Unknown patterns |
| Glacier Instant | Low | Milliseconds | Quarterly archive |
| Glacier Flexible | Lower | Minutes-Hours | Annual archive |
| Glacier Deep | Lowest | Hours | Long-term (7-10 years) |

### Best Practices 2026

**S3 Security:**
1. ✅ Enable Block Public Access (account level)
2. ✅ Use Bucket Policies thay vì ACLs
3. ✅ Enable versioning cho backup
4. ✅ Enable encryption (SSE-S3 hoặc SSE-KMS)
5. ✅ Use lifecycle policies để tự động hóa transitions

**S3 Cost Optimization:**
1. ✅ Use Intelligent-Tiering cho unknown access patterns
2. ✅ Transition infrequent data đến Standard-IA/Glacier
3. ✅ Dùng S3 Analytics để đưa ra quyết định dựa trên data
4. ✅ Clean up incomplete multipart uploads

**S3 Performance:**
1. ✅ Use S3 Transfer Acceleration cho global uploads
2. ✅ Use prefix partitioning cho better performance (bucket/folder/date/)
3. ✅ Consider S3 Select để filter data at source
4. ✅ Use multipart upload cho files > 100MB

**S3 Durability & Availability:**
1. ✅ Cross-Region Replication cho DR
2. ✅ Same-Region Replication cho backup/log aggregation
3. ✅ Versioning + Lifecycle cho data protection
4. ✅ S3 Object Lock (WORM) cho compliance

---

## Liên kết tham khảo

- [Amazon S3 Documentation](https://docs.aws.amazon.com/s3/)
- [S3 Storage Classes](https://docs.aws.amazon.com/AmazonS3/latest/userguide/storage-class-intro.html)
- [S3 Replication](https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication.html)
- [S3 Lifecycle Configuration](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lifecycle-mgmt.html)

---

*Document này được tạo để hỗ trợ học tập cho kỳ thi AWS Certified Developer - Associate*
