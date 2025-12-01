# Lab 03 – Troubleshooting Common VPC Issues

**Objective:**  
Learn to identify and fix three common networking problems in AWS environments:  
1. **Private instances can't reach Internet** (NAT Gateway missing)  
2. **ALB shows unhealthy targets** (Security Group misconfiguration)  
3. **VPC peering traffic fails** (Route Table misconfigured)

---

## Lab Architecture

```
Internet
    │
   IGW
    │
┌───┴─────────────────────────────────────┐
│            MainVPC (10.0.0.0/16)       │
│  ┌─────────────┐    ┌─────────────┐    │
│  │ Public Sub  │    │ Private Sub │    │
│  │10.0.1.0/24  │    │10.0.2.0/24  │    │
│  │    ALB      │    │ WebServer2  │    │
│  │ WebServer1  │    │             │    │
│  └─────────────┘    └─────────────┘    │
└─────────────────────────────────────────┘
           │ (VPC Peering)
┌──────────┴──────────────────────────────┐
│         PartnerVPC (172.16.0.0/16)     │
│  ┌─────────────┐                       │
│  │   Subnet    │                       │
│  │172.16.1.0/24│                       │
│  │ TestServer  │                       │
│  └─────────────┘                       │
└─────────────────────────────────────────┘
```

---

## Step 1 – Build Working Environment

### Create Main VPC Infrastructure
1. **Create MainVPC:**
   - Go to **VPC → Create VPC**
   - **Name:** `MainVPC`
   - **IPv4 CIDR:** `10.0.0.0/16`

2. **Create Subnets:**
   - **PublicSubnet:** `10.0.1.0/24` in `us-east-1a`
   - **PrivateSubnet:** `10.0.2.0/24` in `us-east-1b`
   - **Enable auto-assign public IPv4** on PublicSubnet

3. **Create Internet Gateway:**
   - **Name:** `MainIGW`
   - **Attach to:** `MainVPC`

4. **Create NAT Gateway:**
   - **Name:** `MainNAT`
   - **Subnet:** `PublicSubnet`
   - **Allocate Elastic IP**

5. **Configure Route Tables:**
   - **PublicRT:** Add route `0.0.0.0/0` → `MainIGW`
   - **PrivateRT:** Add route `0.0.0.0/0` → `MainNAT`

### Create Partner VPC for Peering
6. **Create PartnerVPC:**
   - **Name:** `PartnerVPC`
   - **IPv4 CIDR:** `172.16.0.0/16`
   - **Subnet:** `172.16.1.0/24` in `us-east-1a`
   - **Enable auto-assign public IPv4**

7. **Create VPC Peering Connection:**
   - **Name:** `Main-Partner-Peer`
   - **Requester:** `MainVPC`
   - **Accepter:** `PartnerVPC`
   - **Accept the request**

8. **Add Peering Routes (working configuration):**
   - **MainVPC routes:** Add `172.16.0.0/16` → `pcx-xxxxx` to both PublicRT and PrivateRT
   - **PartnerVPC routes:** Add `10.0.0.0/16` → `pcx-xxxxx`

### Launch EC2 Instances
9. **Create Security Groups:**
   - **WebSG:** Allow SSH (port 22) and HTTP (port 80) from anywhere
   - **TestSG:** Allow SSH (port 22) and ICMP from VPC CIDRs

10. **Launch instances:**
    - **WebServer1:** PublicSubnet, use WebSG
    - **WebServer2:** PrivateSubnet, use WebSG
    - **TestServer:** PartnerVPC subnet, use TestSG

### Install Web Server Software
11. **SSH to both WebServer1 and WebServer2:**
    ```bash
    sudo yum update -y
    sudo yum install -y httpd
    echo "<h1>Server $(hostname)</h1>" | sudo tee /var/www/html/index.html
    sudo systemctl start httpd
    sudo systemctl enable httpd
    ```

