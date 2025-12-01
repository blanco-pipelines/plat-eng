# Lab 02 – Building a Load-Balanced Web Tier

**Objective:**  
Create a scalable and fault-tolerant web architecture using **Application Load Balancer (ALB)**.  
You'll deploy two web servers across multiple Availability Zones and test failover behavior.

---

## Topology

```
               ┌────────────┐
Internet ⇄ ALB │ HTTP:80    │
               └────────────┘
               /            \
       EC2-A (AZ1)       EC2-B (AZ2)
```

---

## Step 1 – Launch One new Web Server and reuse the existing one

1. Use your **TrainingVPC** from Day 3 labs.
2. Add a new public subnet :
   - **Subnet name:** `PublicSubnetB` 
   - **Availability Zone:** `us-east-1b` (or your region's second AZ)
   - **IPv4 CIDR block:** `10.0.2.0/24`
   - Select Auto Assign IPv4 adress 
4. Launch one new **EC2 instance** name it **WebServer2** (Amazon Linux 2, t2.micro).
5. Edit VPC and assign TrainingVPC and subnet **PublicSubnetB**
6. Configure user data on bot **Webserver2** and **Webserver**:

```bash
#!/bin/bash
sudo yum install -y httpd
echo "Hello from $(hostname)" > /var/www/html/index.html
sudo systemctl start httpd
sudo systemctl enable httpd
```

> This will install a basic http server to enable testing of our loadbalancer !


---

## Step 2 – Create a Target Group

1. Go to **EC2 → Target Groups → Create Target Group**.  
2. Choose **Instances** as target type.  
3. Name: `WebTG`  
4. Protocol: HTTP  
5. Port: 80  
6. VPC: `TrainingVPC`  
7. Add both EC2 instances as targets.  
8. Health check path: `/`

Wait until targets show as **healthy**.

---

## Step 3 – Create Application Load Balancer

1. Go to **Load Balancers → Create Load Balancer → Application Load Balancer**  
2. Name: `TrainingALB`  
3. Scheme: Internet-facing  
4. Listeners: HTTP (80)  
5. Availability Zones: Select both AZs and attach public subnets.  
6. Security Group: Allow HTTP (80) from 0.0.0.0/0.  
7. Target Group: Select `WebTG`.

ALB will get a **DNS name** like `trainingalb-xxxx.elb.amazonaws.com`.

---

## Step 4 – Test Load Balancing

1. Open your browser:

```
http://trainingalb-xxxx.elb.amazonaws.com
```

2. Refresh several times — you should alternate between:
- "Hello from ip-10-0-1-xx"
- "Hello from ip-10-0-2-xx"

ALB distributes traffic evenly between both servers.

---

## Step 5 – Simulate Failover

1. Stop EC2-B in the AWS Console.  
2. Refresh your browser again.  
All traffic now goes to EC2-A (no downtime).

---

## Verification

| Test | Expected Result |
|------|------------------|
| ALB DNS resolves | ✅ Yes |
| Alternating responses | ✅ Yes |
| Health check status | ✅ Healthy |
| Failover on stop | ✅ Automatic |

---

## Reflection

- What layer (L7 or L4) does the ALB operate at?  
- How do health checks improve availability?  
- Why use multiple AZs instead of one?

---
