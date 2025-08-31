+++
title = 'OTP Authentication Bypass leads to Account Takeover on Mobile App'
date = 2021-09-28
draft = false
description = 'Authentication Bypass'
tags = ["Mobile", "API"]
+++

Hi folks! Recently while testing a mobile application, I found an interesting OTP Authentication Bypass vulnerability that could lead to account takeover. It was super simple and required just a small parameter modification in the API request.

The authentication flow starts with a phone number input page:

![Phone Number Input Page](/posts/otp-bypass/phone_page1.png)

After entering the phone number, you get redirected to the OTP verification page:

![OTP Verification Page](/posts/otp-bypass/otp_page1.png)

I started testing the authentication flow and intercepted the traffic using Burp Suite:

![Intercepted Request](/posts/otp-bypass/request.png)

While analyzing the traffic, I noticed that the OTP verification process was handling two parameters: `otp` and `confirm-OTP`. The application followed a standard two-step verification where users enter their phone number and verify it using an OTP.

Here's where it gets interesting - when sending the OTP verification request, I noticed something odd in the request:

```http
POST /auth/realms/[redacted]/login-actions/authenticate HTTP/1.1
Content-Type: application/x-www-form-urlencoded

otp=1234&confirm-OTP=
```

By simply removing the `otp` parameter and only sending the `confirm-OTP` parameter with any value, the server would accept the request:

```http
POST /auth/realms/[redacted]/login-actions/authenticate HTTP/1.1
Content-Type: application/x-www-form-urlencoded

confirm-OTP=1234
```

And boom! The application accepted this modified request and granted access to the account:

![Successful Login](/posts/otp-bypass/account.png)

This simple modification was enough to bypass the entire phone number verification process.

The impact? Pretty severe. An attacker could:
- Bypass phone verification completely
- Take over any user account
- Automate the attack to compromise multiple accounts

For developers looking to prevent such issues, here are some quick recommendations:
- Always validate both OTP parameters server-side
- Implement proper rate limiting
- Add additional security layers like device binding and temporary lockouts
- Never trust client-side validation alone

Remember folks, even a small oversight in parameter validation can lead to complete authentication bypass. Always thoroughly test your authentication mechanisms!

After reporting this vulnerability, I was rewarded with a bounty of $955 for this finding:

![Bug Bounty Reward](/posts/otp-bypass/rewarded.png)

Stay secure! 🔐


