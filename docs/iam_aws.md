IAM Role Setup on AWS
=====================

We need to create two roles for security monkey. The first role will be an instance profile that we will launch security monkey into. The permissions on this role allow the monkey to use STS to assume to other roles as well as use SES to send email.

Creating SecurityMonkeyInstanceProfile Role
-------------------------------------------

Create a new role and name it “SecurityMonkeyInstanceProfile”:

![image]

Select “Amazon EC2” under “AWS Service Roles”.

![image][1]

Select “Custom Policy”:

![image][2]

Paste in this JSON with the name “SecurityMonkeyLaunchPerms”:

``` sourceCode
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ses:SendEmail"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": "sts:AssumeRole",
      "Resource": "arn:aws:iam::*:role/SecurityMonkey"
    }
  ]
}
```

Review and create your new role:

![image][3]

Creating SecurityMonkey Role
----------------------------

Create a new role and name it “SecurityMonkey”:

![image][4]

Select “Amazon EC2” under “AWS Service Roles”.

![image][1]

Select “Custom Policy”:

![image][2]

Paste in this JSON with the name “SecurityMonkeyReadOnly”:

``` sourceCode
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "acm:describecertificate",
                "acm:listcertificates",
                "cloudtrail:describetrails",
                "cloudtrail:gettrailstatus",
                "config:describeconfigrules",
                "config:describeconfigurationrecorders",
                "directconnect:describeconnections",
                "ec2:describeaddresses",
                "ec2:describedhcpoptions",
                "ec2:describeflowlogs",
                "ec2:describeimages",
                "ec2:describeinstances",
                "ec2:describeinternetgateways",
                "ec2:describekeypairs",
                "ec2:describenatgateways",
                "ec2:describenetworkacls",
                "ec2:describenetworkinterfaces",
                "ec2:describeregions",
                "ec2:describeroutetables",
                "ec2:describesecuritygroups",
                "ec2:describesnapshots",
                "ec2:describesubnets",
                "ec2:describetags",
                "ec2:describevolumes",
                "ec2:describevpcendpoints",
                "ec2:describevpcpeeringconnections",
                "ec2:describevpcs",
```

  [image]: images/resized_name_securitymonkeyinstanceprofile_role.png
  [1]: images/resized_create_role.png
  [2]: images/resized_role_policy.png
  [3]: images/resized_role_confirmation.png
  [4]: images/resized_name_securitymonkey_role.png