### Create Application Load Balancer
12. **Create ALB:**
    - **Name:** `MainALB`
    - **Scheme:** Internet-facing
    - **Subnets:** Both public and private subnets
    - **Security Group:** Allow HTTP from anywhere
    - **Target Group:** Include both WebServers
    - **Health Check Path:** `/` (correct path)

### Verify Everything Works
13. **Test connectivity:**
    - **SSH to WebServer1** via public IP
    - **From WebServer1, SSH to WebServer2** via private IP
    - **From WebServer2, test internet:** `curl -I http://google.com`
    - **Test ALB:** Browse to ALB DNS name
    - **Test peering:** From WebServer1, `ping 172.16.1.x` (TestServer)

 **Everything should work perfectly!**

---

## Step 2 – Break the Environment (Create Problems)

### Break Scenario 1: Remove NAT Gateway Internet Access
1. **Delete NAT Gateway:**
   - Go to **VPC → NAT Gateways → MainNAT**
   - **Actions → Delete NAT Gateway**
   - **Confirm deletion**

2. **Remove route from Private Route Table:**
   - **VPC → Route Tables → PrivateRT → Routes**
   - **Delete the** `0.0.0.0/0` **route**

### Break Scenario 2: Mess Up ALB Health Checks
3. **Change ALB Health Check Path:**
   - **EC2 → Load Balancers → MainALB → Target Groups**
   - **Health checks → Edit**
   - **Change path from** `/` **to** `/health`

4. **Break Security Group Rules:**
   - **EC2 → Security Groups → WebSG**
   - **Remove the HTTP (port 80) inbound rule**

### Break Scenario 3: Remove VPC Peering Routes
5. **Delete peering routes:**
   - **VPC → Route Tables → PublicRT → Routes**
   - **Delete route** `172.16.0.0/16` → `pcx-xxxxx`
   - **VPC → Route Tables → PrivateRT → Routes**  
   - **Delete route** `172.16.0.0/16` → `pcx-xxxxx`
   - **Repeat for PartnerVPC route table**

**Now your environment is properly broken!**

---

## Problem Scenario 1: Private Instance Can't Reach Internet

### Symptom
WebServer2 (in private subnet) cannot download updates or reach external services.

### Investigation Steps
1. **SSH to WebServer1 (public), then SSH to WebServer2 (private):**
   ```bash
   # From WebServer1
   ssh ec2-user@10.0.2.x
   
   # From WebServer2 - test internet connectivity
   curl -I http://google.com
   ping 8.8.8.8
   ```
   **Expected Result:** Connection timeouts

2. **Check route table for private subnet:**
   ```
   VPC → Route Tables → PrivateRT → Routes
   ```
   **Missing:** Route to NAT Gateway for internet access

### Solution
1. **Recreate NAT Gateway:**
   - Go to **VPC → NAT Gateways → Create NAT Gateway**
   - **Name:** `MainNAT-Fixed`
   - **Subnet:** `PublicSubnet` (must be in public subnet)
   - **Connectivity type:** Public
   - **Allocate Elastic IP**

2. **Restore Private Route Table:**
   - **VPC → Route Tables → PrivateRT → Routes → Edit routes**
   - **Add route:** `0.0.0.0/0` → `MainNAT-Fixed`

3. **Verify Fix:**
   ```bash
   # SSH to WebServer1, then SSH to WebServer2
   ssh ec2-user@10.0.2.x
   # From WebServer2
   curl -I http://google.com
   ```
   **Expected Result:** HTTP response received

**Lesson:** Private instances need NAT Gateway for outbound internet access

---

## Problem Scenario 2: ALB Shows Unhealthy Targets

### Symptom
Application Load Balancer shows all targets as "Unhealthy" despite instances running.

### Investigation Steps
1. **Check ALB Target Group Health:**
   ```
   EC2 → Load Balancers → BrokenALB → Target Groups
   ```
   **Status:** Both targets showing "Unhealthy"

2. **Review Health Check Configuration:**
   - **Path:** `/health` (but instances serve content on `/`)
   - **Port:** 80
   - **Protocol:** HTTP

