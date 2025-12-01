# Lab 03 – VPC Peering and Hybrid Connectivity

**Objective:**  
Connect two VPCs privately using **VPC Peering**, simulating a hybrid (on-prem + cloud) connection.  
You'll configure routing and verify private communication between resources.

---

## Topology

```
VPC-A (10.0.0.0/16)         VPC-B (172.16.0.0/16)
┌──────────────┐           ┌──────────────┐
│ WebServerA   │ ⇄ Peering ⇄ │ DBServerB    │
│10.0.1.10     │           │172.16.1.10   │
└──────────────┘           └──────────────┘
```

---

## Step 1 – Create a Second VPC

1. Go to **VPC → Create VPC**.  
2. Name: `OnPremVPC`  
3. CIDR block: `172.16.0.0/16`  
4. Add one subnet: `172.16.1.0/24`.

You now have **TrainingVPC** and **OnPremVPC**.

---

## Step 2 – Create VPC Peering Connection

1. Go to **VPC → Peering Connections → Create Peering Connection**.  
2. Name: `Training-OnPrem-Peer`  
3. Requester: `TrainingVPC (10.0.0.0/16)`  
4. Accepter: `OnPremVPC (172.16.0.0/16)`  
5. Click **Create Peering Connection → Accept Request**.

Status should change to **Active**.

---

## Step 3 – Update Route Tables

### In TrainingVPC
- Add route:
  - Destination: `172.16.0.0/16`
  - Target: `pcx-xxxxx` (Peering ID)

### In OnPremVPC
- Add route:
  - Destination: `10.0.0.0/16`
  - Target: `pcx-xxxxx`

---

## Step 4 – Launch Test Instances

| VPC | Subnet | Instance | IP | SG Rule |
|-----|---------|-----------|----|----------|
| TrainingVPC | PublicSubnetA | WebServerA | 10.0.1.10 | Allow SSH (22), ICMP |
| OnPremVPC | SubnetA | DBServerB | 172.16.1.10 | Allow SSH (22), ICMP |

---

## Step 5 – Test Private Connectivity

1. SSH into WebServerA:

```
ssh ec2-user@<WebServer_Public_IP>
```

2. Ping the DBServerB private IP:

```
ping 172.16.1.10
```

Expected result: Successful private connectivity.

---

## Step 6 – Troubleshoot (Optional)

Use **VPC Reachability Analyzer**:
- Source: WebServerA
- Destination: DBServerB
Confirms packet path via Peering.

---

## Reflection

- Why is VPC Peering **non-transitive**?  
- How does this differ from a **VPN** connection?  
- What are use cases for **PrivateLink**?

---
