---
title:  "Shaman's Guide to REST API Hacking: Part 1 Intrduction and Discovery"
layout: post
excerpt:  "These blog series will first go through key concepts about REST APIs, their use cases and advantages, then will go into details of how to conduct security testing of them, especially the methodology our team follows during real engagements.
"
---

These blog series will first go through key concepts about REST APIs, their use cases and advantages, then will go into details of how to conduct security testing of them, especially the methodology our team follows during real engagements.

# What are APIs

APIs (Application Programming Interfaces) are mechanisms which allow two applications to communicate and share data between each other without concerning about their internal details. These applications usually have different code bases and even don't need to use the same programming language. This feature of APIs makes them so common, because it eliminates the need for dealing with complex code. APIs are widely-used in modern applications and essentially serve the purpose of transferring data from client side to server side in background.

# Introduction to REST API

Different types of APIs were introduced through time, most of which are out of scope of this article. We're going to cover only REST API and their security assessments throughout this guide. REST API allows client side to communicate with server via HTTP protocol to perform different actions. There are defined HTTP methods (GET, POST, PUT, DELETE , etc.) in REST standard to perform required options. The major feature of REST API is being stateless, which means that each request can be made independently and a server doesn't need to store client session or any client specific data. A client interacts with a REST API through sending HTTP requests to its endpoints which contain URLs of services. Most of the API specific attacks are performed at this point - API calls made by clients.

# Discovering API endpoints

First off you need to obtain as many API endpoints of a target application/API server as possible. If an API documentation of the target is available, it makes discovery process even easier, you can quickly refer to it and get detailed information (description, HTTP request method, parameters, so on)  about each endpoint. You will probably come across Swagger UI, which comes so handy while testing APIs. It describes all the required details about API and helps a tester to better visualize API design and to try out making API calls in a browser. 
Even if API documentation is available, you shouldn't stop there and continue discovery process via using specific wordlists in most cases. What you need to do in this stage is basically brute-forcing API endpoints with wordlists. There are really good tools which you can make use of in this stage such as gobuster, ffuf, kiterunner (highly recommended, specifically designed to be used with APIs), etc.
You can also take advantage of the following OSINT techniques (Passive Information Gathering) to discover further API endpoints:
- Google Dorking<br />
	This technique relies on searching for specific keywords using advanced search operators. There are dozens of useful Google dorks which you can get familiar with through [this link](https://www.exploit-db.com/google-hacking-database). As an example, you can use the following Google dork to find interesting API directories for further testing:
  `inurl:"/api/v1" intext:"index of /"`
- WayBackMachine<br />
	WayBackMachine archives websites across the web and allows users to perform searches through them. It actively indexes web pages and stores a copy of them with the time when it indexed in its database. You can view older versions of your target application, which allows you to look at source code of old web pages and extract new API endpoints to test. You can access it with its [website](https://archive.org) or use waybackurls tool to automate the process.
- Github Dorking<br />
	Another discovery method is looking for interesting endpoints and leaked credentials through Github's search feature. Similar to Google Dorking, you need to use specific search operators and keywords here as well. Hundred lines of code is pushed to Github every day by developers, so there is a chance that you will get some interesting findings by querying appropriate searches.  There are good resources out there about how to use power of Github search as a security tester.
It needs to be noted that you shouldn't limit discovery process to these techniques, you can
always get creative and look for new methods of gathering useful information about your target. 

# Conclusion
In this first part of our API Security blog series, we've covered introduction to REST APIs and how to discover API endpoints. Discovery phase is also known as Recon which is an essential part of any security assessment since the more you know about your target the more security flaws you will identify.

If you are interested in assessing security of your APIs, reach out to us using [this link](https://shamanredteam.com/#/contact). We will organize an initial meeting with you to give more information about workflow of our projects and discuss a possible cooperation. 

References :<br />
https://aws.amazon.com/what-is/api/<br />
https://blog.hubspot.com/website/what-is-swagger<br />
https://github.com/assetnote/kiterunner <br/>
https://github.com/ffuf/ffuf <br />
https://github.com/OJ/gobuster <br />
https://archive.org<br />
https://www.exploit-db.com/google-hacking-database<br />
