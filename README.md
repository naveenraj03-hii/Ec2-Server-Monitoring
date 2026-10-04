# 🚀 AWS EC2 Auto-Recovery Automation using Python & Boto3

A Python-based AWS automation project that continuously monitors multiple Amazon EC2 instances and automatically starts an instance when its state changes to `stopped`.

The monitoring agent uses **Python + Boto3** and runs continuously inside a **Docker container**.

---

## 📌 Project Overview

In this project, a Python monitoring agent monitors **two EC2 instances** in the AWS Mumbai (`ap-south-1`) region.

The agent checks the state of both EC2 instances at a **10-second interval**.

If any monitored EC2 instance changes from:

```text
running → stopped
```

the Python agent detects the stopped state and automatically calls the AWS EC2 API:

```python
start_instances()
```

The **same stopped instance** is then started again.

### Architecture

```text
                    ┌─────────────────────────────┐
                    │       Docker Container       │
                    │        Python 3.12           │
                    │       Non-root User          │
                    └──────────────┬──────────────┘
                                   │
                                   ▼
                    ┌─────────────────────────────┐
                    │    Python Monitoring Agent   │
                    │          + Boto3             │
                    └──────────────┬──────────────┘
                                   │
                         Monitor EC2 Instances
                                   │
                    ┌──────────────┴──────────────┐
                    │                             │
                    ▼                             ▼
          ┌─────────────────┐           ┌─────────────────┐
          │   EC2 Instance 1│           │   EC2 Instance 2│
          │   ap-south-1    │           │   ap-south-1    │
          └────────┬────────┘           └────────┬────────┘
                   │                             │
                   ▼                             ▼
             Check State                   Check State
                   │                             │
             stopped?                       stopped?
                   │                             │
                  YES                           YES
                   │                             │
                   ▼                             ▼
        start_instances()             start_instances()
                   │                             │
                   ▼                             ▼
               Running                       Running
```

---

## 🔄 How It Works

The automation follows this workflow:

### 1. Start the monitoring agent

The Python agent starts inside a Docker container.

### 2. Connect to AWS

Boto3 is used to communicate with Amazon EC2.

### 3. Monitor both instances

The agent checks the state of:

```text
EC2 Instance 1
EC2 Instance 2
```

### 4. Detect stopped instances

If an instance is:

```text
running
```

the agent continues monitoring.

If the instance becomes:

```text
stopped
```

the agent detects the change.

### 5. Automatically recover

The agent calls:

```python
ec2.start_instances(InstanceIds=[instance_id])
```

This starts the **same EC2 instance** that was stopped.

### 6. Continue monitoring

After the recovery operation, the agent continues checking both instances every **10 seconds**.

---

## 🔁 Recovery Flow

```text
EC2 Instance
     │
     ▼
  Running
     │
     │ User/AWS stops instance
     ▼
  Stopped
     │
     ▼
Python Agent detects stopped state
     │
     ▼
Boto3 start_instances()
     │
     ▼
EC2 starts
     │
     ▼
  Running ✅
     │
     ▼
Monitoring continues
```

---

## 🖥️ Multiple EC2 Instance Monitoring

The project monitors **two EC2 instances**.

### EC2 Instance 1

```text
Running
   ↓
Stopped
   ↓
Detected by Python Agent
   ↓
start_instances()
   ↓
Running Again
```

### EC2 Instance 2

```text
Running
   ↓
Stopped
   ↓
Detected by Python Agent
   ↓
start_instances()
   ↓
Running Again
```

The agent handles each instance independently.

---

## ⏱️ Monitoring Interval

The agent continuously checks the EC2 instance state every:

```text
10 seconds
```

Example:

```text
08:03:52 → EC2-1: running
08:04:02 → EC2-1: running
08:04:12 → EC2-1: stopped → recovery triggered
```

The exact log timestamps will depend on when the monitoring agent is running.

---

## 🐍 Python & Boto3

The project uses the AWS SDK for Python:

```text
Boto3
```

Boto3 allows the Python application to interact with AWS services.

For this project, the EC2 client is used to retrieve instance states and start stopped instances.

Example:

```python
import boto3

ec2 = boto3.client("ec2", region_name="ap-south-1")

response = ec2.describe_instances(
    InstanceIds=instance_ids
)

ec2.start_instances(
    InstanceIds=[instance_id]
)
```

---

## 🐳 Docker

The monitoring agent is containerized using Docker.

### Docker environment

```text
Python 3.12
Docker
Non-root user
```

Example Dockerfile:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY agent.py .

RUN useradd -u 10001 -m appuser

USER 10001

CMD ["python", "-u", "agent.py"]
```

Running the application as a non-root user provides an additional security layer inside the container.

---

## 📦 Python Dependencies

The project uses:

```text
boto3
```

Example `requirements.txt`:

```text
boto3>=1.35,<2
```

Install dependencies manually with:

```bash
pip install -r requirements.txt
```

---

## 🔐 AWS IAM Permissions

The Python agent requires permission to:

```text
ec2:DescribeInstances
ec2:StartInstances
```

A minimal IAM policy can be configured for the automation.

Example:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeInstances",
        "ec2:StartInstances"
      ],
      "Resource": "*"
    }
  ]
}
```

