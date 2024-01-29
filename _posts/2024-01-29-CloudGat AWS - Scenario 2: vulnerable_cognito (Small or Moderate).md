# CloudGoat Scenario 2: vulnerable_cognito (Small / Moderate): WalkThrough and Mitigation

Hey hi, Soumyanil is back but with Cloud now, let’s have a look into AWS Penetration Testing. I will be covering all the scenarios provided by the CloudGoat vulnerable AWS environment by Rhinosecurity Labs.

**CloudGoat** is **Rhino Security Labs**’ “_Vulnerable by Design_” AWS deployment tool. It allows us to hone our cloud cybersecurity skills by creating and completing several **CTF** style scenarios. Each and every scenario is composed of AWS resources arranged together to create a structured learning experience. These scenarios vary in difficulty and many offer multiple paths to Flag/Goal.

From ***Purple Team point of View***:\
i. Our mission to perform Recon, identify vulnerabilities, and perform exploitation in order to achieve the scenario objective.\
ii. Provide Mitigation for the above Flaw that we came accross.

### <ins>Details about the Scenario :</ins>
i. In this scenario, you are presented with a signup and login page with AWS Cognito in the backend.\
ii. You need to :\
&nbsp;&nbsp;a. ***Bypass restrictions*** \
&nbsp;&nbsp;b. and ***Exploit Misconfigurations in Amazon Cognito*** in order :\
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

So basically, **Amazon Cognito** is a User Identity and Data Syncronization Service, which gives User a very easy time to manage their accounts connected Over Mutiple Accounts. 
