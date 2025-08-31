+++
title = 'Burp Suite Tricks Part 1'
date = 2023-11-02
draft = false
description = 'Collection of Burp Suite Tips and Tricks'
tags = ["Burp"]
+++

## Intro

BurpSuite is undoubtedly one of the most widely used tools for web security and bug bounty hunting. In this guide, I will introduce you to the fundamentals of how the tool works and how it can be used effectively. Additionally, I will provide insights on how to leverage BurpSuite for more advanced applications.

## Miscellaneous

### SSL/TLS Pass Through

Whenever I open a new Burp Project, I always come across these unnecessary URLs, and every time I see them, it is annoying ngl :(

![Unnecessary URLs in Burp](/posts/burp-suite-tricks1/image15.png)

Now, what I've seen people generally do is go to the HTTP History and tick on the 'Show only in-scope items,' and all the requests will just disappear :P

![Show only in-scope items](/posts/burp-suite-tricks1/image3.png)

But when we disable the option, all the proxied requests are still there, right?

![Proxied requests still visible](/posts/burp-suite-tricks1/image1.png)

This is something you don't want because:

1. Everything is stored in the Burp Project File, so if you are on a pentest and the application is using OAuth for login, most likely it could have captured your corporate/personal credentials.
2. This is also the reason why your Burp State is very large. Because it has all these unnecessary requests.

Now, to avoid all these unnecessary requests, you can add the following regexes in SSL Pass-through.

![SSL Pass-through settings](/posts/burp-suite-tricks1/image13.png)

But again, you don't want to add these regexes manually again and again.

So, in order to deal with this, just download the following Burp Configuration file:

```bash
wget https://gist.github.com/n0sandb0x/55edcdee0a6c7b4d566e526de48a6f62
```

Now whenever you are starting a new project under the configuration settings select the JSON file.

![Configuration settings](/posts/burp-suite-tricks1/image11.png)

and all the regexes will be automatically added in SSL/TLS Pass-through.

![Regexes added automatically](/posts/burp-suite-tricks1/image10.png)

### Hidden Fields in the Application

Now, whenever you are testing an application, there are often scenarios where the application might be using Hidden Input fields.

![Hidden input fields](/posts/burp-suite-tricks1/image14.png)

So now to see these fields and additional parameters, you can go to the proxy section, and under Response Modification Rules, you have to check these items:

![Response modification rules](/posts/burp-suite-tricks1/image12.png)

Now Burp will highlight these hidden form fields in the browser itself

![Highlighted hidden fields](/posts/burp-suite-tricks1/image4.png)

### Using Hackvector for Decoding, Scanning and Fuzzing

Suppose one day you are testing an application and you come across a scenario where the application takes base64 encoded value as a user input as shown below.

Here I have modified the SQL lab [https://github.com/h1pmnh/sqli-dojo-docker](https://github.com/h1pmnh/sqli-dojo-docker) created by [pmnh](https://x.com/pmnh_). This takes the base64 value as the correct user input.

![Base64 input example](/posts/burp-suite-tricks1/image2.png)

Now, the first thing people do here is they will copy the user input and paste it into the decoder to see what the correct value is

![Decoder usage](/posts/burp-suite-tricks1/image5.png)

Now to check for other values, they have to encode the payload again, and you have to go back and forth again and again and this is very time-consuming.

This is where HackVector comes in - it uses XML-like tags to specify the encoding or conversion you want to use.

Now first, since we know the application is using base64 as a valid input, we have to use tags like `<@base64>1<@/base64>` and the Hackvector will automatically encode the payload in the backend and send it to the server.

![Hackvector usage](/posts/burp-suite-tricks1/image6.png)

Now we can also utilize this feature to convert all the payloads which are used by the Burp Scanner into base64 encoded values.

First, send the request to Burp Intruder, select the decoded insertion point, and click on "Scan defined insertion points".

![Burp intruder setup](/posts/burp-suite-tricks1/image9.png)

Now notice in Burp Suite logger all the payloads are automatically getting base64 encoded.

![Base64 encoded payloads](/posts/burp-suite-tricks1/image16.png)

If you have a custom wordlist and want to use the same for Burp Intruder simply add the SQL Injection payload and start the attack.

![Custom wordlist usage](/posts/burp-suite-tricks1/image8.png)

Notice that in Logger, all the SQL payloads which we sent using Burp Intruder are automatically encoded with base64.

![Encoded SQL payloads](/posts/burp-suite-tricks1/image7.png)


This is not only limited to the intruder - you can also utilize the power of Burp Suite scanner as well.
Just put the payload inside those tags <@base64>your_payload_here<@/base64> and start the Scan on Defined Insertion Point and you're good to go :)

## Wrapping Up

So these were some of the Burp Suite tricks that I use daily to make my testing more efficient. Quick recap:
- SSL Pass Through to keep your Burp project clean and secure
- Finding hidden form fields right in your browser
- Using HackVector to handle encoded inputs like a pro

There's a lot more to Burp Suite, and I'll be sharing more cool tricks in Part 2. If you found this helpful or have some tricks of your own, feel free to reach out!

Stay tuned for more hacks and happy hunting! 
