Sure! Here's the improved **Markdown-formatted** version of the overview:

---

# **AWS CloudFormation Template Overview**

This AWS CloudFormation template is designed to update **AWS Config Recorder settings** in child accounts created by **AWS Control Tower**. It primarily manages:

✅ **Lambda functions**  
✅ **IAM roles**  
✅ **S3 buckets**  
✅ **SQS queues**  
✅ **EventBridge rules**

to automate AWS Config updates in child accounts.

---

## **🔹 Parameters**

| **Parameter**               | **Description** |
|-----------------------------|----------------|
| `DisabledAccounts`          | List of AWS accounts where AWS Config Recorder should be **disabled**. |
| `EnabledAccounts`           | List of accounts where AWS Config Recorder should be **enabled** (e.g., management, log archive, audit accounts). |
| `ConfigRecorderResourceTypes` | List of AWS **resource types to exclude** from AWS Config Recorder. |
| `CloudFormationVersion`     | Specifies the CloudFormation **template version**. |

---

## **🔹 Resources**

### **📌 S3 Bucket**
- **`LambdaZipsBucket`**:
    - Stores the **Lambda function code ZIP files**.
    - Uses **AES-256 encryption** for security.

### **📌 AWS Lambda Functions**

#### 🏗 **1. ProducerLambda**
🔹 **Purpose:**
- Triggers when **AWS Control Tower events** occur (e.g., new accounts, updates).
- Retrieves **account lists** and sends messages to the **SQS queue**.

🔹 **Configuration:**
- Runtime: **Python 3.10**
- Memory: **128 MB**
- Timeout: **300 seconds**
- Architecture: **x86_64**

#### 🏗 **2. ConsumerLambda**
🔹 **Purpose:**
- Reads messages from the **SQS queue** and **updates AWS Config Recorder** in child accounts.

🔹 **Configuration:**
- Runtime: **Python 3.10**
- Memory: **128 MB**
- Timeout: **180 seconds**
- Architecture: **x86_64**

### **📌 Event Source Mapping**
- **`ConsumerLambdaEventSourceMapping`**:
    - Links **ConsumerLambda** to the **SQS queue**, allowing it to process messages.

### **📌 IAM Roles**

| **IAM Role**                    | **Purpose** |
|----------------------------------|------------|
| `ProducerLambdaExecutionRole`    | Grants permissions for **CloudFormation StackSet operations** and **SQS interactions**. |
| `ConsumerLambdaExecutionRole`    | Grants **STS AssumeRole** permissions and **SQS access**. |
| `CopyZipsRole`                   | Allows **S3 read/write** operations to transfer Lambda ZIP files. |

### **📌 SQS Queue**
- **`SQSConfigRecorder`**:
    - Stores messages for **processing AWS Config Recorder updates**.
    - Uses **AWS KMS encryption** for security.

### **📌 EventBridge Rule**
- **`ProducerEventTrigger`**:
    - **Triggers ProducerLambda** when **AWS Control Tower events** occur:
        - `UpdateLandingZone`
        - `CreateManagedAccount`
        - `UpdateManagedAccount`

### **📌 Custom Resource: Lambda Function for Copying ZIP Files**
- **`CopyZipsFunction`**:
    - Copies **Lambda function ZIPs** from an external S3 bucket to the internal bucket.
    - Deletes ZIP files if the stack is removed.

---

## **⚙️ How It Works**

1️⃣ **An AWS Control Tower event occurs** → **Triggers ProducerLambda**.  
2️⃣ **ProducerLambda sends relevant account details** to the **SQS queue**.  
3️⃣ **ConsumerLambda processes messages** from the **SQS queue** and updates AWS Config Recorder settings.  
4️⃣ **Lambda ZIP files are copied** from an external S3 bucket to an internal S3 bucket via **CopyZipsFunction**.

---

## **🚀 Use Case**
- **Automates AWS Config Recorder settings** for **child accounts in AWS Control Tower**.
- Ensures that **management, log archive, and audit accounts** always have AWS Config **enabled**.
- **Reduces manual effort** in managing AWS Config settings across multiple AWS accounts.

---

## **🚀 How to deploy**
