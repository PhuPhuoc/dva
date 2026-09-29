# AWS DVA Study Guide - Project Documentation

## Mục đích dự án

Đây là bộ tài liệu học tập chi tiết cho kỳ thi **AWS Certified Developer - Associate (DVA-C02)**, được tạo từ file PDF của Cloud Mentor Pro.

**Thư mục output:** `aws-study/`

---

## Quy ước viết cho các Section

### 1. Cấu trúc file

Mỗi section được lưu trong file riêng:
```
aws-study/
├── section1.md   # Getting Started with AWS, IAM & AWS CLI, EC2 Fundamentals
├── section2.md   # EC2 Instance Storage, High Availability & Scalability
├── section3.md   # RDS, Aurora & ElastiCache, Route53
├── section4.md   # Amazon S3 (pending)
├── section5.md   # Amazon VPC (pending)
└── ...
```

### 2. Quy tắc dịch thuật

#### ✅ GIỮ NGUYÊN tiếng Anh:
- **Keywords quan trọng** xuất hiện trong bài thi: IAM, EC2, EBS, AZ, ALB, NLB, ASG, VPC, RDS, Lambda, S3, SNS, SQS, CDK, SAM, API Gateway, CloudFormation, CodePipeline, etc.
- **Tên services**: AWS IAM, Amazon EC2, Amazon RDS, Amazon Aurora, etc.
- **Khái niệm kỹ thuật**: API, SDK, CLI, JSON, IAM Policy, Security Group, etc.
- **Các thuật ngữ chuyên ngành**: scalability, high availability, fault tolerance, disaster recovery, etc.

#### ✅ DỊCH sang tiếng Việt:
- Mô tả/giải thích khái niệm
- Câu văn dùng để học hiểu
- Use case scenarios
- Best practices

#### ✅ Ghi chú thêm (Notes):

**Format ghi chú:**
- `💡` - Ghi chú cho DVA exam (tips,重点)
- `⚠️` - Cảnh báo/quan trọng
- `📌` - Tổng hợp nhanh

**Nội dung ghi chú bao gồm:**
- Exam tips (những gì hay hỏi trong exam)
- Thông tin bổ sung cho năm 2026 (updates, changes)
- Best practices cho production
- Code examples thực tế
- So sánh giữa các options

### 3. Cấu trúc mỗi Section

```markdown
# Section N: [Tên Section]

**AWS Certified Developer - Associate**  
*Nguồn: Cloud Mentor Pro - Cập nhật: 15/02/2026*

---

## Mục lục

1. [Phần 1](#1-phần-1)
2. [Phần 2](#2-phần-2)
3. [Tổng kết](#3-tổng-kết)

---

## 1. Phần 1

### Topic con

> Quote từ slide PDF (giữ nguyên tiếng Anh)
> *Dịch nghĩa tiếng Việt*

**Nội dung giải thích**

| Table | Cho thông tin có cấu trúc |
|-------|--------------------------|
| ...   | ...                       |

**💡 Ghi chú thêm cho exam/thực tế**

```
Code examples (nếu có)
```

---

## 2. Phần 2

[Tiếp tục cấu trúc tương tự...]

---

## 3. Tổng kết

### Kiến thức trọng tâm cho DVA Exam

| Chủ đề | Trọng tâm |
|--------|-----------|
| ...   | ...       |

### Best Practices 2026

1. ...
2. ...

---

## Liên kết tham khảo

- [Link documentation](url)

---

*Document này được tạo để hỗ trợ học tập cho kỳ thi AWS Certified Developer - Associate*
```

### 4. Table of Contents từ PDF

Dựa trên PDF, đây là các sections đã hoàn thành và sắp tới:

