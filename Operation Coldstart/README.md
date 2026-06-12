# Operation Coldstart Write Up

### To begin we are prompted with the following scenario:
Volt Labs, a small SaaS shop, suspects an old staging server has rotted into an exposed liability. Mara has assigned you the engagement. Find your way in and demonstrate full compromise.

**To begin, lets start with a simple nmap scan to detect what services are running on the target server and spot any vulnerable versions of these services**

<img width="1916" height="917" alt="image" src="https://github.com/user-attachments/assets/048239c2-3364-4d99-b37f-5f94c0b97ffa" />

As we can see from the following scan, Volt Labs has the following open ports:
- Port 21: FTP Control Port running vsftpd 3.0.5
- Port 22: SSH port running OpenSSH 9.6p1 Ubuntu 3ubuntu13.16 (Ubuntu Linux; protocol 2.0)
- Port 80: HTTP port running gunicorn

**Now that we know the server is running HTTP we can run a directory brute force scan using gobuster to gain some more information**

<img width="1216" height="684" alt="image" src="https://github.com/user-attachments/assets/62781580-c290-4176-b46b-337c9d978d81" />

We have enumerated an /admin and /preview endpoint

I also tested for common file extensions e.g. .env, .txt, .py but it did not yield any results.

**Lets take a look at the website**

<img width="1916" height="964" alt="image" src="https://github.com/user-attachments/assets/971c217e-0778-419f-8712-b6453501efca" />

Interesting... The website acts as a URL Previews service that fetches external URLs and displays previews of the contents in your browser. The "© Volt Labs · do not expose externally" footer in the page confirms that this page should not be public. It is anyways.

This functionality is a classic Server Side Request Forgery (SSRF) target. If so, we can possibly borrow the server's network position to reach internal services.

**Lets check our endpoints**

**/admin**

<img width="1276" height="620" alt="image" src="https://github.com/user-attachments/assets/ecac16b5-313d-401f-9f40-83d71772cd07" />

This endpoint redirects us to the /admin/ directory with a HTTP 403 Forbidden Status Code.

**/preview**

<img width="1278" height="626" alt="image" src="https://github.com/user-attachments/assets/93aaef9f-c822-49b6-ae56-346d151a26d3" />

This further confirms our suspicions of a possible SSRF entry point

Notice the "staging" label in the top right-hand corner. This tells us that this is likely a dev/staging environment and could have fewer protections

**Lets look into this endpoint in more depth**

Upon testing an SSRF payload we are met with this message:

<img width="1282" height="626" alt="image" src="https://github.com/user-attachments/assets/2fdd6481-ac5a-4306-a870-b56292b619af" />

I tried a couple more obfuscation methods for the query parameter:
?url=http://127.0.0.1
?url=http://0.0.0.0
?url=http://[::1]
?url=http://2130706433   (decimal) 
?url=http://0x7f000001   (hex)

There seems to be a server-side allow list that is blocking our attempts to gain network positioning.

I also pulled up Burp Suite and checked if there were any header injection vulnerabilites that we could take advantage of including a Host header but this did not prove to be worth while.

**The allowlist on the website seem to be pretty strict. After a little while of testing I moved to the FTP service to see if we had any luck there**

<img width="1916" height="957" alt="image" src="https://github.com/user-attachments/assets/6fdd169f-1a56-4225-8675-ea74024ff7c3" />

Anonymous FTP login was enabled on the server

<img width="809" height="559" alt="image" src="https://github.com/user-attachments/assets/b3967466-6fc0-49d4-a1b7-2006f8840afe" />

I have found a file named "backup.tar.gz". I will download the file to see what it contains

**After unpacking the tar.gz file it appears we have a new directory named "/voltlabs-preview"**

Inside this directory we have a README.md, app.py, and requirements.txt file.

Lets look at the README.md file

**README.md**

<img width="656" height="125" alt="image" src="https://github.com/user-attachments/assets/4a956637-63fa-4856-a65b-f7953531ff9b" />

**requirements.txt**

<img width="620" height="81" alt="image" src="https://github.com/user-attachments/assets/00ce4615-8831-40ce-8970-cfa96a1486bd" />

**app.py**

<img width="956" height="657" alt="image" src="https://github.com/user-attachments/assets/f21aefea-ed17-4f2f-a40c-ba7bbfb83e42" />

We seem to have finally found the allow list for the URL preview service which is "kestrel.thm". Better yet, this domain resolves to local host on the server side. We may finally be able to access the /admin/ directory.

**Testing the admin endpoint with the new domain**

<img width="1122" height="888" alt="image" src="https://github.com/user-attachments/assets/a18e90f3-9781-4d5b-b8cd-0c565327a0f3" />

It appears we officially have detected an SSRF vulnerability using this URL preview input.

Upon looking further into the python script, we find something interesting:

<img width="953" height="657" alt="image" src="https://github.com/user-attachments/assets/5c09e5a2-ae6b-4bd5-b92d-4a4d5e538b76" />

We find that the endpoint /admin/notes exits.

**Testing /admin/notes**

<img width="1116" height="885" alt="image" src="https://github.com/user-attachments/assets/f323bbd0-fa3a-4d9a-ade3-d7e8a5527b47" />

We now have the web developer's SSH credentials

After SSH'ing into the target machine we are able to locate the user.txt file which is flag one

<img width="813" height="564" alt="image" src="https://github.com/user-attachments/assets/5f2348d6-f481-44dd-811b-9fc1d6a62b56" />

After running "find / -name "flag.txt" 2>/dev/null", I concluded that the second flag does not exist on the SSH server






