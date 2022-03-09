---
layout: post
title:  "MFA Bypass"
date:   2019-12-19 09:29:20 +0700
categories: bugbounty
---

<img src="https://cdn-images-1.medium.com/max/1600/1*zu7qmgwwQ_SctNTJ0B6opA.jpeg">
<p>
Hello friends, this was my first writeup so please ignore my grammatical mistakes. Today I wanted to share you the way how I have bypassed MFA. Please read the bottom of the writeup where I have described what helped me to find out this bug. Let's get started
<p>
Before starting how I found the vulnerability, I'll give you some information regarding the target. I was invited to a private program so let say it as redacted.com. It had the functionality of maintaining all payment-related activities. So I started to test the application. I just started to sign-up for an account. While signing-up there was a page where you need to verify your phone number, We can't skip that verification. It was something like the below image.
<p>
<img src="https://cdn-images-1.medium.com/max/1600/1*ZQSemrgJfn3nXBApZw498Q.png">
<p>
I entered my phone number and click on send code. And they sent an OTP to my number. I'm thinking of a way to bypass this authentication. I entered the wrong OTP in the OTP verification field and captured the request and the request of the POST data was something like this.{"id":"mph01SQXWUPMXTKTHzcj","token":"123456"}
<p>
I received an error message to enter the Correct-OTP. One thing got in my mind to check the response of the request. So I captured the response of the request and it was like the below image
<p>
<img src="https://cdn-images-1.medium.com/max/1600/1*_p6FP_ZGH7w_ks-OXoHWgA.png">
<p>
I think you got it. Yah I modified the response.
<p>
And successfully I bypassed the MFA. I was like whooola..!
<p>
Special thanks to Udit Pratap Singh Bhadauria(Udit_thakkur)who motivated me to test the application because I wondered that even if I have found out a bug it will be anyhow going to duplicate as a large number of bugs have been reported already. Because of his motivation, I have found out this bug.
<p>
<b>Point to remember</b>: Dig deeper and deeper even a lot of bugs have been reported already, there will be something for you to find out.
<p>
<b>Bug Reported</b> : 31 Oct 2019 morning and they triaged by 31 Oct 2019 afternoon
<hr>
