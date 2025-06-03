---
title: Hacker101 - Photo Gallery
author: first_wan
categories: [Cybersecurity, CTF]
tags: [SQL Injection, Hacker101, CTF]
pin: false
---

This CTF is provided by Hacker101 with moderate difficulty. It have 3 flags to find.

## Reconnaissance

The page show us 3 images. 

![ctf_photo_gallery_image](/blogs/ctf_photo_gallery/reconnaissance_1.png)

I start with inspect the network it used.

![ctf_photo_gallery_image](/blogs/ctf_photo_gallery/reconnaissance_2.png)

Since it fetch the images with sequential id. I tried to loop the id from 1-100, but no luck.

![ctf_photo_gallery_image](/blogs/ctf_photo_gallery/reconnaissance_3.png)

## Flag 1

After I done some research, it seem like vulnerable to SQL injection. So I try use `SQLmap` to exploit it, and it vulnerable to blind SQL injection. Below is the command I used:

`sqlmap -u https://<hacker one URL>/fetch?id=1 --dbs`

### Enumerate database

![ctf_photo_gallery_image](/blogs/ctf_photo_gallery/flag1_1.png)

### Enumerate tables

![ctf_photo_gallery_image](/blogs/ctf_photo_gallery/flag1_2.png)

### Enumerate column

![ctf_photo_gallery_image](/blogs/ctf_photo_gallery/flag1_3.png)

### Dump data from photos table

![ctf_photo_gallery_image](/blogs/ctf_photo_gallery/flag1_4.png)

We able to find flag1 from the database.

## Flag 0

We can see they store the file path as `filename` into database. We look back to the HTTP request to fetch the image.

![ctf_photo_gallery_image](/blogs/ctf_photo_gallery/flag0_1.png)

HTTP request get the file path with id.

So I guess the SQL query would be something like

`SELECT filename from photos where id=<input>` 

Which mean if I able to return a file path from a query, it will return the content of the file.

![ctf_photo_gallery_image](/blogs/ctf_photo_gallery/flag0_2.png)

From the hints provided by Hacker101, it mention something about union. Maybe we can utilize the `UNION SELECT` to return file path from SQL.

Now let try to get `/etc/passwd`{: .filepath} file in Linux system.

`SELECT filename from photos where id=0 UNION SELECT '../../etc/passwd'` 

> We need to specific **id=0** to return empty result from first query in order to return `'../../etc/passwd'` as first result for the application.
{: .prompt-tip }

But this SQL return me nothing. 

Now I refer back hint 3, it said “runs on the **uwsgi-nginx-flask-docker** image”. Flash is one of the popular framework build on top of Python.

Most of the Python application have a **`main.py`** file. Let try to output **`main.py` .**

`SELECT filename from photos where id=0 UNION SELECT 'main.py'`

![ctf_photo_gallery_image](/blogs/ctf_photo_gallery/flag0_3.png)

It return me the whole application file, the flag is at bottom of the file.

## Flag 2

Since we got the whole application file, we can study a bit on the code.

It have 2 endpoint, one is load the albums page as we see, another one is the endpoint used to fetch the images.

![ctf_photo_gallery_image](/blogs/ctf_photo_gallery/flag2_1.png)

From the image above, there are something very interesting on line 59. It seem like spawn a new process and executing system command. Which mean if we able to inject a command into it, it will run our command.

Look deeper on how it form the system command in this application, it seem like this application retrieves all the filename from the database and concatenate all the filename together to form the system command.

So the next question will be how to put our command into the database.

![ctf_photo_gallery_image](/blogs/ctf_photo_gallery/flag2_2.png)

Hints 2 mentioned `Stacked queries`, it a technique in SQL injection to perform multiple operation in single query.

Let form the update SQL statement for our case.

`/fetch?id=1;+UPDATE+photos+SET+filename=";+echo+$(ls)"+where+id=3;commit;`

- `id=1;` - Query the id 1 as normal query
- `UPDATE+photos+SET+filename=";+echo+$(ls)"+where+id=3;` - Update the filename as **“echo $(ls)”** for photo id = 3
- `commit;` - Save the updated data.

![ctf_photo_gallery_image](/blogs/ctf_photo_gallery/flag2_3.png)

After it done, we can refresh our page to check the result.

![ctf_photo_gallery_image](/blogs/ctf_photo_gallery/flag2_4.png)

It successfully list all the files in the directory.

Now look at hint 3 — **“Be aware of your environment”**. It sound like we can get the answer from environment variables.

Let change our command to list all the environment variables.

`/fetch?id=1;+UPDATE+photos+SET+filename="**;+echo+$(printenv)**"+where+id=3;commit;`

We able to see all the flags in environment variables.

![ctf_photo_gallery_image](/blogs/ctf_photo_gallery/flag2_5.png)

## References

- [Hacker101 CTF — Photo Gallery — All FLAGS](https://medium.com/@gus3rmr/hacker101-ctf-photo-gallery-all-flags-25c0136b10e2)
- [Hacker101 CTF - Photo Gallery](https://dev.to/caffiendkitten/hacker101-ctf-photo-gallery-4foi)
- [Hacker101 CTF - Photo Gallery (Medium) - Live Walkthrough](https://www.youtube.com/watch?v=HDOm7ZmSjJw)
- [Linux List All Environment Variables Command](https://www.cyberciti.biz/faq/linux-list-all-environment-variables-env-command/)
