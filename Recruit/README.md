# Recruit — Write-Up

### Scenario

Recruit has just launched its new recruitment portal, allowing HR staff to manage candidate applications and administrators to oversee hiring decisions. While the platform appears functional, management suspects that security may have been overlooked during development. Your task is to assess the application like a real attacker, mapping its structure, abusing exposed functionality, and exploiting vulnerabilities.

Can you gain an initial foothold, escalate your access, and ultimately log in as the administrator?

---

### Recon

<img width="1917" height="965" alt="homepage" src="https://github.com/user-attachments/assets/c02d21be-d55a-48cb-98dd-46511539a1d3" />

<img width="1919" height="848" alt="apidetails" src="https://github.com/user-attachments/assets/fe478b0b-a6b4-43a2-9954-39be2ab23206" />

Manual enumeration of the web UI revealed information about the API that should not be publicly accessible:

> - Backend is running PHP
> - **/file.php?cv=<URL>** endpoint with a query string (possibly vulnerable to SSRF)
> - API fetches CVs from external URLs using HTTP and HTTPS
> - API uses a blacklist to block restricted locations rather than a whitelist

<img width="1030" height="720" alt="gobuster" src="https://github.com/user-attachments/assets/392c51c1-3801-4cc1-bdb8-d5b20e3a6299" />

Ran Gobuster after initial recon with -x flag to enumerate directories and any PHP endpoints

<img width="578" height="189" alt="curlfingerprint-phpsessionid" src="https://github.com/user-attachments/assets/ade184b9-0e74-4aab-ad45-784e2c78cc4f" />

Server fingerprinting revealed more information:

> - Ubuntu server running Apache/2.4.41
> - HttpOnly flag is not set (JavaScript running on the page via document.cookie can read that cookie's value)