| Section | Nội dung | Status |
|---------|----------|--------|
| 1 | Getting Started with AWS, IAM & AWS CLI, EC2 Fundamentals | ✅ Hoàn thành |
| 2 | EC2 Instance Storage, High Availability & Scalability (ELB & ASG) | ✅ Hoàn thành |
| 3 | RDS, Aurora & ElastiCache, Route53 | ✅ Hoàn thành |
| 4 | Amazon S3, Amazon S3 - Advanced, Amazon S3 - Security | ✅ Hoàn thành |
| 5 | Amazon VPC | ✅ Hoàn thành |
| 6 | ECS, ECR & Fargate - Docker in AWS | ✅ Hoàn thành |
| 7 | CloudFront, AWS Elastic Beanstalk | ✅ Hoàn thành |
| 8 | AWS CloudFormation | ✅ Hoàn thành |
| 9 | AWS Integration & Messaging: SQS, SNS & Kinesis | ✅ Hoàn thành |
| 10 | AWS Monitoring, Troubleshooting & Audit | 📋 Pending |
| 11 | AWS Lambda | 📋 Pending |
| 12 | AWS DynamoDB | 📋 Pending |
| 13 | API Gateway | 📋 Pending |
| 14 | AWS Serverless: SAM, CDK, Cognito, Step Functions & AppSync | 📋 Pending |
| 15 | AWS CICD: CodeCommit, CodePipeline, CodeBuild, CodeDeploy | 📋 Pending |
| 16 | Advanced Identity, AWS Security & Encryption, AWS Other Services | 📋 Pending |

---

## Cách tiếp tục viết Section mới

### Bước 1: Trích xuất nội dung PDF

```bash
# Cài đặt pdftotext (nếu chưa có)
# Thường nằm ở: C:\Program Files\Git\mingw64\bin\pdftotext.exe

# Trích xuất nội dung từng section (điều chỉnh page range)
pdftotext.exe -f [start_page] -l [end_page] "[path_to_pdf]" -
```

### Bước 2: Xác định page range cho section

Từ file PDF, mỗi section bắt đầu với tiêu đề:
```
Section N
□ [Nội dung]
```

### Bước 3: Viết file với format chuẩn

1. Copy cấu trúc template ở trên
2. Dịch từng slide theo quy tắc:
   - Giữ nguyên tiếng Anh cho keywords
   - Thêm ghi chú 💡 cho exam tips
   - Thêm code examples khi phù hợp
   - Tổng hợp cuối section với tables và best practices

### Bước 4: Cập nhật README này

Sau khi hoàn thành section, cập nhật status trong bảng Table of Contents.

---

## Mẫu lệnh trích xuất PDF pages

```powershell
# Section 3 (RDS, Aurora, ElastiCache, Route53)
"C:\Program Files\Git\mingw64\bin\pdftotext.exe" -f 131 -l 200 "[path_to_pdf]" -

# Section 4 (S3)
"C:\Program Files\Git\mingw64\bin\pdftotext.exe" -f 200 -l 280 "[path_to_pdf]" -
```

---

## Tiêu chuẩn chất lượng

### Checklist trước khi commit:

- [ ] Giữ nguyên **tất cả keywords tiếng Anh** (IAM, EC2, VPC, etc.)
- [ ] Dịch đầy đủ **câu văn giải thích** sang tiếng Việt
- [ ] Thêm **ghi chú 💡** với exam tips và thông tin 2026
- [ ] Bao gồm **tables** để so sánh features
- [ ] Có **tổng kết cuối section** với:
  - Kiến thức trọng tâm cho DVA Exam
  - Best Practices 2026
- [ ] Có **code examples** cho các CLI commands hoặc configuration
- [ ] Có **liên kết tham khảo** đến AWS Documentation

---

## Thông tin về kỳ thi DVA

**Mã exam:** DVA-C02
**Phiên bản:** 2026
**Số câu hỏi:** 65 câu
**Thời gian:** 130 phút
**Điểm đạt:** 72%

**Các domain thi:**
1. Deployment (22%)
2. Security (26%)
3. Development with AWS Services (30%)
4. Refactoring (10%)
5. Monitoring and Troubleshooting (12%)

---

*Nội dung được tổng hợp từ tài liệu Cloud Mentor Pro - AWS Certified Developer - Associate v5 (15/02/2026)*
