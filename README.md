# Infoblox-workshop
# Infoblox Workshop — Lab Guide
**Duration:** 4 hours total | **Labs:** ~90 minutes hands-on  
**Environment:** Infoblox NIOS (lab/demo appliance or vNIOS)  
**Skill level:** Mixed — project managers observe, engineers execute

---

## Lab Environment Overview

| Component | Details |
|-----------|---------|
| Grid Master | LAB-IBX-GM |
| Grid Member 1 | LAB-IBX-Member-1 (DNS + DHCP) |
| Grid Member 2 | LAB-IBX-Member-2 (DNS only) |
| Admin account | admin / infoblox |
| Lab domain | workshop.lab |
| Lab network | 192.168.20.0/24 |

> **Note:** Replace the IPs above with your actual lab environment addresses.

---

## Lab 1 — Navigation, Zones & DNS Records
**Estimated time:** 45 minutes  
**Objective:** Explore the GUI, create zones and manage host records

---

### 1.1 — Log In and Explore the Dashboard

1. Open a browser login to the lab and navigate to `LAB-IBX-GM`
2. Log in:
   - Username: `admin`
   - Password: `infoblox`
3. You land on the **Dashboard**. Identify the following widgets:
   - Grid health status panel
   - DNS/DHCP service status per member
   - Top CPU / memory usage
4. Navigate to **Infoblox Grid → Grid Manager** — note the Grid Master and Members listed.

> **✅ Expected:** Both members show green / Running status.

---

### 1.2 — Explore Grid Members

1. Go to **Grid → Grid Manager → Members** tab.
2. Click on **Member 1** to open the member detail page.
3. Review:
   - Services running (DNS, DHCP)(Members tab)
   - Hardware type / NIOS version (Upgrade tab)
   - License information (Licenses tab)
4. Notice the sub-tabs.

> **Discussion:** What would happen if Member 1 went offline? What services would be affected?

---

### 1.3 — Examine Existing DNS Zones

1. Go to **Data Management → DNS → Zones**.
2. Expand the `workshop.lab` zone and browse the records inside.
3. Locate and note:
   - SOA record — which server is listed as primary?
   - NS records — how many name servers?
   - An existing A record
4. Switch to the **Reverse Mapping Zones** tab.
5. Find the reverse zone for `192.168.20.x` (should be `20.168.192.in-addr.arpa`).

---

### 1.4 — Create a Forward DNS Zone

1. In **Data Management → DNS → Zones**, click **+ Add → Add Authoritative Zone**.
2. Configure:
   - FQDN: `team-[yourname].workshop.lab` (e.g. `team-jan.workshop.lab`)
   - Zone type: **Primary**
   - Grid Primary: select **Member 1**
   - Click **Next**
3. Leave defaults on the next screen and click **Save & Close**.
4. Verify the new zone appears in the zone list.

> **✅ Expected:** Zone is created and shows green / Active.

---

### 1.5 — Create Host Records

Now add records to your newly created zone.

**Create a server record:**
1. Click into your zone.
2. Click **+ Add → Host Record**.
3. Fill in:
   - **Name:** `webserver1.team-[yourname].workshop.lab`
   - **IPv4 Address:** `192.168.20.10<student number>`
   - **Comment:** `Lab web server`
4. Click **Save & Close**.

**Create a second host:**
1. Repeat the steps for:
   - Name: `appserver1.team-[yourname].workshop.lab`
   - IP: `192.168.20.20<student-number>`
   - PTR: checked

**Create a CNAME alias:**
1. Click **+ Add → CNAME Record**.
2. Fill in:
   - **Name:** `www.team-[yourname].workshop.lab`
   - **Canonical Name:** `webserver1.team-[yourname].workshop.lab`
3. Save & Close.

> **✅ Verify:** Use GUI **Search** → to test: `webserver1.team-[yourname].workshop.lab` should return `192.168.20.10<student number>`.

---

### 1.6 — Verify PTR Records

1. Navigate to **Data Management → DNS → Zones → Reverse Mapping Zones**.
2. Open `20.168.192.in-addr.arpa`.
3. Confirm PTR records exist for `192.168.20.10<student number>` and `192.168.20.20<student number>`.

