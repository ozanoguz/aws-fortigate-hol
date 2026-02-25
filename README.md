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

## 🚀 Section 1: Lab Preparation

### 1.1 Access AWS Console

Log in to the AWS console using the link below. You can use k8s_studentxx as username, password will be provided during the session.

[Access AWS Management Console](https://174296440058.signin.aws.amazon.com/console)

### 1.2 Create SSH Key Pair

You must create an SSH key pair to securely access the EC2 instances (Spoke VMs) deployed in this lab.

1. After logging in to the AWS Management Console**, ensure you are in the **Cape Town region (af-south-1)**.

<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/selectregion.jpg width="300"/>

2. Search for "key pairs" in the top search box. Click "Key pairs" under search results.

<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/selectkeypairs.jpg width="400"/>

3. Configure the key pair:
   * **Name:** `Student01-key` (or use your assigned student ID)
   * **Key pair type:** RSA
   * **Private key file format:** `.pem`

<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/createkeypair.jpg width="500"/>
   
4. Click **Create key pair**.
5. The `.pem` file will download automatically. **Store it securely**—this file cannot be downloaded again.

> 🔐 **Important:** You will select this key pair later during the CloudFormation deployment.  
> 🖥️ **Windows users:** You may need PuTTY or Windows OpenSSH to use the `.pem` file.  
> 🐧 **macOS/Linux users:** Restrict permissions before use:
> ```bash
> chmod 400 Student01-key.pem
> ```

## 🚀 Section 2: Deployment & Provisioning

### 2.1 Subscribe to FortiGate BYOL AMI image

1.  Subscribe FortiGate BYOL AMI image using the AWS marketplace. Click the link below:

[FortiGate BYOL Marketplace Listing](https://aws.amazon.com/marketplace/pp/prodview-lvfwuztjwe5b2?applicationId=AWSMPContessa&ref_=beagle&sr=0-1)

2.  Click "View purchase options" to continue with the subscription.

<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/amipurchaseoptions.jpg width="600"/>

3.  Click "Subscribe" button at the bottom of the next page.

<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/amisubscribe.jpg width="600"/>

### 2.2 Deploying Lab Environment
1.  Log in to your **AWS Management Console** first.
2.  Click to **Launch Stack** button below. This will redirect you to the AWS CloudFormation page.

| **Description** | **1-Button Deployment** |
|-----------------|--------------------------|
| **FortiGate-VM Hub & Spoke Lab** | [![Launch Stack](https://github.com/40net-cloud/fortinet-aws-solutions/blob/master/FortiGate/Active-Passive-Multi-Zone/images/aws_cft_image.png)](https://console.aws.amazon.com/cloudformation/home?region=af-south-1#/stacks/create/review?templateURL=https://ftnt-cfts.s3.amazonaws.com/training/Cape_Town_HoL_CFT.yaml&stackName=FortiGate-Hub-and-Spoke-Lab) |

3.  AWS Cape Town region (af-south-1) is automatically selected for you.
4.  **Parameters to Configure:**
    * **Stack Name:** Enter your student-ID `Student01`
    * **KeyPair:** Select your existing SSH key created in **Section 1.2** above.
    * **ClientIP:** Enter your local public IP (e.g., `x.x.x.x/32`) to whitelist your access. You can use whatismyip.com or ip.me to find out the local public IP.
    * **LicenseType:** Select `BYOL` (Bring-Your-Own-License). Leave it as it is. 
<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/cft1.jpg width="400"/>
<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/cft2.jpg width="400"/>
<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/cft3.jpg width="400"/>
<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/cft4.jpg width="500"/>

5.  Click **Next** through the screens, and then **Create Stack**.

### 2.3 Accessing the Resources
Wait for the status to show **CREATE_COMPLETE**. Navigate to the **Outputs** tab to find your credentials:

<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/cftcomplete.jpg width="500"/>

* 🌐 **FGTURL:** The management link for the FortiGate GUI.
* 👤 **Username:** `admin`
* 🔑 **Password:** Your **Instance ID** (e.g., `i-0a1b2c3d4e5f6g7h8`).

---

## ⚙️ Section 3: FortiGate Configuration

### 3.1 Initial Login
1.  Open the **FGTURL** in your browser.
2.  Log in using the Instance ID. You will be prompted to set a new password.

<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/fgtprompt.jpg width="400"/>

3.  Complete the setup wizard.

### 3.2 Activating FortiGate license
1.  We will use the FortiFlex token for license activation. The token code will be shared with you during the session.
2.  Establish SSH connection with FortiGate-VM using its assigned public IP. 



### 3.3 Interface Verification
Verify that the automation has mapped the interfaces correctly:
* **Port 1 (WAN):** Connected to the Public Subnet (Gateway for Internet).
* **Port 2 (LAN):** Connected to the Private Subnet (Gateway for TGW traffic).

<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/fgtinterface.jpg width="600"/>

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
