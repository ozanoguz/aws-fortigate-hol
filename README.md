# 🛡️ FortiGate-VM Hub & Spoke Lab: AWS Transit Gateway

**⚠️ DISCLAIMER: This environment is prepared specifically for hands-on workshop purposes. Do not use for production without additional hardening.**

This lab guide describes how to protect distributed cloud workloads using a **Centralized Security Hub** architecture. You will deploy a **FortiGate-VM** to inspect North/South (Internet) and East/West traffic for multiple Spoke VPCs using an **AWS Transit Gateway (TGW)**.

---

## 🏗️ Lab Architecture
The CloudFormation Template (CFT) automates a comprehensive Hub & Spoke environment:
* **Central Security Hub:** A VPC containing your FortiGate-VM (Inspection point).
* **Transit Gateway (TGW):** The "Cloud Router" connecting all VPCs.
* **Workload Spokes:** Two separate VPCs (Spoke 1 & Spoke 2) running Ubuntu web servers.
* **Traffic Flow:** All egress and east/west traffic from Spoke instances is routed through the TGW to the FortiGate's private interface for security filtering.



---

## 🚀 Section 1: Login AWS Console


## 🚀 Section 2: Deployment & Provisioning

### 2.1 Launch the CloudFormation Stack
1.  Log in to your **AWS Management Console**.
2.  Navigate to **CloudFormation** > **Create stack** > **With new resources**.
3.  Upload the file: `Cape_Town_HoL_CFT v2.yaml`.
4.  **Parameters to Configure:**
    * **Stack Name:** Enter your student-ID `Student01`
    * **KeyPair:** Select your existing SSH key.
    * **ClientIP:** Enter your local public IP (e.g., `x.x.x.x/32`) to whitelist your access. Use whatismyip.com or ip.me to find out.
    * **LicenseType:** Select `PAYG` (Pay-As-You-Go).
5.  Click **Next** through the screens and then **Submit**.

### 2.2 Accessing the Resources
Wait for the status to show **CREATE_COMPLETE**. Navigate to the **Outputs** tab to find your credentials:
* 🌐 **FGTURL:** The management link for the FortiGate GUI.
* 👤 **Username:** `admin`
* 🔑 **Password:** Your **Instance ID** (e.g., `i-0a1b2c3d4e5f6g7h8`).

---

## ⚙️ Section 3: FortiGate Configuration

### 3.1 Initial Login
1.  Open the **FGTURL** in your browser.
2.  Log in using the Instance ID. You will be prompted to set a new password.
3.  Complete the setup wizard.

### 3.2 Interface Verification
Verify that the automation has mapped the interfaces correctly:
* **Port 1 (WAN):** Connected to the Public Subnet (Gateway for Internet).
* **Port 2 (LAN):** Connected to the Private Subnet (Gateway for TGW traffic).

---

## 🧪 Section 4: Traffic Inspection Lab

### 4.1 Testing Spoke Connectivity
Each Spoke VPC has an Ubuntu "Web Server."
1.  Find the Public IP of **Spoke1-VM** in the EC2 Console.
2.  SSH into the instance using the SSH key pair. Username is `ubuntu`
3.  Test internet access via the FortiGate:
    ```bash
    curl www.fortinet.com
    ```
    *Note: The instances use a "wait-for-FortiGate" script, so they only finish their setup once they successfully reach the Internet through the FortiGate.*

### 4.2 Real-time Log Monitoring
1.  On the FortiGate GUI, go to **Log & Report** > **Forward Traffic**.
2.  Observe the traffic coming from source IPs `10.1.x.x` and `10.2.x.x`.
3.  Verify that the **Egress-Internet-Access** policy is the one processing the traffic.

---

## 🔐 Section 5: Security Policy Automation

### 5.1 Blocking Malicious Categories
1.  Navigate to **Policy & Objects** > **Firewall Policy**.
2.  Edit the **Egress-Internet-Access** policy.
3.  Enable **Web Filter** and select the `default` profile.
4.  From the Spoke VM, try to access a restricted site. You should see the FortiGate replacement message.

### 5.2 Traffic Flow Analysis
Because of the **Transit Gateway Route Tables**, the FortiGate sees the original IP of the Spoke instance, allowing for granular, identity-based security policies.

---

## 🧹 Section 6: Resource Cleanup

To prevent unnecessary AWS costs, please delete the environment when finished:
1.  Go to the **CloudFormation** console.
2.  Select your stack and click **Delete**.
3.  Ensure the status changes to `DELETE_COMPLETE`.
