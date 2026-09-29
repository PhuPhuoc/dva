# Section 8: AWS CloudFormation

**AWS Certified Developer - Associate**  
*Nguồn: Cloud Mentor Pro - Cập nhật: 15/02/2026*

---

## Mục lục

1. [CloudFormation Overview](#1-cloudformation-overview)
2. [CloudFormation Templates](#2-cloudformation-templates)
3. [CloudFormation Resources](#3-cloudformation-resources)
4. [CloudFormation Parameters](#4-cloudformation-parameters)
5. [CloudFormation Mappings](#5-cloudformation-mappings)
6. [CloudFormation Outputs](#6-cloudformation-outputs)
7. [CloudFormation Conditions](#7-cloudformation-conditions)
8. [Intrinsic Functions](#8-intrinsic-functions)
9. [CloudFormation Rollbacks](#9-cloudformation-rollbacks)
10. [CloudFormation Security](#10-cloudformation-security)
11. [DeletionPolicy](#11-deletionpolicy)
12. [Stack Policies](#12-stack-policies)
13. [Custom Resources](#13-custom-resources)
14. [StackSets](#14-stacksets)
15. [Tổng kết](#15-tổng-kết)

---

## 1. CloudFormation Overview

### AWS CloudFormation

> CloudFormation is a declarative way of outlining your AWS Infrastructure, for any resources (most of them are supported)  
> *CloudFormation là cách khai báo để định nghĩa AWS Infrastructure*

**Ví dụ khai báo:**
```yaml
# Tôi muốn:
# - Một security group
# - Hai EC2 instances sử dụng security group này
# - Hai Elastic IPs cho các EC2 instances
# - Một S3 bucket
# - Một Load Balancer (ELB) đặt trước các EC2 instances

# CloudFormation sẽ tạo tất cả cho bạn, theo đúng thứ tự, với cấu hình chính xác
```

---

### Benefits of AWS CloudFormation

**Infrastructure as Code:**
> No resources are manually created, which is excellent for control  
> *Không có tài nguyên được tạo thủ công*

> The code can be version controlled for example using Git  
> *Code có thể được version control*

> Changes to the infrastructure are reviewed through code  
> *Thay đổi được review qua code*

**Cost:**
> Each resources within the stack is tagged with an identifier so you can easily see how much a stack costs you  
> *Mỗi resource được tag để theo dõi chi phí*

> You can estimate the costs of your resources using the CloudFormation template  
> *Có thể ước tính chi phí*

> Savings strategy: In Dev, you could automation deletion of templates at 5 PM and recreated at 8 AM, safely  
> *Automation xóa stack vào 5PM, tạo lại vào 8AM*

**Productivity:**
> Ability to destroy and re-create an infrastructure on the cloud on the fly  
> *Có thể xóa và tạo lại infrastructure nhanh chóng*

> Automated generation of Diagram for your templates!  
> *Tự động tạo diagram cho templates*

> Declarative programming (no need to figure out ordering and orchestration)  
> *Không cần lo về thứ tự và orchestration*

**Separation of Concern:**
> Create many stacks for many apps, and many layers  
> *Tạo nhiều stacks cho nhiều apps và layers*

> Example: VPC stacks, Network stacks, App stacks

**Don't re-invent the wheel:**
> Leverage existing templates on the web!  
> *Tận dụng templates có sẵn*

---

### How CloudFormation Works

> Templates must be uploaded in S3 and then referenced in CloudFormation  
> *Templates phải upload lên S3*

> To update a template, we can't edit previous ones. We have to re-upload a new version of the template to AWS  
> *Để update, phải upload version mới*

> Stacks are identified by a name  
> *Stacks được định danh bằng name*

> Deleting a stack deletes every single artifact that was created by CloudFormation  
> *Xóa stack = xóa tất cả artifacts*

---

## 2. CloudFormation Templates

### CloudFormation - Building Blocks

**Template's Components:**

| Component | Mô tả |
|-----------|--------|
| **AWSTemplateFormatVersion** | "2010-09-09" - identifies template capabilities |
| **Description** | Comments about the template |
| **Resources** (MANDATORY) | AWS resources được khai báo |
| **Parameters** | Dynamic inputs cho template |
| **Mappings** | Static variables |
| **Outputs** | References to what has been created |
| **Conditionals** | Conditions để control resource creation |

**Template's Helpers:**
- References
- Functions

---

### YAML Crash Course

**YAML vs JSON:**
> JSON is horrible for CF  
> *JSON không tốt cho CloudFormation*

> YAML is great in so many ways  
> *YAML tốt hơn*

**YAML Features:**
- Key value Pairs
- Nested objects
- Support Arrays
- Multi line strings
- Can include comments

---

## 3. CloudFormation Resources

### CloudFormation - Resources

> Resources are the core of your CloudFormation template (MANDATORY)  
> *Resources là phần cốt lõi*

> They represent the different AWS Components that will be created and configured  
> *Represent các AWS Components*

> Resources are declared and can reference each other  
> *Resources có thể reference nhau*

> AWS figures out creation, updates and deletes of resources for us  
> *AWS tự xử lý creation, updates, deletes*

> There are over 700 types of resources  
> *Có hơn 700 loại resources*

**Resource Type Format:**
```
service-provider::service-name::data-type-name
```

**Ví dụ:**
```yaml
Resources:
  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-12345678
      InstanceType: t2.micro
```

**Tìm tài liệu Resource:**
```
https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html
```

---

## 4. CloudFormation Parameters

### CloudFormation - Parameters

> Parameters are a way to provide inputs to your AWS CloudFormation template  
> *Parameters cung cấp inputs cho template*

**Khi nào dùng Parameter?**
> Is this CloudFormation resource configuration likely to change in the future? If so, make it a parameter  
> *Nếu config có thể thay đổi trong tương lai → make it a parameter*

**Parameter Settings:**

| Setting | Type | Mô tả |
|---------|------|--------|
| **Type** | String | Text |
| | Number | Số |
| | CommaDelimitedList | Danh sách |
| | List\<Number\> | Danh sách số |
| | AWS-Specific | AWS services (tham chiếu valid values) |
| **ConstraintDescription** | String | Mô tả lỗi |
| **Min/MaxLength** | Number | Độ dài tối thiểu/tối đa |
| **Min/MaxValue** | Number | Giá trị tối thiểu/tối đa |
| **Default** | Any | Giá trị mặc định |
| **AllowedValues** | Array | Cho phép các giá trị cụ thể |
| **AllowedPattern** | Regex | Pattern validation |
| **NoEcho** | Boolean | Ẩn giá trị khi nhập |

**Ví dụ Parameter:**
```yaml
Parameters:
  InstanceType:
    Type: String
    Default: t2.micro
    AllowedValues:
      - t2.micro
      - t2.small
      - t2.medium
    Description: EC2 instance type
```

---

## 5. CloudFormation Mappings

### CloudFormation - Mappings

> Mappings are fixed variables within your CloudFormation template  
> *Mappings là các biến cố định*

> They're very handy to differentiate between different environments (dev vs prod), regions (AWS regions), AMI types...  
> *Hữu ích để phân biệt environments, regions, AMI types*

> All the values are hardcoded within the template  
> *Tất cả values được hardcode*

**Ví dụ Mappings:**
```yaml
Mappings:
  RegionMap:
    us-east-1:
      HVM64: ami-0ff8a01507d60d589
    us-west-2:
      HVM64: ami-0b4940186c31d2d14
    ap-southeast-1:
      HVM64: ami-0cf368f6121e1c01d
```

### Accessing Mapping Values (Fn::FindInMap)

> We use Fn::FindInMap to return a named value from a specific key  
> *Dùng Fn::FindInMap để lấy giá trị*

```yaml
!FindInMap [ MapName, TopLevelKey, SecondLevelKey ]
```

**Ví dụ:**
```yaml
Resources:
  MyEC2:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !FindInMap [ RegionMap, !Ref "AWS::Region", HVM64 ]
```

**Khi nào dùng Mappings vs Parameters?**

| Mappings | Parameters |
|----------|-----------|
| Giá trị cố định, có thể suy ra từ biến | Giá trị do user nhập |
| Region, AZ, Environment | User-specific values |
| An toàn hơn | |

---

## 6. CloudFormation Outputs

### CloudFormation - Outputs

> The Outputs section declares optional outputs values that we can import into other stacks (if you export them first)!  
> *Outputs khai báo values có thể import vào stacks khác*

> You can also view the outputs in the AWS Console or in using the AWS CLI  
> *Có thể xem outputs trong Console hoặc CLI*

**Use case:**
> If you define a network CloudFormation, and output the variables such as VPC ID and your Subnet IDs  
> *Network stack output VPC ID, Subnet IDs*

> It's the best way to perform some collaboration cross stack  
> *Collaboration cross stack*

---

### CloudFormation - Cross-Stack Reference

> We then create a second template that leverages that security group  
> *Template thứ hai sử dụng security group*

> For this, we use the Fn::ImportValue function  
> *Dùng Fn::ImportValue*

> You can't delete the underlying stack until all the references are deleted  
> *Không thể xóa stack nếu còn references*

**Cross-Stack Reference Flow:**
```
Stack A (Network)
    │
    └── Outputs:
          VPC: vpc-123
          Subnet: subnet-456

Stack B (Application)
    │
    └── !ImportValue from Stack A
          VPC: vpc-123
          Subnet: subnet-456
```

---

## 7. CloudFormation Conditions

### CloudFormation - Conditions

> Conditions are used to control the creation of resources or outputs based on a condition  
> *Conditions kiểm soát việc tạo resources/outputs*

**Common conditions:**
- Environment (dev / test / prod)
- AWS Region
- Parameter value

### How to Define a Condition

**Logical Functions:**
```yaml
Fn::And
Fn::Equals
Fn::If
Fn::Not
Fn::Or
```

**Ví dụ:**
```yaml
Conditions:
  IsProduction: !Equals [!Ref Environment, prod]

Resources:
  ProductionDB:
    Type: AWS::RDS::DBInstance
    Condition: IsProduction
    Properties:
      # ...
```

---

## 8. Intrinsic Functions

### CloudFormation - Intrinsic Functions

| Function | Mô tả |
|----------|--------|
| **Ref** | Reference parameters hoặc resources |
| **Fn::GetAtt** | Get attributes của resources |
| **Fn::FindInMap** | Get value từ mapping |
| **Fn::ImportValue** | Import values từ stack khác |
| **Fn::Base64** | Convert string to Base64 |
| **Fn::Sub** | Substitute variables trong string |
| **Fn::Join** | Join strings với delimiter |
| **Fn::Cidr** | Generate CIDR blocks |
| **Fn::Select** | Select item từ list |
| **Fn::Split** | Split string thành list |

---

### Intrinsic Functions - Fn::Ref

> The Fn::Ref function can be leveraged to reference  
> *Ref dùng để reference*

> Parameters - returns the value of the parameter  
> *Returns giá trị của parameter*

> Resources - returns the physical ID of the underlying resource (e.g., EC2 ID)  
> *Returns physical ID của resource*

```yaml
# Shorthand
!Ref MyParameter
!Ref MyEC2Instance
```

---

### Intrinsic Functions - Fn::GetAtt

> Attributes are attached to any resources you create  
> *Attributes được gắn với resources*

**Ví dụ:**
```yaml
!GetAtt MyEC2Instance.AvailabilityZone
!GetAtt MyEC2Instance.PrivateIp
```

---

### Intrinsic Functions - Fn::Sub

> Substitute variables in a string  
> *Thay thế biến trong string*

**Ví dụ:**
```yaml
!Sub |
  Name: ${Name}
  Region: ${AWS::Region}
```

---

## 9. CloudFormation Rollbacks

### CloudFormation - Rollbacks

**Stack Creation Fails:**
> Default: everything rolls back (gets deleted). We can look at the log  
> *Mặc định: rollback tất cả*

> Option to disable rollback and troubleshoot what happened  
> *Có thể disable rollback để troubleshoot*

**Stack Update Fails:**
> The stack automatically rolls back to the previous known working state  
> *Tự động rollback về state trước*

> Ability to see in the log what happened and error messages  
> *Xem log và error messages*

**Rollback Failure:**
> Fix resources manually then issue ContinueUpdateRollback API  
> *Fix resources thủ công rồi gọi ContinueUpdateRollback*

---

## 10. CloudFormation Security

### CloudFormation - Service Role

> IAM role that allows CloudFormation to create/update/delete stack resources on your behalf  
> *IAM role cho phép CloudFormation thực hiện actions*

> Give ability to users to create/update/delete the stack resources even if they don't have permissions to work with the resources in the stack  
> *Users có thể create stack dù không có direct permissions*

**Use cases:**
> You want to achieve the least privilege principle  
> *Áp dụng nguyên tắc least privilege*

> But you don't want to give the user all the required permissions to create the stack resources  
> *Không muốn give user all permissions*

> User must have iam:PassRole permissions  
> *User phải có iam:PassRole*

---

### CloudFormation - Termination Protection

> To prevent accidental deletes of CloudFormation Stacks, use TerminationProtection  
> *Ngăn accidental deletes*

---

## 11. DeletionPolicy

### CloudFormation - DeletionPolicy

> Control what happens when the CloudFormation template is deleted or when a resource is removed from a CloudFormation template  
> *Kiểm soát điều gì xảy ra khi delete*

**Options:**

| Policy | Mô tả |
|--------|--------|
| **Delete** (default) | Xóa resource |
| **Retain** | Giữ lại resource |
| **Snapshot** | Tạo snapshot trước khi xóa |

**⚠️ Lưu ý:** Delete không hoạt động trên S3 bucket nếu bucket không trống.

### DeletionPolicy = Retain

> Specify on resources to preserve in case of CloudFormation deletes  
> *Giữ lại resource khi delete*

> Works with any resources  
> *Hoạt động với mọi resources*

### DeletionPolicy = Snapshot

> Create one final snapshot before deleting the resource  
> *Tạo snapshot trước khi xóa*

**Supported resources:**
- EBS Volume
- ElastiCache Cluster
- RDS DBInstance
- RDS DBCluster
- Redshift Cluster
- Neptune DBCluster
- DocumentDB DBCluster

---

## 12. Stack Policies

### CloudFormation - Stack Policies

> During a CloudFormation Stack update, all update actions are allowed on all resources (default)  
> *Mặc định: tất cả resources có thể update*

> A Stack Policy is a JSON document that defines the update actions that are allowed on specific resources during Stack updates  
> *Stack Policy định nghĩa actions được phép trên resources*

> Protect resources from unintentional updates  
> *Bảo vệ resources khỏi unintentional updates*

> When you set a Stack Policy, all resources in the Stack are protected by default  
> *Khi set Stack Policy, tất cả resources được bảo vệ*

> Specify an explicit ALLOW for the resources you want to be allowed to be updated  
> *Phải specify ALLOW cho resources muốn update*

---

## 13. Custom Resources

### CloudFormation - Custom Resources

**Used to:**
> Define resources not yet supported by CloudFormation  
> *Define resources chưa được support*

> Define custom provisioning logic for resources can that be outside of CloudFormation (on-premises resources, 3rd party resources...)  
> *Custom provisioning logic*

> Have custom scripts run during create / update / delete through Lambda functions  
> *Custom scripts qua Lambda*

**Define Custom Resource:**
```yaml
Type: AWS::CloudFormation::CustomResource
Properties:
  ServiceToken: !GetAtt MyLambdaFunction.Arn
  # Custom properties
```

**Backed by:**
- Lambda function (most common)
- SNS topic

**Use Case - Delete content from S3 bucket:**
> You can't delete a non-empty S3 bucket  
> *Không thể xóa S3 bucket không trống*

> We can use a custom resource to empty an S3 bucket before it gets deleted by CloudFormation  
> *Custom resource để empty S3 bucket trước khi xóa*

---

## 14. StackSets

### CloudFormation - StackSets

> Create, update, or delete stacks across multiple accounts and regions with a single operation/template  
> *Tạo/update/delete stacks across nhiều accounts và regions*

> Target accounts to create, update, delete stack instances from StackSets  
> *Chỉ định target accounts*

> When you update a stack set, all associated stack instances are updated throughout all accounts and regions  
> *Update một lần, affect tất cả*

> Can be applied into all accounts of an AWS Organization  
> *Có thể apply cho tất cả accounts trong Organization*

> Only Administrator account (or Delegated Administrator) can create StackSets  
> *Chỉ Admin account mới tạo được*

---

## 15. Tổng kết

### Kiến thức trọng tâm cho DVA Exam

| Chủ đề | Trọng tâm |
|--------|-----------|
| **CloudFormation Overview** | Declarative, Infrastructure as Code |
| **Template Structure** | Resources (MANDATORY), Parameters, Mappings, Outputs, Conditions |
| **Intrinsic Functions** | Ref, GetAtt, FindInMap, ImportValue, Sub, Join |
| **Rollbacks** | Creation fail, Update fail, ContinueUpdateRollback |
| **DeletionPolicy** | Delete, Retain, Snapshot |
| **Stack Policies** | Protect resources during updates |
| **Custom Resources** | Lambda-backed, SNS-backed |
| **StackSets** | Cross-account, cross-region |

### Template Sections Order

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: ...
Parameters:
  ...
Mappings:
  ...
Conditions:
  ...
Resources:
  ...  # MANDATORY
Outputs:
  ...
```

### Intrinsic Functions Quick Reference

| Function | Use Case |
|----------|----------|
| `!Ref` | Reference Parameter (value) hoặc Resource (ID) |
| `!GetAtt` | Get resource attribute |
| `!FindInMap` | Get value from mapping |
| `!ImportValue` | Import from another stack |
| `!Sub` | String substitution |
| `!Join` | Join strings |
| `!Select` | Select from list |

### Best Practices 2026

1. ✅ Dùng YAML thay vì JSON (dễ đọc, support comments)
2. ✅ Dùng Parameters cho values thay đổi, Mappings cho region/AMI
3. ✅ Dùng Outputs cho cross-stack references
4. ✅ Dùng DeletionPolicy=Retain cho production resources quan trọng
5. ✅ Enable TerminationProtection cho production stacks
6. ✅ Dùng Stack Policy để protect resources khỏi accidental updates
7. ✅ Dùng Custom Resources để handle S3 bucket deletion hoặc unsupported services

---

## Liên kết tham khảo

- [AWS CloudFormation Documentation](https://docs.aws.amazon.com/cloudformation/)
- [CloudFormation Resource Types Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html)
- [CloudFormation Intrinsic Functions](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference.html)

---

*Document này được tạo để hỗ trợ học tập cho kỳ thi AWS Certified Developer - Associate*
