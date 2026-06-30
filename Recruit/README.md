# Recruit — Write-Up

### Scenario

Recruit has just launched its new recruitment portal, allowing HR staff to manage candidate applications and administrators to oversee hiring decisions. While the platform appears functional, management suspects that security may have been overlooked during development. Your task is to assess the application like a real attacker, mapping its structure, abusing exposed functionality, and exploiting vulnerabilities.

Can you gain an initial foothold, escalate your access, and ultimately log in as the administrator?

### Recon

<img width="1917" height="965" alt="image" src="https://github.com/user-attachments/assets/c02d21be-d55a-48cb-98dd-46511539a1d3" />

<img width="1919" height="848" alt="image" src="https://github.com/user-attachments/assets/fe478b0b-a6b4-43a2-9954-39be2ab23206" />

Manual enumeration of the web UI revealed information about the API that should not be publicly accessible:
1. Backend is running PHP
2. **/file.php?cv=<URL>** endpoint with a query string (possibly vulnerable to SSRF)
3. API fetches CVs from external URLs using HTTP and HTTPS
4. API uses a blacklist to block restricted locations rather than a whitelist