> **Discussion:** When would a missing PTR record cause problems in production?

---

### 1.7 — Search the GUI

1. Use the **Global Search bar** (top of page).
2. Search for `192.168.20.101` — what objects appear?
3. Search for `webserver*` — see all matching host records.
4. Search for `team-[yourname]` — see all objects in your zone.

> **Tip:** Search works across DNS records, DHCP leases, and IPAM entries simultaneously.

---

### 1.8 — DNS Views (Demonstration / Optional)

> This step is demonstration only if time is limited.

1. Go to **Data Management → DNS → Views**.
2. Observe the existing views (typically `default` and optionally `external`).
3. Note how each view has its own zone list.
4. Open a view and check the **Match Clients** setting — which networks/IPs use this view?

> **Discussion:** How would you configure split-DNS so internal users get RFC-1918 addresses for `mail.company.com` while external users see the public IP?

---

### 1.9 — Examine ACLs

1. Go to **Data Management → DNS → Named ACL**.
2. Review any existing Named ACLs.
3. Create a new Named ACL:
   - Name: `lab-clients-[yourname]`
   - Add element: your lab subnet `192.168.20.0/24`
4. Save. (You do not need to apply it — this is for understanding.)

---

## Lab 2 — DHCP, IPAM & Troubleshooting
**Estimated time:** 45 minutes  
**Objective:** Configure DHCP, explore IPAM, and practice troubleshooting techniques

---

### 2.1 — Explore Existing DHCP Configuration

1. Go to **Data Management → DHCP → Networks**.
2. Click on the `192.168.20.0/24` network.
3. Review:
   - Configured DHCP ranges
   - Fixed addresses (reservations)
   - DHCP options configured (gateway, DNS servers)
4. Click on an active DHCP range and note the utilization percentage.

---

### 2.2 — Create a New DHCP Network

1. In **Data Management → DHCP → Networks**, click **+ Add → Add IPv4 Network**.
2. Configure:
   - Network: `10.20.<student number>.0/24` (trainer will assign your /24)
   - Comment: `Lab network for [yourname]`
   - Click **Next**
3. On the DHCP Member screen, assign **Member 1** as the DHCP server.
4. Click **Save & Close**.

---

### 2.3 — Add a DHCP Range

1. Click into your newly created network.
2. Click **+ Add → Add Range**.
3. Configure:
   - Start address: `.100` of your network (e.g. `10.20.X.100`)
   - End address: `.200` (e.g. `10.20.X.200`)
   - Name: `lab-range-[yourname]`
4. On the **Member Assignment** tab, ensure Member 1 is assigned.
5. Save & Close.

---

### 2.4 — Configure DHCP Options

1. Click on your DHCP range.
2. Go to the **DHCP Options** tab.
3. Add the following options:
   | Option | Value |
   |--------|-------|
   | 3 (routers) | `10.20.<student number>.250` |
   | 6 (domain-name-servers) | `192.168.10.11` |
   | 15 (domain-name) | `workshop.lab` |
4. Save & Close.

> **Discussion:** What is option 43 used for? When would you use option 66/67?

---

### 2.5 — Create a Fixed Address (Reservation)

1. Still in your network, click **+ Add → Add Fixed Address**.
2. Configure:
   - IP: `10.20.<student number>.50`
   - MAC address: `00:50:56:AA:BB:CC` (simulated)
   - Name: `reserved-printer-[yourname]`
   - Comment: `Colour printer 3rd floor`
3. Save & Close.
4. Verify the fixed address appears in the network view with the 🔒 icon.

---

### 2.6 — IPAM Exploration

1. Go to **Data Management → IPAM**.
2. In the IP Map view, navigate to the `192.168.20.0/24` network.
3. Observe:
   - Black = in use / assigned
   - Purple = DNS object
   - White = available
   - Green = DHCP range
4. Hover over a used IP — a tooltip shows hostname, MAC, and lease info.
5. Click on a used IP for the full record detail.

> **Discussion:** How does IPAM help the operations team prevent IP conflicts? How does it help capacity planning?

