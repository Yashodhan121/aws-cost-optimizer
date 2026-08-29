
# AWS Automated Cost Optimizer 💰☁️

An automated AWS cost optimization system that identifies eligible EC2 instances using tags and automatically stops them on a scheduled basis using **Amazon EventBridge Scheduler and AWS Lambda**.

---

## 📌 Project Overview

Running unused EC2 instances can increase AWS costs unnecessarily.

This project provides a simple automated solution that identifies EC2 instances marked for cost optimization and stops them automatically according to a scheduled policy.

The system uses **EC2 tags** to determine which instances are eligible for automatic stopping.

### Main objective

> Automatically identify selected EC2 instances and stop them during scheduled periods to reduce unnecessary AWS compute costs.

---

## 🏗️ Architecture

```text
                  ┌─────────────────────────┐
                  │  Amazon EventBridge     │
                  │       Scheduler         │
                  │                         │
                  │  Daily at 8:00 PM IST   │
                  └────────────┬────────────┘
                               │
                               │ { "action": "stop" }
                               ▼
                  ┌─────────────────────────┐
                  │      AWS Lambda         │
                  │                         │
                  │ automated-aws-cost-     │
                  │ optimizer               │
                  └────────────┬────────────┘
                               │
                               │ Check EC2 Tags
                               ▼
                  ┌─────────────────────────┐
                  │       Amazon EC2        │
                  │                         │
                  │ cost-optimizer-test-    │
                  │ server                  │
                  └────────────┬────────────┘
                               │
                         Eligible?
                         /       \
                       Yes        No
                        │          │
                        ▼          ▼
                    STOP EC2    No Action
                        │
                        ▼
                  ┌─────────────────────────┐
                  │    Amazon CloudWatch    │
                  │                         │
                  │ Execution & decision    │
                  │ logs                    │
                  └─────────────────────────┘
```

---

## ☁️ AWS Services Used

| AWS Service                      | Purpose                                                 |
| -------------------------------- | ------------------------------------------------------- |
| **Amazon EC2**                   | Provides the compute instance managed by the optimizer  |
| **AWS Lambda**                   | Contains the automation logic                           |
| **Amazon EventBridge Scheduler** | Executes the Lambda automatically on a schedule         |
| **AWS IAM**                      | Provides required permissions to Lambda and EventBridge |
| **Amazon CloudWatch**            | Stores Lambda execution logs and monitoring information |

---

## ⚙️ How It Works

The system follows these steps:

1. An EC2 instance is created for testing.
2. Specific tags are assigned to identify resources that can be automatically stopped.
3. AWS Lambda scans EC2 instances using the configured tag criteria.
4. Lambda determines whether an instance is eligible.
5. A safe `check` action can be used to identify eligible instances without modifying them.
6. EventBridge Scheduler invokes Lambda every day at **8:00 PM IST**.
7. The scheduled event sends:

```json
{
  "action": "stop"
}
```

8. Lambda identifies eligible EC2 instances.
9. Eligible instances are stopped automatically.
10. Lambda execution details are recorded in CloudWatch Logs.

---

## 🏷️ EC2 Tag-Based Selection

The test EC2 instance uses the following tags:

| Tag Key         | Value                        |
| --------------- | ---------------------------- |
| `Project`       | `AWS-Cost-Optimizer`         |
| `CostOptimizer` | `enabled`                    |
| `Environment`   | `dev`                        |
| `AutoStop`      | `true`                       |
| `AutoSchedule`  | `yes`                        |
| `Name`          | `cost-optimizer-test-server` |

These tags allow the Lambda function to identify resources that are intended for automatic cost optimization.

---

## 🔍 Safe CHECK Mode

Before performing any EC2 action, the Lambda function supports a safe `check` mode.

Example:

```json
{
  "action": "check"
}
```

The Lambda checks the instance and reports whether it is eligible without starting or stopping any EC2 instance.

Example CloudWatch output:

```text
AWS COST OPTIMIZER
Requested action: check

Instance ID: i-0434c743a79daf230
Current state: running
Decision: ELIGIBLE

CHECK MODE: No EC2 action will be performed.

Eligible instances: ['i-0434c743a79daf230']

No EC2 instances were started or stopped.
```

This provides a safe way to verify the selection logic before enabling automatic actions.

---

## 🛑 Automated STOP Action

For the actual automation, the Lambda receives:

```json
{
  "action": "stop"
}
```

The Lambda then evaluates the EC2 instance tags and stops eligible instances.

The action was tested successfully using the project test instance:

```text
Instance Name: cost-optimizer-test-server
Instance ID: i-0434c743a79daf230
Instance Type: t3.micro
Region: ap-south-1
```

---

## ⏰ EventBridge Scheduler

The project uses an Amazon EventBridge Scheduler named:

```text
aws-cost-optimizer-daily
```

### Schedule configuration

```text
Schedule: aws-cost-optimizer-daily
Status: Enabled
Frequency: Daily
Time: 8:00 PM IST
Time Zone: Asia/Calcutta
```

The scheduler invokes:

```text
automated-aws-cost-optimizer
```

with the payload:

```json
{
  "action": "stop"
}
```

---

## 🔐 IAM Permissions

IAM roles are used to provide the required permissions without embedding AWS credentials inside the Lambda code.

The Lambda execution role requires permissions to:

