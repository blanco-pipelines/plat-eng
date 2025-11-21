# Lab 03 – AWS Network ACLs (NACLs)

**Objective:**  
Configure Network ACLs (NACLs) to manage subnet-level traffic control.  
You'll implement explicit allow and deny rules and compare their behavior to Security Groups.

---

## Topology

```
                Internet
                   │
              [IGW – TrainingIGW]
                   │
         ┌──────────────────────────────┐
         │ VPC (10.0.0.0/16)           │
         │ ┌────────────┐ ┌────────────┐
         │ │PublicSubnet│ │PrivateSubnet│
         │ │Web Server  │ │DB Server   │
         └──────────────────────────────┘
```

> You will modify the **NACLs** associated with each subnet created in **Lab 05**.

---

## Step 1 – Locate Default NACL

1. In the **VPC Dashboard**, go to **Network ACLs**.  
2. Find the **default NACL** for your VPC.  
   - It allows **all inbound and outbound traffic** by default.

---

## Step 2 – Create Custom NACLs

### a. Public NACL
1. Click **Create network ACL**.  
   - **Name:** `PublicNACL`
   - **VPC:** `TrainingVPC`
2. Add **Inbound Rules**:
   | Rule # | Type | Protocol | Port | Source | Allow/Deny |
   |---------|------|-----------|-------|---------|------------|
   | 100 | HTTP | TCP | 80 | 0.0.0.0/0 | ALLOW |
   | 110 | SSH | TCP | 22 | Your_IP/32 | ALLOW |
   | 200 | ALL | ALL | ALL | 0.0.0.0/0 | DENY |
3. Add **Outbound Rules**:
   | Rule # | Type | Protocol | Port | Destination | Allow/Deny |
   |---------|------|-----------|-------|--------------|------------|
   | 100 | ALL | ALL | ALL | 0.0.0.0/0 | ALLOW |

4. Associate this NACL with **PublicSubnetA**.

---

### b. Private NACL
1. Create another NACL:
   - **Name:** `PrivateNACL`
   - **VPC:** `TrainingVPC`
2. Add **Inbound Rules**:
   | Rule # | Type | Protocol | Port | Source | Allow/Deny |
   |---------|------|-----------|-------|---------|------------|
   | 100 | MySQL | TCP | 3306 | 10.0.1.0/24 | ALLOW |
   | 110 | SSH | TCP | 22 | 10.0.1.0/24 | ALLOW |
   | 200 | ALL | ALL | ALL | 0.0.0.0/0 | DENY |
3. Add **Outbound Rules**:
   | Rule # | Type | Protocol | Port | Destination | Allow/Deny |
   |---------|------|-----------|-------|--------------|------------|
   | 100 | ALL | ALL | ALL | 0.0.0.0/0 | ALLOW |

> **Note:** Outbound rules are stateless; you must explicitly allow return traffic on all ports

4. Associate this NACL with **PrivateSubnetB**.

---

## Step 3 – Test Connectivity

1. **From your computer:**
   - SSH to WebServer → Should work.
   - Try to SSH to DBServer (private subnet) → Should fail.

2. **From WebServer (inside VPC):**

```
ssh ec2-user@10.0.2.10
```

Should work (NACL allows internal 10.0.1.0/24).

3. Open browser → `http://<WebServer_Public_IP>` → Should load.

4. Modify a rule (e.g., DENY port 80 in Public NACL) → test again → Web access fails.

---

## Verification Checklist

| Test | Expected Result |
|------|------------------|
| HTTP to WebServer | Allowed |
| SSH from your IP | Allowed |
| SSH to DBServer from outside | Denied |
| WebServer → DBServer (port 3306) | Allowed |

---

## Reflection Questions

- How are NACLs different from Security Groups?  
- Why do NACLs have **explicit deny** rules?  
- What happens if rule numbers conflict or overlap?  
- How can NACLs and SGs work together for layered defense?

---

## Key Takeaways

| Concept | Description |
|----------|--------------|
| NACL | Subnet-level firewall |
| Stateless | Requires explicit inbound & outbound rules |
| Rule Order | Lowest number first |
| Deny Support | Explicit deny possible |
| Layered Security | Use with SGs for defense-in-depth |

---