3. **Check Security Group Rules:**
   ```
   EC2 → Security Groups → WebSG-Broken → Inbound rules
   ```
   **Missing:** ALB health check access

### Solution
1. **Fix Health Check Path:**
   - **EC2 → Target Groups → Health checks → Edit**
   - **Health check path:** Change from `/health` back to `/`

2. **Restore Security Group Rules:**
   - **EC2 → Security Groups → WebSG → Inbound rules → Edit**
   - **Add rule:**
     - **Type:** HTTP
     - **Port:** 80  
     - **Source:** Anywhere (0.0.0.0/0) or ALB Security Group

3. **Verify Fix:**
   - **Wait 2-3 minutes for health checks**
   - **EC2 → Load Balancers → MainALB → Target Groups**
   - **Target Group status should show "Healthy"**
   - **Test ALB DNS name in browser**

**Lesson:** ALB health checks must match actual application paths, and security groups must allow ALB traffic

---

## Problem Scenario 3: VPC Peering Traffic Fails

### Symptom
Cannot communicate between MainVPC and PartnerVPC despite peering connection being active.

### Investigation Steps
1. **Verify Peering Connection:**
   ```
   VPC → Peering Connections → Main-Partner-Peer
   ```
   **Status:** Should be "Active"

2. **Test Connectivity:**
   ```bash
   # From WebServer1 (10.0.1.x), try to reach TestServer (172.16.1.x)
   ping 172.16.1.10
   ```
   **Expected Result:** ❌ No response

3. **Check Route Tables:**
   ```
   VPC → Route Tables → MainVPC routes
   VPC → Route Tables → PartnerVPC routes
   ```
   **Missing:** Routes pointing to peering connection

### Solution
1. **Restore Routes in MainVPC:**
   - **VPC → Route Tables → PublicRT → Routes → Edit routes**
   - **Add route:** `172.16.0.0/16` → `pcx-xxxxx` (Peering Connection ID)
   
   - **VPC → Route Tables → PrivateRT → Routes → Edit routes**
   - **Add route:** `172.16.0.0/16` → `pcx-xxxxx` (Peering Connection ID)

2. **Restore Routes in PartnerVPC:**
   - **VPC → Route Tables → PartnerRT → Routes → Edit routes**
   - **Add route:** `10.0.0.0/16` → `pcx-xxxxx` (Peering Connection ID)

3. **Verify Fix:**
   ```bash
   # From WebServer1
   ping 172.16.1.x  # TestServer IP
   ssh ec2-user@172.16.1.x
   ```
   **Expected Result:** ✅ Connection successful

**Lesson:** VPC peering requires bidirectional route table entries - creating the peering connection is only half the work

---

## Final Verification Checklist

| Problem | Symptom | Root Cause | Solution | Status |
|---------|---------|------------|----------|--------|
| Internet Access | Private instance timeout | Missing NAT Gateway | Added NAT + route | ✅ |
| ALB Health | Unhealthy targets | Wrong health check + SG | Fixed path + rules | ✅ |
| VPC Peering | Cross-VPC ping fails | Missing routes | Added peering routes | ✅ |

---

## Troubleshooting Best Practices

### Systematic Approach
1. **Layer by Layer:** Physical → Network → Transport → Application
2. **Inside-Out:** Start from source, trace path to destination
3. **Use AWS Tools:** VPC Flow Logs, Reachability Analyzer, CloudWatch

### Common Debugging Commands
```bash
# Network connectivity tests
ping <destination>
curl -I <url>
telnet <host> <port>

# Route and DNS checks  
ip route show
nslookup <hostname>
dig <domain>

# AWS CLI diagnostics
aws ec2 describe-route-tables
aws ec2 describe-security-groups
aws elbv2 describe-target-health
```

### Prevention Tips
- **Document network architecture** before making changes
- **Test connectivity** after each configuration change  
- **Use Infrastructure as Code** (CloudFormation/Terraform) for consistency
- **Monitor with CloudWatch** and set up alerts for failures

---
