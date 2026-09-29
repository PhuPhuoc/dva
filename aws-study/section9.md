# Section 9: AWS Integration & Messaging (SQS, SNS & Kinesis)

**AWS Certified Developer - Associate**  
*Nguồn: Cloud Mentor Pro - Cập nhật: 15/02/2026*

---

## Mục lục

1. [Introduction to Messaging](#1-introduction-to-messaging)
2. [Amazon SQS - Standard Queue](#2-amazon-sqs---standard-queue)
3. [SQS Producers & Consumers](#3-sqs-producers--consumers)
4. [SQS Security](#4-sqs-security)
5. [SQS Message Visibility](#5-sqs-message-visibility)
6. [Dead Letter Queue (DLQ)](#6-dead-letter-queue-dlq)
7. [Amazon SNS](#7-amazon-sns)
8. [SNS vs SQS](#8-sns-vs-sqs)
9. [Amazon Kinesis](#9-amazon-kinesis)
10. [Tổng kết](#10-tổng-kết)

---

## 1. Introduction to Messaging

### Section Introduction

> When we start deploying multiple applications, they will inevitably need to communicate with one another  
> *Khi deploy nhiều applications, chúng cần giao tiếp với nhau*

**Hai patterns giao tiếp:**

| Pattern | Mô tả |
|---------|--------|
| **Synchronous** | Ứng dụng giao tiếp trực tiếp |
| **Asynchronous/Decoupled** | Ứng dụng giao tiếp qua messaging service |

> Synchronous between applications can be problematic if there are sudden spikes of traffic  
> *Synchronous có thể gây vấn đề khi có traffic spikes*

> What if you need to suddenly encode 1000 videos but usually it's 10?  
> *Nếu đột nhiên cần encode 1000 videos nhưng thường chỉ 10?*

**Decouple applications:**
> In that case, it's better to decouple your applications  
> *Tốt hơn nên decouple applications*

| Service | Pattern | Use Case |
|---------|---------|----------|
| **SQS** | Queue model | Batch processing, task queues |
| **SNS** | Pub/sub model | Notifications, fan-out |
| **Kinesis** | Real-time streaming | Analytics, logs, IoT |

> These services can scale independently from our application!  
> *Các services này có thể scale độc lập*

---

## 2. Amazon SQS - Standard Queue

### Amazon SQS Overview

> Oldest offering (over 10 years old)  
> *Dịch vụ oldest (hơn 10 năm)*

> Fully managed service, used to decouple applications  
> *Service được quản lý hoàn toàn*

**SQS Attributes:**

| Attribute | Value |
|-----------|-------|
| **Throughput** | Unlimited |
| **Messages in queue** | Unlimited |
| **Default retention** | 4 days |
| **Maximum retention** | 14 days |
| **Latency** | <10 ms |
| **Message size** | Max 256KB |

**Characteristics:**
> Can have duplicate messages (at least once delivery, occasionally)  
> *Có thể có duplicate messages*

> Can have out of order messages (best effort ordering)  
> *Có thể có out of order messages*

---

## 3. SQS Producers & Consumers

### SQS - Producing Messages

> Produced to SQS using the SDK (SendMessage API)  
> *Produce message qua SDK*

> The message is persisted in SQS until a consumer deletes it  
> *Message được lưu trong SQS cho đến khi consumer xóa*

> Message retention: default 4 days, up to 14 days  
> *Message retention: 4-14 days*

**Message content:**
```python
# Ví dụ: gửi order để xử lý
message = {
    "order_id": "12345",
    "customer_id": "67890",
    "attributes": {...}
}

sqs.send_message(
    QueueUrl="https://sqs.us-east-1.amazonaws.com/...",
    MessageBody=json.dumps(message)
)
```

### SQS - Consuming Messages

> Consumers (running on EC2 instances, servers, or AWS Lambda)...  
> *Consumers có thể chạy trên EC2, servers, hoặc Lambda*

> Poll SQS for messages (receive up to 10 messages at a time)  
> *Poll messages (receive up to 10 messages)*

> Process the messages (example: insert the message into an RDS database)  
> *Xử lý messages*

> Delete the messages using the DeleteMessage API  
> *Xóa messages sau khi xử lý*

### SQS with Auto Scaling Group (ASG)

> Consumers receive and process messages in parallel  
> *Consumers nhận và xử lý messages song song*

> At least once delivery  
> *Ít nhất một lần*

> Best-effort message ordering  
> *Best-effort ordering*

> Consumers delete messages after processing them  
> *Consumers xóa message sau khi xử lý*

> We can scale consumers horizontally to improve throughput of processing  
> *Scale consumers horizontally để improve throughput*

---

## 4. SQS Security

### SQS - Security

**Encryption:**

| Type | Mô tả |
|------|--------|
| **In-flight** | HTTPS API |
| **At-rest** | KMS keys |
| **Client-side** | Client tự encrypt/decrypt |

**Access Controls:**

| Type | Mô tả |
|------|--------|
| **IAM policies** | Regulate access to SQS API |
| **SQS Access Policies** | Cross-account access, allow SNS/S3 to write |

### SQS Queue Access Policy

**Cross-account Access:**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": { "AWS": ["111122223333"] },
    "Action": ["sqs:ReceiveMessage"],
    "Resource": "arn:aws:sqs:us-east-1:444455556666:queue1"
  }]
}
```

**Allow SNS to write to SQS:**
```json
{
  "Effect": "Allow",
  "Principal": { "AWS": "*" },
  "Action": ["sqs:SendMessage"],
  "Resource": "arn:aws:sqs:us-east-1:444455556666:queue1",
  "Condition": {
    "ArnLike": { "aws:SourceArn": "arn:aws:s3:*:*:bucket1" }
  }
}
```

---

## 5. SQS Message Visibility

### SQS - Message Visibility Timeout

> After a message is polled by a consumer, it becomes invisible to other consumers  
> *Sau khi consumer poll, message invisible với consumers khác*

> By default, the "message visibility timeout" is 30 seconds  
> *Mặc định: 30 seconds*

> That means the message has 30 seconds to be processed  
> *Message có 30 giây để xử lý*

> After the message visibility timeout is over, the message is "visible" in SQS  
> *Sau timeout, message visible lại*

**Visibility Timeout Issues:**

> If a message is not processed within the visibility timeout, it will be processed twice  
> *Nếu không xử lý trong timeout, message sẽ được xử lý 2 lần*

> A consumer could call the ChangeMessageVisibility API to get more time  
> *Consumer có thể gọi ChangeMessageVisibility để xin thêm thời gian*

> If visibility timeout is high (hours), and consumer crashes, re-processing will take time  
> *Timeout cao → re-processing lâu nếu crash*

> If visibility timeout is too low (seconds), we may get duplicates  
> *Timeout thấp → duplicates*

---

## 6. Dead Letter Queue (DLQ)

### Amazon SQS - Dead Letter Queue (DLQ)

> If a consumer fails to process a message within the Visibility Timeout... the message goes back to the queue!  
> *Nếu consumer fail → message quay lại queue*

> We can set a threshold of how many times a message can go back to the queue  
> *Có thể set threshold cho số lần message quay lại*

> After the MaximumReceives threshold is exceeded, the message goes into a Dead Letter Queue (DLQ)  
> *Sau khi exceed MaximumReceives → message vào DLQ*

> Useful for debugging!  
> *Hữu ích cho debugging*

**DLQ Rules:**
> DLQ of a FIFO queue must also be a FIFO queue  
> *FIFO queue → FIFO DLQ*

> DLQ of a Standard queue must also be a Standard queue  
> *Standard queue → Standard DLQ*

> Make sure to process the messages in the DLQ before they expire  
> *Xử lý DLQ messages trước khi expire*

> Good to set a retention of 14 days in the DLQ  
> *Set retention 14 days*

### SQS DLQ - Redrive to Source

> Feature to help consume messages in the DLQ to understand what is wrong with them  
> *Giúp xử lý DLQ messages để understand issues*

> When our code is fixed, we can redrive the messages from the DLQ back into the source queue (or any other queue) in batches without writing custom code  
> *Có thể redrive messages từ DLQ về source queue*

---

## 7. Amazon SNS

### Amazon SNS Overview

> SNS = Simple Notification Service  
> *Dịch vụ notification*

> Publisher publishes message to SNS topic  
> *Publisher gửi message đến SNS topic*

> Subscribers receive message via supported protocols  
> *Subscribers nhận message qua các protocols*

**SNS Features:**
- Fan-out pattern
- Multiple subscribers
- Retry mechanism
- Message filtering

**SNS vs SQS:**

| Feature | SNS | SQS |
|---------|-----|-----|
| **Pattern** | Pub/Sub | Queue |
| **Delivery** | Push | Poll |
| **Subscribers** | Multiple | Single consumer |
| **Message handling** | Ephemeral | Persistent |

---

## 8. SNS vs SQS

### When to use?

| Service | Use Case |
|---------|----------|
| **SQS** | Decouple consumers, queue processing, batch jobs |
| **SNS** | Fan-out notifications, multiple subscribers, pub/sub |

**Common pattern: SNS + SQS**

> Use SNS to publish to multiple SQS queues  
> *Dùng SNS để publish đến nhiều SQS queues*

> This gives you fan-out pattern with SQS durability  
> *Fan-out pattern với SQS durability*

```
Publisher → SNS Topic
    │
    ├── SQS Queue 1 (Processing)
    ├── SQS Queue 2 (Analytics)
    └── SQS Queue 3 (Logging)
```

---

## 9. Amazon Kinesis

### Amazon Kinesis Overview

> Real-time streaming data  
> *Dữ liệu streaming real-time*

**Kinesis Services:**

| Service | Use Case |
|---------|----------|
| **Kinesis Data Streams** | Collect and process streaming data |
| **Kinesis Data Firehose** | Load data to destinations |
| **Kinesis Data Analytics** | Analyze data with SQL |
| **Kinesis Video Streams** | Stream video |

**Kinesis Data Streams:**
- Shards-based
- Custom consumers
- Real-time processing

**Kinesis Data Firehose:**
- Fully managed
- Auto-scaling
- Near real-time
- Load to S3, Redshift, Elasticsearch, etc.

---

## 10. Tổng kết

### Kiến thức trọng tâm cho DVA Exam

| Chủ đề | Trọng tâm |
|--------|-----------|
| **SQS Standard Queue** | Attributes, throughput, retention |
| **SQS Producers/Consumers** | SendMessage, ReceiveMessage, DeleteMessage |
| **SQS Security** | IAM policies, SQS Access Policies |
| **Message Visibility** | Timeout, ChangeMessageVisibility |
| **Dead Letter Queue** | MaximumReceives, DLQ, Redrive |
| **SNS** | Pub/Sub, Fan-out, Subscribers |
| **Kinesis** | Data Streams, Firehose, Analytics |

### SQS Configuration Options

| Option | Default | Description |
|--------|---------|-------------|
| **Visibility Timeout** | 30s | Time message invisible |
| **Message Retention** | 4 days | Max 14 days |
| **Receive Count** | - | Times message received |
| **MaximumReceives** | - | Threshold for DLQ |

### Best Practices 2026

**SQS:**
1. ✅ Dùng Dead Letter Queue để handle failed messages
2. ✅ Set visibility timeout phù hợp (đủ để process, không quá lâu)
3. ✅ Dùng SQS Access Policies cho cross-account access
4. ✅ Implement idempotent message processing
5. ✅ Use Long Polling để giảm empty responses

**SNS:**
1. ✅ Dùng SNS + SQS cho fan-out với durability
2. ✅ Implement message filtering để subscribers chỉ nhận relevant messages
3. ✅ Dùng SNS for mobile push, email, SMS

**Kinesis:**
1. ✅ Dùng Firehose cho load data đến S3/Redshift
2. ✅ Dùng Data Analytics cho real-time SQL analysis
3. ✅ Design shard capacity properly

---

## Liên kết tham khảo

- [Amazon SQS Documentation](https://docs.aws.amazon.com/sqs/)
- [Amazon SNS Documentation](https://docs.aws.amazon.com/sns/)
- [Amazon Kinesis Documentation](https://docs.aws.amazon.com/kinesis/)

---

*Document này được tạo để hỗ trợ học tập cho kỳ thi AWS Certified Developer - Associate*
