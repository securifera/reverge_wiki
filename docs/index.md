Reverge is a red team focused, attack surface management framework




## Installation

Goto Amazon Marketplace and deploy a reverge instance


**When configuring the security groups for the EC2 instance consider the following:**

* Reverge web interface is running on port **80**
* TLS certificate issuance using Letsencrypt requires port **443** open to the internet
* SSH port forwarding for a remote database is running on port **2222**

<br>
<img src="../../assets/ec2_sec_group.png" alt="EC2 Security Groups" width="750" style="box-shadow: 5px 5px 15px rgba(0, 0, 0, 0.3);">
<br>
