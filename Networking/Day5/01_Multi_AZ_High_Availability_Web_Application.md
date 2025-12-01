# Lab 01 – Multi-AZ High Availability Web Application

**Objective:**  
Deploy a web application that automatically scales and stays online even if an Availability Zone (AZ) fails.  
You will use an **Auto Scaling Group (ASG)** across multiple AZs with an **Application Load Balancer (ALB)**.

---

## Topology

```
            ┌────────────────────────────┐
            │   ALB (HTTP:80)            │
            └──────────┬─────────────────┘
                       │
   ┌───────────────────┴───────────────────┐
   │                                       │
   ▼                                       ▼
[EC2-AZ1]                           [EC2-AZ2]
(Auto Scaling)                   (Auto Scaling)
```

---

## Step 1 – Create a Launch Template

1. Navigate to **EC2 → Launch Templates → Create Launch Template**.  
2. Name: `WebAppTemplate`  
3. AMI: Amazon Linux 2  
4. Instance type: `t2.micro`  
5. User data:

```bash
#!/bin/bash
sudo yum install -y httpd
echo "Hello from $(hostname) in $(curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone)" > /var/www/html/index.html
sudo systemctl start httpd
sudo systemctl enable httpd
```

6. Network: Use `TrainingVPC` created previously.  
7. Security Group: Allow HTTP (80) and SSH (22).

Launch template created successfully.

---

## Step 2 – Create an Auto Scaling Group (ASG)

1. Go to **EC2 → Auto Scaling Groups → Create Auto Scaling Group**.  
2. Name: `WebAppASG`.  
3. Choose `WebAppTemplate`.  
4. Select **two subnets in different AZs** (e.g., `us-east-1a`, `us-east-1b`).  
5. Attach an **Application Load Balancer**:
   - Listener: HTTP port 80
   - Target Group: `WebAppTG` (create new if needed)
6. Set desired capacity: `2`, min `1`, max `3`.  
7. Configure health checks: `ELB` + `EC2`.  

ASG will now deploy instances across AZs.

---

## Step 3 – Test High Availability

1. Get the ALB DNS name from the **Load Balancers** page.  
2. Open it in your browser.  

```
http://WebAppALB-xxxx.elb.amazonaws.com
```

3. Refresh several times → you'll see responses from different AZs.

4. Stop one instance manually.  
ALB should redirect traffic to the healthy instance automatically.  
ASG will launch a new one to maintain capacity.

---

## Verification

| Check | Expected Result |
|--------|------------------|
| ALB DNS responds | ✅ Yes |
| Round-robin responses from both AZs | ✅ Yes |
| Instance failure auto-healed | ✅ Yes |
| Health checks show green | ✅ Yes |

---

## Reflection

- What's the difference between Multi-AZ and Multi-Region?  
- Why does AWS recommend designing for failure?  
- What metrics would you monitor to detect degraded service?

---

## Key Takeaways

| Concept | Description |
|----------|--------------|
| Multi-AZ | Fault tolerance within one region |
| ALB + ASG | Automated scaling and recovery |
| Health Checks | Continuous monitoring of instance status |
| Design for Failure | Core AWS architecture principle |

---

