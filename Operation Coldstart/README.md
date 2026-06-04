# Operation Coldstart Write Up

### To begin we are prompted with the following scenario:
Volt Labs, a small SaaS shop, suspects an old staging server has rotted into an exposed liability. Mara has assigned you the engagement. Find your way in and demonstrate full compromise.

## To begin, lets start with a simple nmap scan to detect what services are running on the target server and spot any vulnerable versions of these services

<img width="1916" height="917" alt="image" src="https://github.com/user-attachments/assets/048239c2-3364-4d99-b37f-5f94c0b97ffa" />

As we can see from the following scan, Volt Labs has the following open ports:
- Port 21: FTP Control Port running vsftpd 3.0.5
- Port 22: SSH port running OpenSSH 9.6p1 Ubuntu 3ubuntu13.16 (Ubuntu Linux; protocol 2.0)
- Port 80: HTTP port running gunicorn

## Now that we know the server is running HTTP we can run a directory brute force scan using gobuster to gain some more information

<img width="1216" height="684" alt="image" src="https://github.com/user-attachments/assets/62781580-c290-4176-b46b-337c9d978d81" />

We have enumerated an /admin and /preview endpoint

I also tested for common file extensions e.g. .env, .txt, .py but it did not yield any results.

## Lets take a look at the website

<img width="1916" height="964" alt="image" src="https://github.com/user-attachments/assets/971c217e-0778-419f-8712-b6453501efca" />

Interesting... The website acts as a URL Previews service that fetches external URLs and displays previews of the contents in your browser

This functionality is a classic Server Side Request Forgery Target (SSRF). If we can borrow the server's network position to possibly gain access to the contents of restricted directories within the application.

## Lets check our endpoints

### /admin

<img width="1276" height="620" alt="image" src="https://github.com/user-attachments/assets/ecac16b5-313d-401f-9f40-83d71772cd07" />

This is as expected if we refer to the HTTP 308 status code in the dirbuster scan

### /preview

<img width="1278" height="626" alt="image" src="https://github.com/user-attachments/assets/93aaef9f-c822-49b6-ae56-346d151a26d3" />

This further confirms our suspicions of a possible SSRF entry point