* Describe EC2 instances
* Stop EC2 instances
* Write logs to CloudWatch

EventBridge Scheduler uses an execution role that allows it to invoke the Lambda function.

### Security principle

The project follows the principle of **least privilege** by granting only the permissions required for the automation.

---

## 📊 CloudWatch Monitoring

AWS CloudWatch Logs is used to monitor Lambda executions.

The Lambda creates the following log group:

```text
/aws/lambda/automated-aws-cost-optimizer
```

The logs record:

* Requested action
* Execution time
* Instance ID
* Current EC2 state
* Eligibility decision
* Stop operation
* Execution result

Example:

```text
AWS COST OPTIMIZER
Requested action: check
Instance ID: i-0434c743a79daf230
Current state: running
Decision: ELIGIBLE
CHECK MODE
No EC2 instances were started or stopped.
```

---

## 🧪 Testing

The project was tested in multiple stages.

### Test 1 — Instance Detection

The Lambda successfully detected the tagged EC2 instance.

```text
Instance ID: i-0434c743a79daf230
Current state: running
Decision: ELIGIBLE
```

### Test 2 — CHECK Mode

The Lambda successfully identified the eligible instance without modifying it.

```text
CHECK MODE: No EC2 action will be performed.
```

### Test 3 — STOP Action

The actual stop operation was tested successfully.

The EC2 instance transitioned from:

```text
Running
   ↓
Stopping
   ↓
Stopped
```

### Test 4 — Scheduled Automation

EventBridge Scheduler was configured to automatically invoke the Lambda every day at 8:00 PM IST.

---

## 📁 Project Structure

```text
aws-cost-optimizer/
│
├── lambda_function.py
│
├── README.md
│
└── screenshots/
    ├── ec2-instance-tags.png
    ├── lambda-function.png
    ├── lambda-check-test.png
    ├── lambda-stop-test.png
    ├── eventbridge-schedule.png
    └── cloudwatch-logs.png
```

> The exact file structure may vary depending on how the Lambda code is uploaded to AWS.

---

## 🚀 Deployment Steps

### 1. Create an EC2 Instance

Create a test EC2 instance and apply the required cost optimizer tags.

### 2. Create the IAM Role

Create an IAM execution role for Lambda with the required EC2 and CloudWatch permissions.

### 3. Create the Lambda Function

Create:

```text
automated-aws-cost-optimizer
```

and upload the Python automation code.

### 4. Test CHECK Mode

Use:

```json
{
  "action": "check"
}
```

to verify that the correct EC2 instances are identified.

### 5. Test STOP Mode

Use:

```json
{
  "action": "stop"
}
```

to verify the actual EC2 stop operation.

### 6. Configure EventBridge Scheduler

Create:

```text
aws-cost-optimizer-daily
```

and configure it to run daily at:

```text
8:00 PM IST
```

### 7. Connect Lambda

Set the Lambda function as the EventBridge target and use:

```json
{
  "action": "stop"
}
```

as the target payload.

### 8. Monitor with CloudWatch

Verify Lambda execution logs in:

```text
/aws/lambda/automated-aws-cost-optimizer
```

---

## 💡 Benefits

* Reduces unnecessary EC2 running time
* Automates cost-saving operations
* Uses tag-based resource selection
* Avoids manual daily intervention
* Provides a safe CHECK mode
* Provides CloudWatch execution logs
* Uses serverless Lambda architecture
* Uses scheduled automation through EventBridge

---

## 🔒 Safety Considerations

Automatic stopping of EC2 instances can affect running applications.

Therefore:

* Only instances with the appropriate tags should be eligible.
* Production resources should not be tagged for automatic stopping unless explicitly intended.
* CHECK mode should be used before enabling automatic actions.
* IAM permissions should follow least-privilege principles.
* Important workloads should be excluded from the automation.

---

## 🔮 Future Enhancements

Possible improvements include:

* Automatic start/stop schedules based on environment
* Cost estimation and monthly savings reports
* SNS email notifications after an instance is stopped
* DynamoDB-based optimization history
* Web dashboard for cost optimization statistics
* Detection of idle EC2 instances using CloudWatch metrics
* Support for EBS volume optimization
* Automated reports showing estimated savings
* Multi-account and multi-region support

---

## 🎓 Project Information

**Project Title:** Automated AWS Cost Optimizer

**Cloud Platform:** Amazon Web Services (AWS)

**Primary Services:** EC2, Lambda, EventBridge Scheduler, IAM, CloudWatch

**Region:** Asia Pacific (Mumbai) — `ap-south-1`

**Automation Frequency:** Daily at 8:00 PM IST

**Project Type:** Cloud Automation / Serverless / Cost Optimization

---

## 👨‍💻 Author

**Yashodhan Kolhe**

MCA — Master of Computer Applications

---

## 📜 Conclusion

The AWS Automated Cost Optimizer demonstrates how AWS serverless services can be combined to automate cloud cost-management tasks.

The solution identifies eligible EC2 instances using tags, validates them through a safe CHECK mode, and automatically stops selected instances according to a scheduled EventBridge policy. CloudWatch provides execution visibility and logging, while IAM provides controlled access to AWS resources.

This project demonstrates practical knowledge of:

* AWS Lambda
* Amazon EC2
* EventBridge Scheduler
* IAM
* CloudWatch
* Serverless automation
* Tag-based resource management
* AWS cost optimization
