# Recruit — Write-Up

### Scenario

Recruit has just launched its new recruitment portal, allowing HR staff to manage candidate applications and administrators to oversee hiring decisions. While the platform appears functional, management suspects that security may have been overlooked during development. Your task is to assess the application like a real attacker, mapping its structure, abusing exposed functionality, and exploiting vulnerabilities.

Can you gain an initial foothold, escalate your access, and ultimately log in as the administrator?

---

### Recon

<img width="578" height="189" alt="curlfingerprint-phpsessionid" src="https://github.com/user-attachments/assets/ade184b9-0e74-4aab-ad45-784e2c78cc4f" />

Server fingerprinting the following information:

> - Ubuntu server running Apache/2.4.41
> - HttpOnly flag is not set (JavaScript running on the page via document.cookie can read that cookie's value)

Considering this is a CTF challenge, there is not much research to perform in this stage.

---

### Enumeration

<img width="1917" height="965" alt="homepage" src="https://github.com/user-attachments/assets/c02d21be-d55a-48cb-98dd-46511539a1d3" />

<img width="1919" height="848" alt="apidetails" src="https://github.com/user-attachments/assets/fe478b0b-a6b4-43a2-9954-39be2ab23206" />

**Walking the application revealed information about the API that should not be publicly accessible:**

> - Backend is running PHP
> - **/file.php?cv=<URL>** endpoint with a query string (possibly vulnerable to SSRF/LFI)
> - API fetches CVs from external URLs using HTTP and HTTPS
> - API uses a blacklist to block restricted locations rather than a whitelist

<img width="1030" height="720" alt="gobuster" src="https://github.com/user-attachments/assets/392c51c1-3801-4cc1-bdb8-d5b20e3a6299" />

**Ran Gobuster after initial recon with -x flag to enumerate directories and any PHP endpoints**

<img width="979" height="864" alt="maillog" src="https://github.com/user-attachments/assets/0e95ab28-d1ca-4c13-9041-a96a79f1594c" />

**Mail.log was found after initial enumeration that revealed sensitive info regarding user accounts including:**

> - Organization emails
>   - hr@recruit.thm
>   - it-support@recruit.thm
> - Logged email including HR login credentials' configuration file (config.php) and the HR username

**Note:** Admin credentials are not stored in the application. They are stored within the backend database.

Directly accessing the config.php file 

<img width="648" height="820" alt="assets" src="https://github.com/user-attachments/assets/7b399695-c707-42c8-9a91-55b37c94608e" />

The /assets endpoint serves a directory listing rather than an index file. This endpoint did not have information to further enumeration however it was worth noting.

### Testing the Application

**Testing the /file.php?cv=<URL> endpoint with file://config.php exposed source code for the file**


<img width="730" height="848" alt="config" src="https://github.com/user-attachments/assets/84debef4-c404-468a-89c5-047a58a849b0" />

We have confirmed LFI (Local File Inclusion) and obtained the temporary HR password for production

**Using the HR username we obtained from the email log and the newly obtained password, we have gained access through the login portal, logged in as a normal user, and obtained the first flag**