---

### 2.7 — DHCP Lease Lookup

1. Go to **Data Management → DHCP → Leases**.
2. Browse the active lease list.
3. Search for a specific IP using the filter bar.
4. Click a lease to see:
   - Assigned IP and MAC address
   - Lease start / expiry time
   - Hostname sent by client
   - DHCP member that assigned the lease

---

### 2.8 — Troubleshooting Exercises

Work through these scenarios. Use the tools covered in the presentation.

---

**Scenario A — Missing A record**

> A user reports that `broken.workshop.lab` cannot be resolved.

1. Search for `broken.team-[yourname].workshop.lab` in the GUI — does it exist?
2. If not, create the record: IP `10.20.<student number>.150`.
3. Use **DNS → Diagnostic Tools → nslookup** from Member 1 to verify resolution.
4. What was the root cause?

---

**Scenario B — PTR record mismatch**

> The security team reports that `192.168.20.10<student number>` resolves in reverse to a different name than expected.

1. Navigate to the PTR record for `192.168.20.10<student number>` in the reverse zone.
2. What does it say? Does it match the forward A/Host record?
3. If there is a mismatch, fix it.

Testing can be done with dig or nslookup: `dig -x 192.168.20.101 @192.168.10.11`

---

**Scenario C — DHCP range exhaustion alert**

> You receive an alert that a DHCP range is at 85% utilization.

1. Navigate to the range in **Data Management → DHCP → Networks**.
2. Review the utilization.
3. What options do you have to resolve this? (Extend range, add exclusions, reduce lease time, add new range)
4. Extend the range end address by 20 more IPs if space is available.

---

**Scenario D — Checking the Audit Log**

> Your manager asks: "Who deleted the record for `appserver2.workshop.lab` and when?"

1. Go to **Administration → Logs → Audit Log**.
2. Filter by:
   - Object type: DNS Record
   - Action: Delete
   - Time range: last 24 hours
3. Can you identify the user and timestamp?
4. Discuss: why is the audit log important for compliance?

---

### 2.9 — Windows Integration Check (if Windows DC is available)

1. On the Windows DC, open **DNS Manager**.
2. Verify that `workshop.lab` zone shows correct NS records pointing to Infoblox members.

---

## Summary Checklist

By the end of both labs you should be able to:

- [ ] Log into the Infoblox GUI and navigate the Dashboard
- [ ] Identify Grid Master and Members and their roles
- [ ] Create a forward DNS zone and authoritative records
- [ ] Understand the difference between Host record, A record, CNAME, and PTR
- [ ] Search for any object using the global search and wildcards
- [ ] Explain DNS Views and how split-DNS works
- [ ] Create and apply Named ACLs
- [ ] Create a DHCP network, range, and fixed address
- [ ] Configure standard DHCP options
- [ ] Use the IPAM map for IP visibility
- [ ] Look up DHCP leases and trace IP-to-MAC-to-hostname
- [ ] Use Diagnostic Tools (dig/nslookup from GUI) to test DNS resolution
- [ ] Read the Audit Log to trace changes
- [ ] Explain the key considerations for Windows AD DNS integration

---

## Quick Reference: Useful CLI Commands (SSH to member)

```bash
# DNS diagnostics
show dns statistics
set log_local on           # enable debug logging temporarily

# DHCP diagnostics
show dhcp lease <ip-address>
show dhcp statistics

# System
show status
show version
show network
show log tail              # tail the syslog

# Restart a service (use with caution in production!)
restart dns
restart dhcp
```

## Quick Reference: dig Examples

```bash
# Forward lookup
dig @192.168.10.11 webserver1.workshop.lab

# Reverse lookup
dig @192.168.10.11 -x 192.168.20.101

# Query a specific record type
dig @192.168.10.11 workshop.lab MX
dig @192.168.10.11 workshop.lab NS

# Trace full resolution path
dig +trace www.google.com

# Short output
dig +short webserver1.workshop.lab @192.168.10.11
```

---

*Infoblox Workshop Lab Guide — Prepared for customer workshop | English | ~4 hours*
