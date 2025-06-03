---
title: Hacker101 - Cody's First Blog
author: first_wan
categories: [Cybersecurity, CTF]
tags: [SQL Injection, Hacker101, CTF]
pin: false
---
This CTF are provided by Hacker101 with moderate difficulty. It have 3 flags to find.

## Reconnaissance

It will redirect to a page with one comments box in the page. We can know that this web site was develop with PHP.

![ctf_cody_first_blog_image](/blogs/ctf_cody_first_blog/recon_1.png)

When we inspect the HTML element, we able to get the URL for admin login.

![ctf_cody_first_blog_image](/blogs/ctf_cody_first_blog/recon_2.png)

## Flag 0

I tried to inject JavaScript into the comment, but nothing happen. 

![ctf_cody_first_blog_image](/blogs/ctf_cody_first_blog/flag0_1.png)

It seem like this comment need to be approval from the owner.

![ctf_cody_first_blog_image](/blogs/ctf_cody_first_blog/flag0_2.png)

But how about PHP code?

![ctf_cody_first_blog_image](/blogs/ctf_cody_first_blog/flag0_3.png)

It also require approval from owner, but we get the flag by injecting PHP code.

![ctf_cody_first_blog_image](/blogs/ctf_cody_first_blog/flag0_4.png)

## Flag 1

Let analyze the admin login page.

![ctf_cody_first_blog_image](/blogs/ctf_cody_first_blog/flag1_1.png)

I try some simple SQL injection on it, but it doesn’t work.

![ctf_cody_first_blog_image](/blogs/ctf_cody_first_blog/flag1_2.png)

I refer to the hints from Hacker101, but not much useful information.

![ctf_cody_first_blog_image](/blogs/ctf_cody_first_blog/flag1_3.png)

I also tried to access the admin page directly feed in “admin” to the page param, but it said unable to find that file.

![ctf_cody_first_blog_image](/blogs/ctf_cody_first_blog/flag1_4.png)

After some of the research from Internet, I realize the login page name is **“admin.auth.inc”**. I able to access the admin page without login by remove the **auth** from the path.

The flag is at the bottom of the page.

![ctf_cody_first_blog_image](/blogs/ctf_cody_first_blog/flag1_5.png)

## Flag 2

From the hints provided by Hacker101, this flag related to file inclusion vulnerability. 

![ctf_cody_first_blog_image](/blogs/ctf_cody_first_blog/flag2_1.png)

But we need to know what file or URL to apply file inclusion vulnerability.

First I apply “index” into the URL, but it return exhausted memory size error.

![ctf_cody_first_blog_image](/blogs/ctf_cody_first_blog/flag2_2.png)

Next, I tried navigate index page via localhost and it getting interesting. It execute the `echo 'hi'` in the command I submitted before (I did approved the comment when I accessed to the admin page). 

![ctf_cody_first_blog_image](/blogs/ctf_cody_first_blog/flag2_3.png)

I form a PHP code to read the content of `index.php`{: .filepath} and put into comment.

![ctf_cody_first_blog_image](/blogs/ctf_cody_first_blog/flag2_4.png)

Then I approve the comment in the admin page and refresh the page again.

We can see it added a new command with the flag on it, but the flag is belong to flag 1.

Then I inspect the HTML elements and the content of `index.php`{: .filepath} is showing. The flag is inside the contents.

![ctf_cody_first_blog_image](/blogs/ctf_cody_first_blog/flag2_5.png)

## References
- [Hacker101 CTF - Cody's First Blog](https://dev.to/caffiendkitten/hacker101-ctf-cody-s-first-blog-da8)
