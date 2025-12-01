# Lab 02 – VPN Failover and CloudWatch Monitoring

**Objective:**  
Simulate redundancy in hybrid connectivity using **AWS Site-to-Site VPN**.  
You'll observe how AWS automatically switches tunnels during a failure and monitor the event with **CloudWatch metrics**.

---

## Topology

```
On-Prem Network (Customer Gateway)
            │
            │  (Primary Tunnel)
            │
    ┌───────┴────────┐
    │  AWS VPC (VGW) │
    └───────┬────────┘
            │  (Secondary Tunnel)
            ▼
       CloudWatch Alarms
```

---

## Step 1 – Create a Virtual Private Gateway (VGW)

1. Navigate to **VPC → Virtual Private Gateways → Create VGW**.  
   - Name: `TrainingVGW`.  
   - ASN: leave default.  
2. Attach to `TrainingVPC`.

VGW attached to VPC.

---

## Step 2 – Create a Customer Gateway (CGW)

1. Go to **VPC → Customer Gateways → Create CGW**.  
   - Name: `OnPremCGW`.  
   - IP address: use your home router or static test IP (simulate on-prem).  
   - Routing: dynamic (BGP) or static (manual CIDR entry).

---

## Step 3 – Create the VPN Connection

1. Go to **VPC → Site-to-Site VPN Connections → Create VPN Connection**.  
2. Name: `TrainingVPN`.  
3. Select:
   - Target Gateway: `TrainingVGW`
   - Customer Gateway: `OnPremCGW`
   - Routing: static (e.g., 192.168.1.0/24)
4. Create the connection → Download configuration file (Cisco or generic).

Two tunnels are created automatically for redundancy.

---

## Step 4 – Monitor Tunnel State in CloudWatch

1. Go to **CloudWatch → Metrics → VPN → By Connection ID**.  
2. Select metric: `TunnelState`.  
   - `1` = UP  
   - `0` = DOWN  

3. Create an alarm:
   - Metric: `TunnelState`
   - Condition: `< 1` for 1 data point
   - Action: Send notification via SNS topic `VPNAlerts`.

CloudWatch will email you when a tunnel goes down.

---

## Step 5 – Simulate Failover

1. Manually disable the **primary tunnel** (simulate outage).  
2. Observe CloudWatch → one tunnel goes DOWN, the other stays UP.  
3. Verify traffic still passes through VPN (ping test from on-prem host).

---

## Verification

| Test | Expected Result |
|------|------------------|
| One tunnel down | ✅ Secondary active |
| CloudWatch Alarm triggered | ✅ Email received |
| VPN traffic continuous | ✅ Yes |

---

## Reflection

- Why does AWS create two VPN tunnels by default?  
- What happens if both tunnels are down?  
- How can you combine **Direct Connect + VPN** for resilience?

---

