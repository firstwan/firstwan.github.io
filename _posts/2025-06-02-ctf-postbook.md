---
title: Hacker101 - Postbook
author: first_wan
categories: [Cybersecurity, CTF]
tags: [Hacker101, CTF]
pin: false
---
This CTF are provider by Hacker101 with easy difficulty. It have 7 flags to find.

## Reconnaissance

As we redirect to the page, it show a simple web site with just “Sign in” and “Sign up” features.

![ctf_postbook_image](/blogs/ctf_postbook/recon_1.png)

The first method I try was unauthorized access with SQL injection on sign in page.

![ctf_postbook_image](/blogs/ctf_postbook/recon_2.png)

But it seem like it have protection on it.

![ctf_postbook_image](/blogs/ctf_postbook/recon_3.png)

Next I sign up a account with admin as username with weak password.

![ctf_postbook_image](/blogs/ctf_postbook/recon_4.png)

I able to log in with my account.

![ctf_postbook_image](/blogs/ctf_postbook/recon_5.png)

## Flag 1

I click one of the post from “Post timeline”.

![ctf_postbook_image](/blogs/ctf_postbook/flag1_1.png)

And found id=3 from the URL.

![ctf_postbook_image](/blogs/ctf_postbook/flag1_2.png)

I try to input different id into the URL and found the flag in id 2.

![ctf_postbook_image](/blogs/ctf_postbook/flag1_3.png)

## Flag2

Now let turn on Burp Suite and play on the POST request. We are able to create new post from the home page.

![ctf_postbook_image](/blogs/ctf_postbook/flag2_1.png)

Let turn on the intercept and create a new post.

![ctf_postbook_image](/blogs/ctf_postbook/flag2_2.png)

After I study the POST request, it put the `user_id` as part of the request to indicate who are the owner of this post. Let change to other user id and forward the request.

![ctf_postbook_image](/blogs/ctf_postbook/flag2_3.png)

The flag will show on home page.

![ctf_postbook_image](/blogs/ctf_postbook/flag2_4.png)

## Flag 4

Let do the same thing for post edit request. First we create a post.

![ctf_postbook_image](/blogs/ctf_postbook/flag4_1.png)

After created the post, we will able to see edit button on the post.

![ctf_postbook_image](/blogs/ctf_postbook/flag4_2.png)

Click the button to edit the post.

![ctf_postbook_image](/blogs/ctf_postbook/flag4_3.png)

Before you click the “Save post” button, turn on Burp Suite and intercept the POST request. We can see that the id are not in the request payload but in the URL request. Modify the id to other id and forward the request.

![ctf_postbook_image](/blogs/ctf_postbook/flag4_4.png)

The flag will show on the page.

![ctf_postbook_image](/blogs/ctf_postbook/flag4_5.png)

## Flag 0

From the hint provided by Hacker101, this flag is related to a account name as “user”.

![ctf_postbook_image](/blogs/ctf_postbook/flag0_1.png)

This hint state that the account have a weak password. I decide to manually test some of the password before I use any tools too brute force it.

After few try with the top 10 popular weak password, I successfully login as “user.” And the flag is in the home page.

![ctf_postbook_image](/blogs/ctf_postbook/flag0_2.png)

## Flag 3

From this hint provided by Hacker101, we will get a number of 945.

![ctf_postbook_image](/blogs/ctf_postbook/flag3_1.png)

We will able to get the flag by search post related id 945.

![ctf_postbook_image](/blogs/ctf_postbook/flag3_2.png)

## Flag 5 & Flag 6

From the hint provided by Hacker101, it seem like related to section id store in cookie.

![ctf_postbook_image](/blogs/ctf_postbook/flag5_1.png)

When we login into the web site, the server will assign a section id for us. And after some research, this section id is the user id hashed with MD5.

![ctf_postbook_image](/blogs/ctf_postbook/flag5_2.png)

To log in as user with ID 1, we need to generate MD5 hash for digital 1 and replace the cookie in our browser.

We can use [CyberChef](https://gchq.github.io/CyberChef/#recipe=MD5()&input=MQ) to generate the MD5 hash.

![ctf_postbook_image](/blogs/ctf_postbook/flag5_3.png)

And replace the cookie in browser on developer tools.

![ctf_postbook_image](/blogs/ctf_postbook/flag5_4.png)

Flag 5 will show in home page.

![ctf_postbook_image](/blogs/ctf_postbook/flag5_5.png)

Flag 6 also the same, but you need to use your own account to delete other post that own by other user.

The post id had been MD5 hashed, you need to replace with the MD5 hashed post id you want to delete.

![ctf_postbook_image](/blogs/ctf_postbook/flag5_6.png)

## References
- [Hacker101 CTF - Postbook](https://dev.to/caffiendkitten/ctf-postbook-2dpd)
