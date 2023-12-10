---
title:  "Shaman's Guide to Web Cache Deception Attacks"
layout: post
excerpt:  "In this blog we’ll dive into Web Cache Deception attack which has gained popularity in recent years. It has been found that the websites of many large companies was vulnerable to it and it is likely that vulnerable instances may still be discovered today as well."
---

In this blog we’ll dive into Web Cache Deception attack which has gained popularity in recent years. It has been found that the websites of many large companies was vulnerable to it and it is likely that vulnerable instances may still be discovered today as well. This attack was firstly documented in 2017 by Omer Gil with some example attack scenarios and actual findings. Throughout this guide, first some concepts related to web caching will be discussed, then the main focus will be how the attack works and exploitation of a target which was found to be vulnerable to Web Cache Deception in wild.

### Introduction
Before going into details of Web Cache Deception attacks, let's take a quick look at the purpose of web caches and how they work. These days websites are likely to have some caching mechanisms in place for reducing time required to load static files. Basically, this mechanism sits between a user and a web server and stores the files which are requested frequently from web server. Most of the time, these mechanisms are Content Delivery Networks (CDN), which distribute content of web resources across the world through caching and serving it on the closest server to users. This approach is highly effective because it reduces the amount of traffic to the origin web server and allows users to get better website performance via improving loading speed.

This behaviour of websites is not necessarily security issue, since it is designed to store and deliver only static files (style sheets, Javascript files, images, etc.) which don't include any user-specific sensitive information. The problem arises when there is a security misconfiguration which leads to path confusion between a web server and a caching mechanism (CDN). Path confusion simply means that web server and proxy (in this case CDN) interprets a requested URL differently which causes a conflict between these servers in determining whether the response of a requested URL can be cached or not. This conflict can be exploited to deceive CDN caching mechanism into storing dynamic responses which may potentially contain sensitive information.

### Preconditions
First of all, let's take a look at what the preconditions for the Web Cache Deception attacks are.

Obviously, a target website should have a caching mechanism, which can be detected due to specific headers, such as  Cache-Control and X-Cache which is a strong indication of caching. When coming across `X-Cache: hit` in response headers it denotes that the response is retrieved from a cache, while `X-Cache: miss` means that the response hasn't been cached yet and is retrieved from the origin server.

Even if there isn't any header related to caching in a response, it is still recommended to check if a website caches responses manually. It can be done via requesting the same URL on two separate browsers subsequently. If the second browser returns the response which is identical to the first one, it means that caching is in place.

After confirming caching, the next next step is testing for path confusion. This can be detected via appending arbitrary static files to the end of existing paths of a web application and observing the behaviour of it. Let's assume that the target application has a valid path `https://example.com/account.php` which displays information about the current user. To identify if there is a path confusion issue, the request similar to the following can be requested:

`https://example.com/account.php/nonexistent.jpg`

Note that, the requested resource doesn't exist on the server and is only used to detect any misconfiguration related to paths. If the response to the above request displays the content of account.php and CDN caches the request, all the conditions are met for Web Cache Deception attack. Here, CDN determines the cacheability of response solely based on the file extension (.css), however securely-configured caching server should consider content-type and caching headers returned from the origin server too.

### Attack scenario
After confirming necessary conditions for the attack, the target application should be analyzed to identify interesting endpoints which return user-specific details, such as API tokens, session cookies, CSRF tokens, etc. 

In our last example, it was discovered that CDN server caches the response when sending a request to the endpoint /account.php/nonexistent.jpg. An attacker can send full URL of this endpoint to a victim to obtain their account details. This attack flow is illustrated as follows:
1. The victim visits the URL sent by the attacker. Since there is no cache for this resource, CDN server (Web Cache) forwards the request to the web server
2. CDN server ignores any caching headers returned from the web server and caches the response as the URL ends with the static file extension (.jpg)
3. After the victim's account page is cached, the attacker requests the same URL and gets the victim's details in the response.
![diagram](https://github.com/blog-shamanredteam/blog-shamanredteam.github.io/assets/147247315/d579b6fd-d66a-4234-a4bd-91477705e890)

Source: Cached and Confused: Web Cache Deception in the Wild, Aug 2020, USENIX Security Symposium
.com/what-is/api/)
### Case Study
In one of our recent engagements, we discovered that a target application is vulnerable to a Web Cache Deception attack. It was a e-commerce website and used `/getinfo` endpoint to fetch account details from the backend. The web server was configured in such a way that it returned the same response when appending any static file extension to the end of endpoints. For example, it was possible to get account details by requesting `/getinfo.css` path. After that, we checked if the response got cached via requesting the same URL on another browser and observed that the same response was returned. Since both of the mentioned conditions were met for Web Cache Deception attacks, we concluded that the target was vulnerable.

In order to demonstrate the impact of the attack, we used a CSRF token returned in the `/getinfo` endpoint. This token could be used to perform CSRF attacks on victims to change their account details and settings using another endpoint `/my-account/change-personal-details`.

### Conclusion
In this blog we looked at key concepts about caching and covered Web Cache Deception attacks in detail. These attacks could be very dangerous, that's why it is essential to detect them in advance and mitigate security misconfigurations. The recommendations to prevent these attacks are as follows:
 - Make sure that caching are based on content type and headers returned by origin server
 - Configure web server in such a way that it returns Not Found error when requesting static resources which don't exist and rejects requests with unexpected path components.

### References
 - [http://omergil.blogspot.com/2017/02/web-cache-deception-attack.html](http://omergil.blogspot.com/2017/02/web-cache-deception-attack.html)
 - [https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management_Testing/13-Test_for_Path_Confusion](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management_Testing/13-Test_for_Path_Confusion)
 - [https://www.researchgate.net/publication/350568189_Cached_and_Confused_Web_Cache_Deception_in_the_Wild](https://www.researchgate.net/publication/350568189_Cached_and_Confused_Web_Cache_Deception_in_the_Wild)
