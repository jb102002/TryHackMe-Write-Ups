# Operation Coldstart Write Up

### To begin we are prompted with the following scenario:
Volt Labs, a small SaaS shop, suspects an old staging server has rotted into an exposed liability. Mara has assigned you the engagement. Find your way in and demonstrate full compromise.

## To begin, lets start with a simple nmap scan to detect what services are running on the target server and spot any vulnerable versions of these services

<img width="1916" height="917" alt="image" src="https://github.com/user-attachments/assets/048239c2-3364-4d99-b37f-5f94c0b97ffa" />

As we can see from the following scan, Volt Labs has the following open ports:
- Port 21: FTP Control Port running vsftpd 3.0.5
- Port 22: SSH port running OpenSSH 9.6p1 Ubuntu 3ubuntu13.16 (Ubuntu Linux; protocol 2.0)
- Port 80: HTTP port running gunicorn

