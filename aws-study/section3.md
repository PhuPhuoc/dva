# Section 3: RDS, Aurora & ElastiCache, Route53

**AWS Certified Developer - Associate**  
*Nguồn: Cloud Mentor Pro - Cập nhật: 15/02/2026*

---

## Mục lục

1. [Amazon RDS](#1-amazon-rds)
2. [Amazon Aurora](#2-amazon-aurora)
3. [Amazon ElastiCache](#3-amazon-elasticache)
4. [Amazon Route 53](#4-amazon-route-53)
5. [Tổng kết](#5-tổng-kết)

---

## 1. Amazon RDS

### Amazon RDS Overview (Tổng quan RDS)

> RDS stands for Relational Database Service  
> *RDS = Relational Database Service (Dịch vụ Cơ sở dữ liệu Quan hệ)*

> It's a managed DB service for DB use SQL as a query language  
> *Là managed DB service dùng SQL như query language*

> It allows you to create databases in the cloud that are managed by AWS  
> *Cho phép tạo databases trong cloud được quản lý bởi AWS*

**Các database engines được hỗ trợ:**

| Engine | Mô tả |
|--------|--------|
| **PostgreSQL** | Open-source, enterprise-grade |
| **MySQL** | Open-source, popular |
| **MariaDB** | MySQL-compatible, open-source |
| **Oracle** | Commercial, enterprise |
| **Microsoft SQL Server** | Commercial, Windows-centric |
| **IBM DB2** | Commercial, IBM |
| **Aurora** | AWS Proprietary (PostgreSQL/MySQL compatible) |

---

### Advantage over using RDS versus deploying DB on EC2 (Ưu điểm của RDS so với DB trên EC2)

> RDS is a managed service  
> *RDS là managed service*

**Lợi ích của RDS:**

| Tính năng | Mô tả |
|-----------|--------|
| **Automated provisioning** | Tự động cung cấp |
| **OS patching** | Tự động patch OS |
| **Continuous backups** | Backup liên tục |
| **Point in Time Restore** | Khôi phục đến bất kỳ thời điểm nào |
| **Monitoring dashboards** | Dashboard giám sát |
| **Read replicas** | Replicas để cải thiện read performance |
| **Multi AZ setup** | Thiết lập Multi AZ cho DR |
| **Maintenance windows** | Cửa sổ bảo trì cho upgrades |
| **Scaling capability** | Khả năng scale (vertical và horizontal) |
| **Storage backed by EBS** | Storage backed by EBS (gp2 hoặc io1) |

---

### RDS - Storage Auto Scaling

> Helps you increase storage on your RDS DB instance dynamically  
> *Giúp tăng storage trên RDS DB instance một cách linh hoạt*

> When RDS detects you are running out of free database storage, it scales automatically  
> *Khi RDS phát hiện sắp hết storage, nó tự động scale*

**Điều kiện tự động modify storage:**

| Điều kiện | Chi tiết |
|-----------|----------|
| Free storage < 10% | Dưới 10% allocated storage |
| Low-storage lasts ≥ 5 minutes | Low-storage kéo dài ít nhất 5 phút |
| 6 hours passed | Đã 6 giờ kể từ lần modify cuối |

> Useful for applications with unpredictable workloads  
> *Hữu ích cho ứng dụng có workload không thể đoán trước*

> Supports all RDS database engines  
> *Hỗ trợ tất cả RDS database engines*

**💡 Thiết lập:**
```sql
-- Enable storage autoscaling
ALTER DATABASE mydb MODIFY STORAGE (ENABLE STORAGE AUTOSCALING = ON);
```

---

### RDS Read Replicas for Read Scalability (Read Replicas cho Read Scalability)

> Up to 15 Read Replicas  
> *Lên đến 15 Read Replicas*

> Within AZ, Cross AZ or Cross Region  
> *Trong cùng AZ, Cross AZ, hoặc Cross Region*

> Replication is ASYNC, so reads are eventually consistent  
> *Replication là ASYNC, reads cuối cùng sẽ consistent*

> Replicas can be promoted to their own DB  
> *Replicas có thể được promote thành DB riêng*

> Applications must update the connection string to leverage read replicas  
> *Applications phải update connection string để tận dụng read replicas*

**Use cases:**
> You have a production database that is taking on normal load  
> *Bạn có production database đang xử lý load bình thường*

> You want to run a reporting application to run some analytics  
> *Bạn muốn chạy reporting application để phân tích*

> You create a Read Replica to run the new workload there  
> *Tạo Read Replica để chạy workload mới*

> The production application is unaffected  
> *Production application không bị ảnh hưởng*

> Read replicas are used for SELECT (=read) only kind of statements (not INSERT, UPDATE, DELETE)  
> *Read replicas chỉ dùng cho SELECT (đọc), không dùng cho INSERT, UPDATE, DELETE*

**💡 Exam Tip:** Replication là ASYNC → eventual consistency. Writes không bao giờ đi qua replicas.

---

### RDS Read Replicas - Network Cost (Chi phí Network)

> In AWS there's a network cost when data goes from one AZ to another  
> *Có network cost khi data đi từ AZ này sang AZ khác*

> For RDS Read Replicas within the same region, you don't pay that fee  
> *Cho Read Replicas trong cùng region, không mất phí*

**💡 Tối ưu chi phí:** Tạo Read Replicas trong cùng region để tránh cross-AZ data transfer fees.

---

### RDS Multi AZ (Disaster Recovery)

> SYNC replication  
> *Replication đồng bộ (SYNC)*

> One DNS name - automatic app failover to standby  
> *Một DNS name - tự động failover đến standby*

> Increase availability  
> *Tăng availability*

> Failover in case of loss of AZ, loss of network, instance or storage failure  
> *Failover khi mất AZ, network, instance hoặc storage failure*

> No manual intervention in apps  
> *Không cần can thiệp thủ công vào apps*

> Not used for scaling  
> *Không dùng để scale*

**💡 Khác biệt quan trọng:**
| | Read Replicas | Multi-AZ |
|--|--------------|----------|
| **Replication** | ASYNC | SYNC |
| **Mục đích** | Scale reads | HA/DR |
| **Standby** | Có thể serve reads | Chỉ standby, không serve reads |

> Note: The Read Replicas can be setup as Multi AZ for Disaster Recovery (DR)  
> *Note: Read Replicas có thể được setup như Multi AZ cho DR*

---

### RDS - From Single-AZ to Multi-AZ

> Zero downtime operation (no need to stop the DB)  
> *Không downtime (không cần stop DB)*

> Just click on "modify" for the database  
> *Chỉ cần click "modify"*

**Quy trình internal:**
1. Snapshot được tạo
2. DB mới được restore từ snapshot trong AZ mới
3. Synchronization được thiết lập giữa hai databases

---

## 2. Amazon Aurora

### Amazon Aurora Overview

> Aurora is a proprietary technology from AWS (not open sourced)  
> *Aurora là công nghệ độc quyền của AWS (không open source)*

> Postgres and MySQL are both supported as Aurora DB  
> *Hỗ trợ cả Postgres và MySQL*

> Aurora is "AWS cloud optimized" and claims 5x performance improvement over MySQL on RDS, over 3x the performance of Postgres on RDS  
> *Aurora được tối ưu cho AWS cloud, cải thiện 5x performance so với MySQL trên RDS, 3x so với Postgres trên RDS*

> Aurora storage automatically grows in increments of 10GB, up to 128 TB  
> *Aurora storage tự động tăng 10GB một lần, lên đến 128 TB*

> Aurora can have up to 15 replicas and the replication process is faster than MySQL (sub 10 ms replica lag)  
> *Aurora có thể lên đến 15 replicas với replication nhanh hơn MySQL (dưới 10ms replica lag)*

> Failover in Aurora is instantaneous. It's HA (High Availability) native  
> *Failover trong Aurora là tức thì. HA được tích hợp sẵn*

> Aurora costs more than RDS (20% more) - but is more efficient  
> *Aurora đắt hơn RDS (~20%) - nhưng hiệu quả hơn*

---

### Aurora High Availability and Read Scaling (HA và Read Scaling)

> 6 copies of your data across 3 AZ:  
> *6 copies của data trên 3 AZ:*

> 4 copies out of 6 needed for writes  
> *4/6 copies cần cho writes*

> 3 copies out of 6 need for reads  
> *3/6 copies cần cho reads*

> Self healing with peer-to-peer replication  
> *Tự chữa lành với peer-to-peer replication*

> Storage is striped across 100s of volumes  
> *Storage được stripe across hàng trăm volumes*

> One Aurora Instance takes writes (master)  
> *Một Aurora Instance nhận writes (master)*

> Automated failover for master in less than 30 seconds  
> *Tự động failover master trong vòng 30 giây*

> Master + up to 15 Aurora Read Replicas serve reads  
> *Master + đến 15 Aurora Read Replicas phục vụ reads*

> Support for Cross Region Replication  
> *Hỗ trợ Cross Region Replication*

**💡 Aurora Cluster Architecture:**
```
Writer Endpoint → Master (Primary Instance)
       ↓
Aurora Replicas (up to 15) → Reader Endpoint
       ↓
Shared Storage (6 copies across 3 AZ)
```

---

### Features of Aurora

| Tính năng | Mô tả |
|-----------|--------|
| **Automatic fail-over** | Tự động failover |
| **Backup and Recovery** | Backup và khôi phục |
| **Isolation and security** | Cô lập và bảo mật |
| **Industry compliance** | Tuân thủ tiêu chuẩn ngành |
| **Push-button scaling** | Scale chỉ bằng nút bấm |
| **Automated Patching with Zero Downtime** | Patch tự động không downtime |
| **Advanced Monitoring** | Giám sát nâng cao |
| **Routine Maintenance** | Bảo trì định kỳ |
| **Backtrack** | Khôi phục data tại bất kỳ thời điểm nào mà không cần backup |

**💡 Backtrack độc đáo của Aurora:** Cho phép rewind database về trạng thái trước đó trong vài phút mà không cần restore từ backup.

---

### RDS & Aurora Security (Bảo mật)

**At-rest encryption (Mã hóa data at rest):**

| Yếu tố | Mô tả |
|---------|--------|
| **KMS** | Database master & replicas encryption sử dụng AWS KMS |
| **Launch time** | Phải được định nghĩa tại launch time |
| **Unencrypted DB** | Nếu master không encrypt, replicas không thể encrypt |
| **Encrypt unencrypted** | Snapshot & restore as encrypted |

**In-flight encryption:**
> TLS-ready by default, use the AWS TLS root certificates client-side  
> *TLS-ready mặc định, sử dụng AWS TLS root certificates phía client*

**IAM Authentication:**
> IAM roles to connect to your database (instead of username/pw)  
> *Dùng IAM roles để kết nối database thay vì username/password*

**Security Groups:**
> Control Network access to your RDS / Aurora DB  
> *Kiểm soát network access đến RDS/Aurora*

**SSH:**
> No SSH available except on RDS Custom  
> *Không có SSH ngoại trừ RDS Custom*

**Audit Logs:**
> Audit Logs can be enabled and sent to CloudWatch Logs for longer retention  
> *Có thể enable Audit Logs và gửi đến CloudWatch Logs*

---

### Amazon RDS Proxy

> Fully managed database proxy for RDS  
> *Database proxy được quản lý hoàn toàn cho RDS*

> Allows apps to pool and share DB connections established with the database  
> *Cho phép apps pool và share DB connections*

> Improving database efficiency by reducing the stress on database resources (e.g., CPU, RAM) and minimize open connections (and timeouts)  
> *Cải thiện database efficiency bằng cách giảm stress trên resources*

> Serverless, autoscaling, highly available (multi-AZ)  
> *Serverless, autoscaling, HA (multi-AZ)*

> Reduced RDS & Aurora failover time by up 66%  
> *Giảm failover time đến 66%*

> Supports RDS (MySQL, PostgreSQL, MariaDB, MS SQL Server) and Aurora (MySQL, PostgreSQL)  
> *Hỗ trợ nhiều engines*

> No code changes required for most apps  
> *Không cần thay đổi code cho hầu hết apps*

> Enforce IAM Authentication for DB, and securely store credentials in AWS Secrets Manager  
> *Enforce IAM Authentication, lưu credentials trong Secrets Manager*

> RDS Proxy is never publicly accessible (must be accessed from VPC)  
> *RDS Proxy không bao giờ publicly accessible*

**💡 Use case chính:** Khi có nhiều Lambda functions kết nối đồng thời → RDS Proxy giảm connection exhaustion.

---

## 3. Amazon ElastiCache

### Amazon ElastiCache Overview

> The same way RDS is to get managed Relational Databases...  
> *Cũng như RDS cho managed Relational Databases...*

> ElastiCache is to get managed Redis or Memcached  
> *ElastiCache là managed Redis hoặc Memcached*

> Caches are in-memory databases with really high performance, low latency  
> *Caches là in-memory databases với hiệu năng cao, độ trễ thấp*

> Helps reduce load off of databases for read intensive workloads  
> *Giúp giảm load trên databases cho read-intensive workloads*

> Helps make your application stateless  
> *Giúp ứng dụng stateless*

> AWS takes care of OS maintenance / patching, optimizations, setup, configuration, monitoring, failure recovery and backups  
> *AWS lo OS maintenance, patching, optimizations, setup, configuration, monitoring, failure recovery, backups*

> Using ElastiCache involves heavy application code changes  
> *Việc sử dụng ElastiCache đòi hỏi thay đổi code ứng dụng đáng kể*

**💡 ElastiCache KHÔNG tự động cache - phải implement logic cache trong code.**

---

### ElastiCache Solution Architecture - DB Cache

> Applications queries ElastiCache, if not available, get from RDS and store in ElastiCache.  
> *Application query ElastiCache trước, nếu không có thì lấy từ RDS và store vào ElastiCache*

> Helps relieve load in RDS  
> *Giúp giảm load trên RDS*

> Cache must have an invalidation strategy to make sure only the most current data is used in there.  
> *Cache phải có invalidation strategy để đảm bảo chỉ data mới nhất được dùng*

**Flow:**
```
App → ElastiCache (check) → Hit? → Return data
                    ↓ Miss
                    RDS (fetch) → Store in Cache → Return data
```

---

### ElastiCache Solution Architecture - User Session Store

> User logs into any of the application  
> *User đăng nhập vào ứng dụng*

> The application writes the session data into ElastiCache  
> *Application ghi session data vào ElastiCache*

> The user hits another instance of our application  
> *User truy cập instance khác của ứng dụng*

> The instance retrieves the data and the user is already logged in  
> *Instance lấy data và user đã logged in*

**💡 Use case:** Khi có nhiều EC2 instances behind Load Balancer → Sessions được share qua ElastiCache → User không bị logout khi chuyển instance.

---

### ElastiCache - Redis vs Memcached

| Feature | Redis | Memcached |
|---------|-------|-----------|
| **Replication** | Multi AZ with Auto-Failover | Multi-node for partitioning (sharding) |
| **High Availability** | ✅ Có | ❌ Không |
| **Data Durability** | AOF persistence | ❌ Non-persistent |
| **Backup/Restore** | ✅ Có | ❌ Không |
| **Data Types** | Sets, Sorted Sets | Simple key-value |
| **Architecture** | Single-threaded | Multi-threaded |
| **Use case** | Complex data, persistence | Simple caching, high throughput |

**Redis:**
> Multi AZ with Auto-Failover  
> *Multi AZ với Auto-Failover*

> Read Replicas to scale reads and have high availability  
> *Read Replicas để scale reads và có HA*

> Data Durability using AOF persistence  
> *Data Durability dùng AOF persistence*

> Backup and restore features  
> *Có backup và restore*

> Supports Sets and Sorted Sets  
> *Hỗ trợ Sets và Sorted Sets*

**Memcached:**
> Multi-node for partitioning of data (sharding)  
> *Multi-node để partition data (sharding)*

> No high availability (replication)  
> *Không có HA*

> Non persistent  
> *Không persistent*

> No backup and restore  
> *Không có backup/restore*

> Multi-threaded architecture  
> *Architecture đa luồng*

**💡 Khi nào dùng Redis vs Memcached:**
- **Redis:** Cần persistence, HA, complex data structures, sorted sets
- **Memcached:** Cần simple caching, multi-core performance, sharding đơn giản

---

### Caching Implementation Considerations

> Is it safe to cache data? Data may be out of date, eventually consistent  
> *Có an toàn khi cache data? Data có thể outdated, eventually consistent*

> Is caching effective for that data?  
> *Caching có hiệu quả cho data đó không?*

**Patterns:**
- ✅ **Good pattern:** Data thay đổi chậm, vài keys được access thường xuyên
- ❌ **Anti pattern:** Data thay đổi nhanh, toàn bộ key space được access thường xuyên

> Is data structured well for caching?  
> *Data có cấu trúc tốt cho caching không?*

> Which caching design pattern is the most appropriate?  
> *Pattern caching nào phù hợp nhất?*

---

### Lazy Loading / Cache-Aside / Lazy Population

> Only requested data is cached (the cache isn't filled up with unused data)  
> *Chỉ data được request mới được cache*

> Node failures are not fatal (just increased latency to warm the cache)  
> *Node failures không fatal (chỉ tăng latency để warm cache)*

**Pros:**
- ✅ Chỉ cache data được request
- ✅ Node failure không gây fatal error

**Cons:**
> Cache miss penalty that results in 3 round trips, noticeable delay for that request  
> *Cache miss gây ra 3 round trips, delay đáng kể*

> Stale data: data can be updated in the database and outdated in the cache  
> *Stale data: data được update trong DB nhưng outdated trong cache*

**Pseudocode:**
```python
def get_user(user_id):
    # Check cache first
    cached = cache.get(f"user:{user_id}")
    if cached:
        return cached
    
    # Cache miss - get from DB
    user = db.query(f"SELECT * FROM users WHERE id = {user_id}")
    
    # Store in cache
    cache.set(f"user:{user_id}", user, ttl=300)
    return user
```

---

### Write Through

> Data in cache is never stale, reads are quick  
> *Data trong cache không bao giờ stale, reads nhanh*

> Write penalty vs Read penalty (each write requires 2 calls)  
> *Write penalty: mỗi write cần 2 calls*

**Pros:**
- ✅ Data không bao giờ stale
- ✅ Reads nhanh

**Cons:**
> Missing Data until it is added / updated in the DB. Mitigation is to implement Lazy Loading strategy as well  
> *Missing data cho đến khi được add/update. Có thể implement Lazy Loading*

> Cache churn - a lot of the data will never be read  
> *Cache churn - nhiều data sẽ không bao giờ được đọc*

**Pseudocode:**
```python
def update_user(user_id, data):
    # Write to DB
    db.update(f"UPDATE users SET ... WHERE id = {user_id}")
    
    # Write to cache immediately
    cache.set(f"user:{user_id}", data, ttl=300)
```

---

### Cache Evictions and Time-to-live (TTL)

**Cache eviction xảy ra 3 cách:**

| Cách | Mô tả |
|------|--------|
| **Explicit delete** | Xóa item một cách tường minh |
| **LRU** | Item bị evict vì memory đầy và không được sử dụng gần đây |
| **TTL** | Item hết hạn sau TTL |

**TTL hữu ích cho:**
- Leaderboards
- Comments
- Activity streams

> TTL can range from few seconds to hours or days  
> *TTL có thể từ vài giây đến vài giờ hoặc ngày*

> If too many evictions happen due to memory, you should scale up or out  
> *Nếu quá nhiều evictions do memory, nên scale up hoặc out*

---

### Final Words of Wisdom

> Lazy Loading / Cache aside is easy to implement and works for many situations as a foundation, especially on the read side  
> *Lazy Loading dễ implement và hoạt động cho nhiều situations, đặc biệt cho read side*

> Write-through is usually combined with Lazy Loading as targeted for the queries or workloads that benefit from this optimization  
> *Write-through thường được kết hợp với Lazy Loading cho workloads được hưởng lợi*

> Setting a TTL is usually not a bad idea, except when you're using Write-through. Set it to a sensible value for your application  
> *Setting TTL thường là ý hay, ngoại trừ khi dùng Write-through*

> Only cache the data that makes sense (user profiles, blogs, etc...)  
> *Chỉ cache data hợp lý (user profiles, blogs, etc.)*

---

## 4. Amazon Route 53

### What is DNS? (DNS là gì?)

> Domain Name System which translates the human friendly hostnames into the machine IP addresses  
> *DNS (Domain Name System) dịch hostname thân thiện sang IP addresses*

> www.google.com => 172.217.18.36

> DNS is the backbone of the Internet  
> *DNS là backbone của Internet*

> DNS uses hierarchical naming structure  
> *DNS sử dụng hierarchical naming structure*

```
.com (TLD)
  └── example.com (SLD)
        ├── www.example.com
        └── api.example.com
```

---

### DNS Terminologies

| Thuật ngữ | Mô tả | Ví dụ |
|-----------|--------|-------|
| **Domain Registrar** | Nơi đăng ký domain | Amazon Route 53, GoDaddy |
| **DNS Records** | Các bản ghi DNS | A, AAAA, CNAME, NS |
| **Zone File** | File chứa DNS records | - |
| **Name Server** | Server resolve DNS queries | Authoritative/Non-Authoritative |
| **Top Level Domain (TLD)** | Level cao nhất | .com, .us, .gov, .org |
| **Second Level Domain (SLD)** | Level thứ 2 | amazon.com, google.com |

---

### Amazon Route 53 Overview

> A highly available, scalable, fully managed and Authoritative DNS  
> *DNS Authoritative được quản lý hoàn toàn, có tính khả dụng cao và scalable*

> Authoritative = the customer (you) can update the DNS records  
> *Authoritative = bạn có thể update DNS records*

> Route 53 is also a Domain Registrar  
> *Route 53 cũng là Domain Registrar*

> Ability to check the health of your resources  
> *Khả năng kiểm tra health của resources*

> The only AWS service which provides 100% availability SLA  
> *Dịch vụ AWS duy nhất có 100% availability SLA*

---

### Route 53 - Records

> How you want to route traffic for a domain  
> *Cách bạn muốn route traffic cho một domain*

**Mỗi record chứa:**

| Thành phần | Ví dụ | Mô tả |
|-----------|--------|--------|
| **Domain/subdomain Name** | example.com | Tên domain/subdomain |
| **Record Type** | A, AAAA | Loại record |
| **Value** | 12.34.56.78 | Giá trị |
| **Routing Policy** | Simple, Weighted | Cách Route 53 respond |
| **TTL** | 300 | Thời gian record được cached |

**Record types cần biết:**
> (must know) A / AAAA / CNAME / NS  
> *Phải biết*

> (advanced) CAA / DS / MX / NAPTR / PTR / SOA / TXT / SPF / SRV  
> *Nâng cao*

---

### Route 53 - Record Types

**A Record:**
> A - maps a hostname to IPv4  
> *Map hostname đến IPv4*

**AAAA Record:**
> AAAA - maps a hostname to IPv6  
> *Map hostname đến IPv6*

**CNAME Record:**
> CNAME - maps a hostname to another hostname  
> *Map hostname đến hostname khác*

> The target is a domain name which must have an A or AAAA record  
> *Target phải là domain có A hoặc AAAA record*

> Can't create a CNAME record for the top node of a DNS namespace (Zone Apex)  
> *Không thể tạo CNAME cho Zone Apex*

> Example: you can't create for example.com, but you can create for www.example.com  
> *Ví dụ: không thể tạo cho example.com, nhưng có thể cho www.example.com*

**NS Record:**
> NS - Name Servers for the Hosted Zone  
> *Name Servers cho Hosted Zone*

> Control how traffic is routed for a domain  
> *Kiểm soát cách traffic được route*

> Define which organization manages your domain  
> *Định nghĩa tổ chức quản lý domain*

---

### Route 53 - Hosted Zones

> A container for records that define how to route traffic to a domain and its subdomains  
> *Container cho records định nghĩa cách route traffic*

**Hai loại Hosted Zones:**

| Loại | Mô tả | Ví dụ |
|------|--------|-------|
| **Public Hosted Zones** | Route traffic on the Internet | application1.mypublicdomain.com |
| **Private Hosted Zones** | Route traffic within VPC(s) | application1.company.internal |

> You pay $0.50 per month per hosted zone  
> *Phí $0.50/tháng/hosted zone*

---

### Route 53 - Records TTL (Time To Live)

**High TTL (e.g., 24 hr):**
> Less traffic on Route 53  
> *Ít traffic trên Route 53*

> Possibly outdated records  
> *Records có thể outdated*

**Low TTL (e.g., 60 sec):**
> More traffic on Route 53 ($$)  
> *Nhiều traffic trên Route 53 (tốn phí)*

> Records are outdated for less time  
> *Records ít outdated hơn*

> Easy to change records  
> *Dễ dàng thay đổi records*

> Except for Alias records, TTL is mandatory for each DNS record  
> *Ngoại trừ Alias records, TTL bắt buộc cho mỗi DNS record*

**💡 Best practice:** Dùng low TTL (60-300s) khi cần thay đổi nhanh, high TTL (24h) để giảm chi phí khi ổn định.

---

### CNAME vs Alias

**Scenario:** AWS Resources expose hostname như `lb1-1234.us-east-2.elb.amazonaws.com`, và bạn muốn route `myapp.mydomain.com` đến đó.

**CNAME:**
> Points a hostname to any other hostname  
> *Points một hostname đến hostname khác*

> ONLY FOR NON ROOT DOMAIN (aka. something.mydomain.com)  
> *CHỈ cho NON ROOT DOMAIN*

**Alias:**
> Points a hostname to an AWS Resource  
> *Points đến AWS Resource*

> Works for ROOT DOMAIN and NON ROOT DOMAIN  
> *Hoạt động cho cả ROOT và NON ROOT DOMAIN*

> Free of charge  
> *Miễn phí*

> Native health check  
> *Có native health check*

**So sánh:**

| Feature | CNAME | Alias |
|---------|-------|-------|
| Root domain (example.com) | ❌ | ✅ |
| Non-root (www.example.com) | ✅ | ✅ |
| AWS Resource | ❌ | ✅ |
| Multiple targets | ✅ | ❌ |
| Phí | Có thể có | Miễn phí |
| Health check | ❌ | ✅ |

**💡 Exam Tip:** Alias record chỉ hoạt động cho AWS resources (ELB, CloudFront, S3, etc.)

---

### Route 53 - Alias Records

> Maps a hostname to an AWS resource  
> *Map hostname đến AWS resource*

> Automatically recognizes changes in the resource's IP addresses  
> *Tự động nhận biết thay đổi IP của resource*

> Unlike CNAME, it can be used for the top node e.g.: example.com (ROOT DOMAIN)  
> *Khác với CNAME, có thể dùng cho ROOT DOMAIN*

> Alias Record is always of type A/AAAA for AWS resources (IPv4 / IPv6)  
> *Alias Record luôn là type A/AAAA cho AWS resources*

> You can't set the TTL  
> *Không thể set TTL*

---

### Route 53 - Routing Policies

> Define how Route 53 responds to DNS queries  
> *Định nghĩa cách Route 53 respond DNS queries*

> Don't get confused by the word "Routing"  
> *Đừng nhầm lẫn với "Routing" của Load Balancer*

> It's not the same as Load balancer routing which routes the traffic  
> *DNS routing không giống Load Balancer routing*

> DNS does not route any traffic, it only responds to the DNS queries  
> *DNS không route traffic, chỉ respond DNS queries*

**Các Routing Policies:**

| Policy | Mô tả |
|--------|--------|
| **Simple** | Route đến một resource (hoặc random nếu nhiều) |
| **Weighted** | Route theo % weight |
| **Failover** | Primary + Secondary (DR) |
| **Latency based** | Route đến resource có latency thấp nhất |
| **Geolocation** | Route dựa trên vị trí user |
| **Multi-Value Answer** | Route đến nhiều healthy resources |
| **Geoproximity** | Route dựa trên vị trí địa lý (cần Traffic Flow) |

---

### Routing Policies - Simple

> Typically, route traffic to a single resource  
> *Thường route đến một resource*

> Can specify multiple values in the same record  
> *Có thể chỉ định nhiều values*

> If multiple values are returned, a random one is chosen by the client  
> *Nếu có nhiều values, client chọn random*

> When Alias enabled, specify only one AWS resource  
> *Khi dùng Alias, chỉ định một AWS resource*

> No health checks  
> *Không có health checks*

---

### Routing Policies - Weighted

> Control the % of the requests that go to each specific resource  
> *Kiểm soát % requests đến mỗi resource*

> DNS records must have the same name and type  
> *DNS records phải cùng name và type*

> Can be associated with Health Checks  
> *Có thể kết hợp với Health Checks*

> Use cases: load balancing between regions, testing new application versions...  
> *Use cases: load balancing giữa regions, testing versions mới*

> Assign a weight of 0 to a record to stop sending traffic to a resource  
> *Set weight = 0 để ngừng gửi traffic*

> If all records have weight of 0, then all records will be returned equally  
> *Nếu tất cả weight = 0, tất cả records được trả đều nhau*

**💡 Use case:** Blue-green deployment với 10% traffic đến version mới.

---

### Routing Policies - Latency-based

> Redirect to the resource that has the least latency close to us  
> *Redirect đến resource có latency thấp nhất*

> Super helpful when latency for users is a priority  
> *Rất hữu ích khi latency là ưu tiên*

> Latency is based on traffic between users and AWS Regions  
> *Latency dựa trên traffic giữa users và AWS Regions*

> Associated with Health Checks (has a failover capability)  
> *Kết hợp với Health Checks (có failover capability)*

**💡 Route 53 đo latency thực tế và chọn region nhanh nhất cho user.**

---

### Route 53 - Health Checks

> HTTP Health Checks are only for public resources  
> *HTTP Health Checks chỉ cho public resources*

**Ba loại Health Checks:**

| Loại | Mô tả |
|------|--------|
| **Monitor an endpoint** | Giám sát endpoint (application, server, AWS resource) |
| **Monitor other health checks** | Calculated Health Checks |
| **Monitor CloudWatch Alarms** | Full control - DynamoDB throttles, RDS alarms, custom metrics |

**Automated DNS Failover:**
```
Health Check fails → Route 53 tự động redirect traffic
```

**💡 Private resources:** Health checkers nằm ngoài VPC → không truy cập được private endpoints. Giải pháp: CloudWatch Alarm + Health Check.

---

### Routing Policies - Failover (Active-Passive)

> Primary resource is the main site/app  
> *Primary resource là site/app chính*

> Secondary resource is the DR site/app  
> *Secondary resource là DR site/app*

> When health check fails → Route 53 automatically fails over  
> *Khi health check fails → Route 53 tự động failover*

**Use case:** Primary region → Secondary region (DR)

---

### Routing Policies - Geolocation

> Different from Latency-based!  
> *Khác với Latency-based!*

> This routing is based on user location  
> *Routing dựa trên vị trí user*

> Specify location by Country or by US State  
> *Chỉ định location theo Country hoặc US State*

> Should create a "Default" record (in case there's no match on location)  
> *Nên tạo "Default" record*

> Use cases: website localization, restrict content distribution, load balancing, ...  
> *Use cases: localization, restrict content, load balancing*

> Can be associated with Health Checks  
> *Có thể kết hợp với Health Checks*

**💡 Geolocation vs Latency:**
- **Geolocation:** Dựa trên vị trí địa lý (quốc gia, tiểu bang)
- **Latency:** Dựa trên độ trễ mạng thực tế

---

### Routing Policies - Geoproximity

> Route traffic to your resources based on the geographic location of users and resources  
> *Route traffic dựa trên vị trí địa lý của users và resources*

> Ability to shift more traffic to resources based on the defined bias  
> *Khả năng shift traffic dựa trên bias*

**Bias values:**

| Bias | Effect |
|------|--------|
| **1 to 99** | Expand - more traffic đến resource |
| **-1 to -99** | Shrink - less traffic đến resource |

**Resources:**
> AWS resources (specify AWS region)  
> *AWS resources (chỉ định region)*

> Non-AWS resources (specify Latitude and Longitude)  
> *Non-AWS resources (chỉ định Lat/Long)*

> You must use Route 53 Traffic Flow to use this feature  
> *Phải dùng Route 53 Traffic Flow*

---

### Route 53 - Traffic Flow

> Simplify the process of creating and maintaining records in large and complex configurations  
> *Đơn giản hóa việc tạo và maintain records trong cấu hình phức tạp*

> Visual editor to manage complex routing decision trees  
> *Visual editor để quản lý routing decision trees*

> Configurations can be saved as Traffic Flow Policy  
> *Có thể lưu thành Traffic Flow Policy*

> Can be applied to different Route 53 Hosted Zones (different domain names)  
> *Có thể apply cho nhiều Hosted Zones*

> Supports versioning  
> *Hỗ trợ versioning*

---

### Routing Policies - IP-based Routing

> Routing is based on clients' IP addresses  
> *Routing dựa trên IP addresses của clients*

> You provide a list of CIDRs for your clients and the corresponding endpoints/locations (user-IP-to-endpoint mappings)  
> *Cung cấp list CIDRs và endpoints tương ứng*

> Use cases: Optimize performance, reduce network costs...  
> *Use cases: Optimize performance, giảm network costs*

> Example: route end users from a particular ISP to a specific endpoint  
> *Ví dụ: route users từ một ISP cụ thể đến endpoint cụ thể*

---

### Routing Policies - Multi-Value

> Use when routing traffic to multiple resources  
> *Dùng khi route đến nhiều resources*

> Up to 8 healthy records are returned for each Multi-Value query  
> *Trả về đến 8 healthy records*

> It's very similar to simple routing, but with two differences:  
> *Giống simple routing nhưng có 2 khác biệt:*

> Routing traffic to multiple resources  
> *Route đến nhiều resources*

> You can have health checks  
> *Có health checks*

**💡 Khác với Simple:** Simple không có health checks, Multi-Value có.

---

### Domain Registrar vs. DNS Service

> You buy or register your domain name with a Domain Registrar typically by paying annual charges  
> *Mua/đăng ký domain với Domain Registrar*

> The Domain Registrar usually provides you with a DNS service to manage your DNS records  
> *Domain Registrar thường cung cấp DNS service*

> But you can use another DNS service to manage your DNS records  
> *Nhưng có thể dùng DNS service khác*

> Example: purchase the domain from GoDaddy and use Route 53 to manage your DNS records  
> *Ví dụ: mua domain ở GoDaddy, dùng Route 53 quản lý DNS*

---

### 3rd Party Registrar with Amazon Route 53

**Các bước sử dụng Route 53 làm DNS Service:**

1. Create a Hosted Zone in Route 53
2. Update NS Records on 3rd party website to use Route 53 Name Servers

> Domain Registrar != DNS Service  
> *Domain Registrar KHÁC DNS Service*

> But every Domain Registrar usually comes with some DNS features  
> *Nhưng mỗi Registrar thường có DNS features*

---

## 5. Tổng kết

### Kiến thức trọng tâm cho DVA Exam

| Chủ đề | Trọng tâm |
|--------|-----------|
| **RDS** | Multi-AZ vs Read Replicas, Storage Auto Scaling, Encryption |
| **Aurora** | 6 copies across 3 AZ, Auto-scaling storage, Backtrack, Serverless |
| **ElastiCache** | Redis vs Memcached, Caching patterns (Lazy Loading, Write-through) |
| **Route 53** | Record types (A, AAAA, CNAME, NS, Alias), Routing Policies |

### So sánh quan trọng

**RDS vs Aurora:**

| Feature | RDS | Aurora |
|---------|-----|--------|
| Storage | Fixed (EBS) | Auto-growing (10GB increments, up to 128TB) |
| Performance | Baseline | 5x MySQL / 3x Postgres vs RDS |
| Replicas | Up to 15 | Up to 15, sub-10ms lag |
| Cost | Base | ~20% more |
| Backtrack | ❌ | ✅ |

**Read Replicas vs Multi-AZ:**

| | Read Replicas | Multi-AZ |
|--|--------------|----------|
| **Replication** | ASYNC | SYNC |
| **Purpose** | Scale reads | HA/DR |
| **Failover** | Manual promote | Automatic |
| **Serves reads** | Yes | Standby only |

**Redis vs Memcached:**

| | Redis | Memcached |
|--|-------|-----------|
| **HA** | ✅ Multi-AZ | ❌ |
| **Persistence** | ✅ AOF/RDB | ❌ |
| **Data structures** | Advanced (Sets, Sorted) | Simple key-value |
| **Architecture** | Single-threaded | Multi-threaded |

**CNAME vs Alias:**

| | CNAME | Alias |
|--|-------|-------|
| Root domain | ❌ | ✅ |
| AWS resources | ❌ | ✅ |
| Free | ❌ | ✅ |
| Health checks | ❌ | ✅ |

### Best Practices 2026

**RDS/Aurora:**
1. Enable Multi-AZ cho production
2. Dùng Aurora thay vì RDS MySQL/Postgres cho performance tốt hơn
3. Enable encryption at launch
4. Dùng RDS Proxy cho Lambda connections
5. Implement proper connection pooling

**ElastiCache:**
1. Dùng Redis cho most cases (HA, persistence)
2. Implement TTL cho cache entries
3. Kết hợp Lazy Loading + Write-through
4. Monitor cache hit ratio
5. Implement cache invalidation strategy

**Route 53:**
1. Dùng Alias cho AWS resources
2. Enable health checks cho production records
3. Dùng appropriate routing policy theo use case
4. Set TTL hợp lý (thấp khi thay đổi, cao khi ổn định)
5. Dùng Private Hosted Zones cho internal services

---

## Liên kết tham khảo

- [Amazon RDS Documentation](https://docs.aws.amazon.com/rds/)
- [Amazon Aurora Documentation](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/)
- [Amazon ElastiCache Documentation](https://docs.aws.amazon.com/elasticache/)
- [Amazon Route 53 Documentation](https://docs.aws.amazon.com/route53/)

---

*Document này được tạo để hỗ trợ học tập cho kỳ thi AWS Certified Developer - Associate*
