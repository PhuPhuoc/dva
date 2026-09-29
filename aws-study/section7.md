# Section 7: CloudFront & AWS Elastic Beanstalk

**AWS Certified Developer - Associate**  
*Nguồn: Cloud Mentor Pro - Cập nhật: 15/02/2026*

---

## Mục lục

1. [Amazon CloudFront](#1-amazon-cloudfront)
2. [CloudFront Origins](#2-cloudfront-origins)
3. [CloudFront Caching](#3-cloudfront-caching)
4. [CloudFront Security](#4-cloudfront-security)
5. [AWS Elastic Beanstalk](#5-aws-elastic-beanstalk)
6. [Elastic Beanstalk Deployment](#6-elastic-beanstalk-deployment)
7. [Tổng kết](#7-tổng-kết)

---

## 1. Amazon CloudFront

### Amazon CloudFront Overview

> Content Delivery Network (CDN)  
> *Mạng phân phối nội dung*

> Improves read performance, content is cached at the edge  
> *Cải thiện read performance, content được cache ở edge*

> Improves users experience  
> *Cải thiện trải nghiệm người dùng*

> 600+ Point of Presence globally (edge locations)  
> *600+ Points of Presence toàn cầu*

> DDoS protection (because worldwide), integration with Shield, AWS Web Application Firewall  
> *Bảo vệ DDoS, tích hợp với Shield và WAF*

---

## 2. CloudFront Origins

### CloudFront - Origins

**S3 bucket:**
> For distributing files and caching them at the edge  
> *Phân phối files và cache ở edge*

> Enhanced security with CloudFront Origin Access Control (OAC)  
> *Bảo mật hơn với OAC*

> OAC is replacing Origin Access Identity (OAI)  
> *OAC đang thay thế OAI*

> CloudFront can be used as an ingress (to upload files to S3)  
> *CloudFront có thể dùng làm ingress*

**Custom Origin (HTTP):**

| Origin | Mô tả |
|--------|--------|
| **Application Load Balancer** | ALB phía sau |
| **EC2 instance** | EC2 trực tiếp |
| **S3 website** | S3 static website |
| **Any HTTP backend** | Bất kỳ HTTP backend nào |

---

### CloudFront vs S3 Cross Region Replication

| Feature | CloudFront | S3 Cross Region Replication |
|---------|------------|----------------------------|
| **Scope** | Global Edge network | Per-region |
| **Caching** | Files cached for TTL | Files updated in near real-time |
| **Use case** | Static content, globally available | Dynamic content, low-latency in few regions |
| **Real-time** | ❌ Cached | ✅ Near real-time |

> CloudFront: Great for static content that must be available everywhere

> S3 CRR: Great for dynamic content that needs to be available at low-latency in few regions

---

## 3. CloudFront Caching

### CloudFront Caching

> The cache lives at each CloudFront Edge Location  
> *Cache nằm ở mỗi Edge Location*

> CloudFront identifies each object in the cache using the Cache Key  
> *CloudFront xác định object bằng Cache Key*

> You want to maximize the Cache Hit ratio to minimize requests to the origin  
> *Muốn maximize Cache Hit ratio*

> You can invalidate part of the cache using the CreateInvalidation API  
> *Có thể invalidate cache*

---

### What is CloudFront Cache Key?

> A unique identifier for every object in the cache  
> *Định danh duy nhất cho mỗi object*

> By default, consists of hostname + resource portion of the URL  
> *Mặc định: hostname + resource path*

**Customize Cache Key:**
- Add HTTP headers
- Add cookies
- Add query strings

**Use cases:**
- User-specific content (headers)
- A/B testing (cookies)
- Search filtering (query strings)

---

### CloudFront - Cache Behaviors

> Configure different settings for a given URL path pattern  
> *Cấu hình khác nhau cho URL patterns*

**Ví dụ:**
| Path Pattern | Behavior |
|-------------|----------|
| `/images/*` | Cache 1 day, S3 origin |
| `/api/*` | No cache, ALB origin |
| `/*` | Default cache behavior |

> When adding additional Cache Behaviors, the Default Cache Behavior is always the last to be processed and is always `/*`  
> *Default Cache Behavior luôn xử lý cuối cùng*

---

### CloudFront - Cache Invalidations

> In case you update the back-end origin, CloudFront doesn't know about it and will only get the refreshed content after the TTL has expired  
> *CloudFront không biết khi origin update*

> However, you can force an entire or partial cache refresh (thus bypassing the TTL) by performing a CloudFront Invalidation  
> *Có thể force cache refresh*

**Invalidation:**
```bash
# Invalidate all files
aws cloudfront create-invalidation --distribution-id XYZ --paths "/*"

# Invalidate specific path
aws cloudfront create-invalidation --distribution-id XYZ --paths "/images/*"
```

---

## 4. CloudFront Security

### CloudFront Geo Restriction

> You can restrict who can access your distribution  
> *Có thể restrict ai có thể truy cập*

**Allowlist:**
> Allow your users to access your content only if they're in one of the countries on a list of approved countries  
> *Chỉ cho phép users từ approved countries*

**Blocklist:**
> Prevent your users from accessing your content if they're in one of the countries on a list of banned countries  
> *Block users từ banned countries*

> The "country" is determined using a 3rd party Geo-IP database  
> *Country được xác định bằng Geo-IP database*

> Use case: Copyright Laws to control access to content  
> *Use case: tuân thủ Copyright Laws*

---

### CloudFront Signed URL / Signed Cookies

> You want to distribute paid shared content to premium users  
> *Phân phối paid content cho premium users*

**Policy includes:**
- URL expiration
- IP ranges to access
- Trusted signers (AWS accounts có thể tạo signed URLs)

**URL Expiration:**
| Content Type | TTL |
|-------------|-----|
| Shared content (movie, music) | A few minutes |
| Private content | Years |

**Signed URL vs Signed Cookies:**

| Type | Use Case |
|------|---------|
| **Signed URL** | Access to individual files (one URL per file) |
| **Signed Cookies** | Access to multiple files (one cookie for many files) |

---

### CloudFront - Field Level Encryption

> Protect user sensitive information through application stack  
> *Bảo vệ sensitive information*

> Adds an additional layer of security along with HTTPS  
> *Thêm layer bảo mật*

> Sensitive information encrypted at the edge close to user  
> *Encrypt ở edge gần user*

> Uses asymmetric encryption  
> *Dùng asymmetric encryption*

**Usage:**
- Specify up to 10 fields trong POST requests cần encrypt
- Specify public key để encrypt

---

### CloudFront - Real Time Logs

> Get real-time requests received by CloudFront sent to Kinesis Data Streams  
> *Gửi real-time logs đến Kinesis Data Streams*

> Monitor, analyze, and take actions based on content delivery performance  
> *Monitor và phân tích performance*

**Options:**
- Sampling Rate (%)
- Specific fields
- Specific Cache Behaviors

---

### CloudFront - Price Classes

> You can reduce the number of edge locations for cost reduction  
> *Giảm edge locations để giảm chi phí*

| Price Class | Coverage |
|-------------|---------|
| **Price Class All** | All regions - best performance |
| **Price Class 200** | Most regions, excludes most expensive |
| **Price Class 100** | Only least expensive regions |

---

## 5. AWS Elastic Beanstalk

### AWS Elastic Beanstalk Overview

> Deploying applications in AWS safely and predictably  
> *Deploy ứng dụng trong AWS an toàn và có thể dự đoán*

---

### Developer Problems on AWS

> Managing infrastructure  
> *Quản lý infrastructure*

> Deploying Code  
> *Deploy code*

> Configuring all the databases, load balancers, etc  
> *Cấu hình databases, load balancers*

> Scaling concerns  
> *Lo lắng về scaling*

> Most web apps have the same architecture (ALB + ASG)  
> *Hầu hết web apps có cùng architecture*

> All the developers want is for their code to run!  
> *Developers chỉ muốn code chạy!*

---

### Elastic Beanstalk - Overview

> Elastic Beanstalk is a developer centric view of deploying an application on AWS  
> *Góc nhìn developer-centric để deploy*

> It uses all the component's we've seen before: EC2, ASG, ELB, RDS, ...  
> *Dùng tất cả components: EC2, ASG, ELB, RDS*

**Managed service:**
> Automatically handles capacity provisioning, load balancing, scaling, application health monitoring, instance configuration, ...  
> *Tự động handle capacity, load balancing, scaling, monitoring*

> Just the application code is the responsibility of the developer  
> *Chỉ cần lo về application code*

> We still have full control over the configuration  
> *Vẫn có full control*

> Beanstalk is free but you pay for the underlying instances  
> *Beanstalk miễn phí, trả tiền cho underlying instances*

---

### Elastic Beanstalk - Components

| Component | Mô tả |
|-----------|--------|
| **Application** | Collection of Beanstalk components |
| **Application Version** | Iteration của application code |
| **Environment** | Collection of AWS resources (chạy một version) |
| **Tiers** | Web Server Tier, Worker Tier |

> You can create multiple environments (dev, test, prod, ...)  
> *Có thể tạo nhiều environments*

---

### Elastic Beanstalk - Supported Platforms

| Platform | Description |
|----------|-------------|
| **Go** | Go application |
| **Java SE** | Java |
| **Java with Tomcat** | Java Tomcat |
| **.NET Core on Linux** | .NET Core |
| **.NET on Windows Server** | .NET Windows |
| **Node.js** | Node.js |
| **PHP** | PHP |
| **Python** | Python |
| **Ruby** | Ruby |
| **Packer Builder** | Custom |
| **Single Container Docker** | Docker single container |
| **Multi-container Docker** | Docker multiple containers |
| **Preconfigured Docker** | Docker preconfigured |

---

## 6. Elastic Beanstalk Deployment

### Beanstalk Deployment Options

| Option | Downtime | Additional Cost | Speed |
|--------|----------|----------------|-------|
| **All at once** | ✅ Có | ❌ | Fastest |
| **Rolling** | Một phần | ❌ | Medium |
| **Rolling with additional batches** | ❌ | ✅ Có | Medium |
| **Immutable** | ❌ | ✅ Có | Slow |
| **Blue Green** | ❌ | ✅ Có | Slow |
| **Traffic Splitting** | ❌ | ✅ Có | Medium |

---

### All at Once

> Fastest deployment  
> *Deployment nhanh nhất*

> Application has downtime  
> *Có downtime*

> Great for quick iterations in development environment  
> *Phù hợp cho development*

> No additional cost  
> *Không có additional cost*

---

### Rolling

> Application is running below capacity  
> *Application chạy dưới capacity*

> Can set the bucket size  
> *Có thể set bucket size*

> Application is running both versions simultaneously  
> *Cả hai versions cùng chạy*

> No additional cost  
> *Không có additional cost*

> Long deployment  
> *Deployment lâu hơn*

---

### Rolling with Additional Batches

> Application is running at capacity  
> *Application chạy ở full capacity*

> Can set the bucket size  
> *Có thể set bucket size*

> Application is running both versions simultaneously  
> *Cả hai versions cùng chạy*

> Small additional cost  
> *Có một chút additional cost*

> Additional batch is removed at the end of the deployment  
> *Batch được remove sau khi deploy xong*

> Longer deployment  
> *Deployment lâu hơn*

> Good for production  
> *Phù hợp cho production*

---

### Immutable

> Spins up new instances in a new ASG, deploys version to these instances, and then swaps all the instances when everything is healthy  
> *Tạo instances mới trong ASG mới, deploy, swap khi healthy*

**Ưu điểm:**
- Zero downtime
- Rollback đơn giản (terminate new instances)
- Temporary ASG được tạo

---

### Blue Green

> Create a new environment and switch over when ready  
> *Tạo environment mới, switch khi ready*

**Steps:**
1. Tạo new environment
2. Deploy new version
3. Swap URLs
4. Terminate old environment

---

### Traffic Splitting

> Canary testing - send a small % of traffic to new deployment  
> *Testing với % traffic nhỏ*

**Use case:**
- A/B testing
- Gradual rollout
- Instant rollback

---

## 7. Tổng kết

### Kiến thức trọng tâm cho DVA Exam

| Chủ đề | Trọng tâm |
|--------|-----------|
| **CloudFront** | CDN, Origins, Caching, Cache Key |
| **CloudFront Security** | Geo Restriction, Signed URLs, Field Level Encryption |
| **Beanstalk** | Components, Platforms, Deployment options |
| **Deployment** | All at once, Rolling, Immutable, Blue Green |

### So sánh CloudFront vs S3 CRR

| | CloudFront | S3 CRR |
|--|-----------|---------|
| Scope | Global | Per-region |
| Cache | TTL-based | Real-time |
| Cost | Edge locations | Data transfer |
| Use case | Static, global | Dynamic, few regions |

### Deployment Options Comparison

| Option | Downtime | Cost | Rollback |
|--------|----------|------|---------|
| All at once | ✅ | Lowest | Manual |
| Rolling | Partial | Low | Reverse |
| Rolling + batches | ❌ | Medium | Terminate |
| Immutable | ❌ | Higher | Terminate |
| Blue Green | ❌ | Highest | Swap URL |

### Best Practices 2026

**CloudFront:**
1. ✅ Dùng OAC thay vì OAI (mới hơn)
2. ✅ Minimize Cache Key để maximize hits
3. ✅ Dùng Cache Behaviors để phân tách static/dynamic
4. ✅ Dùng Signed URLs cho premium content
5. ✅ Chọn Price Class phù hợp để tối ưu chi phí

**Elastic Beanstalk:**
1. ✅ Dùng Immutable cho production (zero downtime, easy rollback)
2. ✅ Rolling with additional batches cho balance
3. ✅ Dùng `.ebextensions` để customize environment
4. ✅ Health monitoring và alerts
5. ✅ Separate environments cho dev/staging/prod

---

## Liên kết tham khảo

- [Amazon CloudFront Documentation](https://docs.aws.amazon.com/cloudfront/)
- [CloudFront Cache Key](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/cache-key-select.html)
- [AWS Elastic Beanstalk Documentation](https://docs.aws.amazon.com/elastic-beanstalk/)
- [Beanstalk Deployment](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.deploy-existing-version.html)

---

*Document này được tạo để hỗ trợ học tập cho kỳ thi AWS Certified Developer - Associate*
