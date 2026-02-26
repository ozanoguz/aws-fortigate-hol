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

Log in to the AWS Management Console using the provided link. 

Username: studentxx
Password: Provided during the session

[Access AWS Management Console](https://174296440058.signin.aws.amazon.com/console)

### 1.2 Create SSH Key Pair

You will need an SSH key pair to securely access the EC2 instances (Spoke VMs).

1. Ensure you are in the **Cape Town region (af-south-1)**.

<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/selectregion.jpg width="300"/>

2. In the AWS Console, search for "key pairs" and select it.

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

2.  Click "View purchase options"

<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/amipurchaseoptions.jpg width="600"/>

3.  Click "Subscribe" at the bottom of the page.

<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/amisubscribe.jpg width="600"/>

### 2.2 Deploying Lab Environment
1.  Log in to the **AWS Management Console**.
2.  Click the **Launch Stack** button (1-Button Deployment). This redirects you to AWS CloudFormation.

| **Description** | **1-Button Deployment** |
|-----------------|--------------------------|
| **FortiGate-VM Hub & Spoke Lab** | [![Launch Stack](https://github.com/40net-cloud/fortinet-aws-solutions/blob/master/FortiGate/Active-Passive-Multi-Zone/images/aws_cft_image.png)](https://console.aws.amazon.com/cloudformation/home?region=af-south-1#/stacks/create/review?templateURL=https://ftnt-cfts.s3.amazonaws.com/training/Cape_Town_HoL_CFT.yaml&stackName=FortiGate-Hub-and-Spoke-Lab) |

3.  AWS Cape Town region (af-south-1) is preselected.
4.  **Parameters to Configure:**
    * **Stack Name:** Enter your student-ID `Student01`
    * **KeyPair:** Select your existing SSH key created in **Section 1.2** above.
    * **ClientIP:** Enter your local public IP (e.g., `x.x.x.x/32`) to whitelist your access. You can use whatismyip.com or ip.me to find out the local public IP.
    * **LicenseType:** Select `BYOL` (Bring-Your-Own-License). Leave it as it is. 
<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/cft1.jpg width="400"/>
<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/cft2.jpg width="400"/>
<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/cft3.jpg width="400"/>
<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/cft4.jpg width="500"/>

5.  Click **Next** through the screens, then **Create Stack**.

### 2.3 Accessing the Resources
Wait until the stack status shows **CREATE_COMPLETE**. Navigate to the **Outputs** tab for credentials:

<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/cftcomplete.jpg width="500"/>

* 🌐 **FGTURL:** The management link for the FortiGate GUI.
* 👤 **Username:** `admin`
* 🔑 **Password:** Your **Instance ID** (e.g., `i-0a1b2c3d4e5f6g7h8`).

---

## ⚙️ Section 3: FortiGate Preparation

### 3.1 Initial Login
1.  Open the **FGTURL** in your browser.
2.  Log in using the Instance ID as the first password. Set a new password when prompted.

<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/fgtprompt.jpg width="300"/>

3.  Complete the setup wizard.

### 3.2 License Activation
1.  Use the FortiFlex token (shared during the session) to activate the license via the GUI.
2.  You can use the FortiGate web GUI to activate the license.

<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/flextoken.jpg width="500"/>

3.  Confirm reboot when prompted (takes 2–3 minutes).

<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/fgtreboot.jpg width="400"/>

4.  Verify that the FortiGate-VM shows a valid serial number and licensed status.

<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/fgtlicensevalid.jpg width="300"/>

### 3.3 Interface & Policy Verification

Verify that the automation has mapped the interfaces correctly:

* **Port 1 (WAN):** Connected to the Public Subnet (Gateway for Internet).
* **Port 2 (LAN):** Connected to the Private Subnet (Gateway for TGW traffic).

<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/fgtinterface.jpg width="600"/>

Confirm that the egress policy for Ubuntu VMs is preconfigured for Internet access.

<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/fgtexistingpolicy.jpg width="700"/>

---

## 🧪 Section 4: FortiGate SDN Connector

### 4.1 What is FortiGate SDN connector?

**SDN Connector**: The FortiGate SDN connector automatically integrates with cloud platforms to pull dynamic attributes such as IP addresses, tags, or security groups. This ensures firewall policies stay up to date with real-time changes in the cloud environment, reducing manual effort and enabling adaptive security.

### 4.2 Configure FortiGate SDN connector for AWS

1.  Navigate to "External Connectors". Choose "AWS".

<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/fgtsdn1.jpg width="500"/>

2.  Configure AWS SDN connector as shown below. "Use metadata IAM" and "Alternative resources" should be enabled.

<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/fgtsdn2.jpg width="300"/>

3.  Right click on SDN connector and choose "View Connector Objects"

<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/fgtsdn3.jpg width="300"/>

4.  Verify that the SDN connector pulled related AWS objects such as InstanceID, tag key, subnet ID, VPC ID etc.

<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/fgtsdn4.jpg width="600"/>

## 🧪 Section 5: Traffic Inspection Lab

### 4.1 Testing Spoke to Internet Connectivity (South to North - Egress)
Each Spoke VPC has an Ubuntu "Web Server."
1.  Find the Public IP of **Spoke1-VM** in the EC2 Console.

<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/spokevm1eip.jpg width="600"/>

2.  SSH into the instance using the SSH key pair. Username is `ubuntu`
3.  Test internet access via the FortiGate:
    ```bash
    ping 8.8.8.8
    telnet www.fortinet.com 443
    ```
    *Note: The instances use a "wait-for-FortiGate" script, so they only finish their setup once they successfully reach the Internet through the FortiGate.*

<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/egresstest.jpg width="500"/>

4.  On the FortiGate GUI, go to **Log & Report** > **Forward Traffic**.
5.  Observe the traffic coming from source IPs `10.1.x.x` and `10.2.x.x`.
6.  Verify that the **Egress-Internet-Access** policy is the one processing the traffic.

<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/egresslog.jpg width="700"/>

### 4.2 Testing Spoke to Spoke Connectivity (East/West)

1.  Create a firewall policy to enable East/West inspection.

<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/eastwest1.jpg width="600"/>
<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/eastwest2.jpg width="600"/>

2.  Initiate ICMP and HTTP traffic from Spoke-1 (10.1.0.100) to Spoke-2 (10.2.0.100)

<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/eastwest3.jpg width="400"/>

3.  Verify traffic logs by checking the FortiGate GUI

<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/eastwest4.jpg width="700"/>

### 4.2 Testing Internet to Spoke Connectivity (North to South - Ingress)

1.  Create VIP objects for Spoke1 and Spoke2. Spoke1 HTTP will be exposed to Internet using TCP/8081 port, Spoke2 HTTP port will be exposed using TCP/8082 port.

<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/vip1.jpg width="700"/>
<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/vip2.jpg width="700"/>

2.  Create a firewall policy to enable Ingress inspection.

<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/ingresspolicy1.jpg width="600"/>
<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/ingresspolicy2.jpg width="600"/>

3.  Initiate the traffic using your web browser

<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/ingresstraffic1.jpg width="700"/>
<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/ingresstraffic2.jpg width="700"/>


4.  Verify traffic logs by checking the FortiGate GUI.

<img src=https://github.com/ozanoguz/aws-fortigate-hol/blob/main/images/ingresslogs.jpg width="700"/>

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
