reverge is a red team focused, attack surface management framework




## Installation

Goto Amazon Marketplace and deploy a reverge instance


**When configuring the security groups for the EC2 instance consider the following:**

* reverge web interface is running on port **80**
* TLS certificate issuance using Letsencrypt requires port **443** open to the internet
* SSH port forwarding for a remote database is running on port **2222**

<br>
<img src="../../assets/ec2_sec_group.png" alt="EC2 Security Groups" width="750" style="box-shadow: 5px 5px 15px rgba(0, 0, 0, 0.3);">
<br>

## Secrets Manager

reverge stores various sensitive third-party credentials used to interact with cloud services and other applications. By default, these credentials are encrypted with a key that is also stored in the database, offering minimal protection if the database is compromised. However, reverge offers the ability to utilize AWS Secrets Manager to store the encryption key separately, thereby providing an extra layer of security for the stored credentials.

After setting up a reverge AWS instance, it is strongly recommended to create an AWS IAM role accompanied by a suitable permission policy. This setup will enable the creation and retrieval of the reverge encryption key. Once the role is created, attach it to the reverge EC2 instance.


```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowCreateSecret",
            "Effect": "Allow",
            "Action": [
                "secretsmanager:CreateSecret"
            ],
            "Resource": "*"
        },
        {
            "Sid": "AllowManageOwnSecrets",
            "Effect": "Allow",
            "Action": [
                "secretsmanager:DescribeSecret",
                "secretsmanager:GetSecretValue",
                "secretsmanager:UpdateSecret",
                "secretsmanager:DeleteSecret",
                "secretsmanager:ListSecretVersionIds"
            ],
            "Resource": "arn:aws:secretsmanager:*:*:secret:reverge_key*"
        }
    ]
}

```