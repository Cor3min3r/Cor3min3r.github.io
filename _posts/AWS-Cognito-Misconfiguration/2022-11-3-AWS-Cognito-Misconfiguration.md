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
- https://aws.amazon.com/cognito/
- https://www.techtarget.com/searchaws/definition/Amazon-Cognito

The Target application was using AWS Cognito for authentication purpose. 