> For production environments, permissions should be restricted as much as possible instead of using `"Resource": "*"` where the service supports resource-level restrictions.

---

## 📝 Logging & Exception Handling

The monitoring agent uses structured logging to help track:

* EC2 instance states
* Monitoring activity
* Recovery actions
* AWS API errors
* Unexpected exceptions

Example:

```text
INFO - EC2 Instance i-xxxxxxxxxxxx is running
INFO - EC2 Instance i-xxxxxxxxxxxx is stopped
INFO - Starting EC2 Instance i-xxxxxxxxxxxx
INFO - EC2 Instance i-xxxxxxxxxxxx started successfully
```

Exception handling helps prevent the monitoring process from terminating unexpectedly because of a temporary AWS/API error.

---

## 📁 Project Structure

```text
aws-ec2-auto-recovery/
│
├── agent.py
├── requirements.txt
├── Dockerfile
├── .dockerignore
├── .gitignore
└── README.md
```

---

## ⚙️ Configuration

Update the AWS region and EC2 instance IDs in the Python configuration.

Example:

```python
REGION = "ap-south-1"

INSTANCE_IDS = [
    "YOUR_EC2_INSTANCE_ID_1",
    "YOUR_EC2_INSTANCE_ID_2"
]
```

**Do not commit real AWS credentials or sensitive information to GitHub.**

---

## 🏗️ Setup

### Step 1 — Clone the repository

```bash
git clone <your-repository-url>
cd aws-ec2-auto-recovery
```

### Step 2 — Install dependencies

```bash
pip install -r requirements.txt
```

### Step 3 — Configure AWS credentials

Configure AWS credentials using a secure method such as:

```bash
aws configure
```

or use an appropriate IAM role when running the container on AWS infrastructure.

### Step 4 — Configure EC2 instance IDs

Update:

```python
INSTANCE_IDS = [
    "YOUR_EC2_INSTANCE_ID_1",
    "YOUR_EC2_INSTANCE_ID_2"
]
```

### Step 5 — Run the agent

```bash
python agent.py
```

---

# 🐳 Run with Docker

### Build the Docker image

```bash
docker build -t ec2-auto-recovery .
```

### Run the container

```bash
docker run -d \
  --name ec2-auto-recovery \
  ec2-auto-recovery
```

### View logs

```bash
docker logs -f ec2-auto-recovery
```

---

## 🧪 Testing the Automation

To test the recovery mechanism:

### 1. Start the Python monitoring agent

Make sure both EC2 instances are running.

### 2. Stop one EC2 instance

For example:

```text
EC2 Instance 1
Running → Stopped
```

### 3. Wait for the monitoring interval

The agent checks the instance approximately every:

```text
10 seconds
```

### 4. Agent detects the stopped state

The Python agent identifies:

```text
State = stopped
```

### 5. Boto3 starts the instance

The agent executes:

```python
start_instances()
```

### 6. Verify recovery

The instance should transition back toward:

```text
Running
```

The monitoring agent then continues monitoring both instances.

---

## 📊 Key Features

| Feature                   | Description                            |
| ------------------------- | -------------------------------------- |
| Multi-instance monitoring | Monitors two EC2 instances             |
| Auto-recovery             | Automatically starts stopped instances |
| AWS SDK                   | Uses Python Boto3                      |
| Continuous monitoring     | Checks every 10 seconds                |
| Logging                   | Tracks monitoring and recovery events  |
| Exception handling        | Handles runtime/API errors             |
| Docker                    | Runs the agent inside a container      |
| Python 3.12               | Application runtime                    |
| Non-root container        | Improved container security            |
| IAM                       | Controls AWS API permissions           |

---

## 🛠️ Technologies Used

```text
Python 3.12
AWS EC2
AWS Boto3
AWS IAM
Docker
Linux
```

---

## 🎯 What I Learned

Through this project, I practiced:

* Automating AWS infrastructure using Python
* Working with the AWS Boto3 SDK
* Monitoring EC2 instance states
* Calling AWS EC2 APIs programmatically
* Implementing automatic recovery
* Writing logging and exception handling
* Containerizing Python applications with Docker
* Running containers using non-root users
* Understanding IAM permissions for AWS automation
* Building a continuously running cloud automation agent

---

## 🚀 Future Improvements

Possible enhancements for this project include:

* Add **Amazon CloudWatch** monitoring and alarms
* Send recovery notifications using **Amazon SNS**
* Store monitoring events in **CloudWatch Logs**
* Add health checks and retry mechanisms
* Add configuration through environment variables
* Add Docker health checks
* Deploy the monitoring agent on an AWS compute service
* Add a dashboard for monitoring multiple EC2 instances

---

## 👨‍💻 Author

**Naveen Raj K**

Aspiring Cloud & DevOps Engineer

**Skills:** AWS | Linux | Docker | Kubernetes | Terraform | Git | Python | Boto3

---

⭐ If you find this project useful, feel free to star the repository!
