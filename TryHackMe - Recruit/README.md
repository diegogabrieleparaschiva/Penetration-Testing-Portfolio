# TryHackMe - Recruit Writeup

This challenge is part of the "Web Application Vulnerabilities I" module, in which we studied the five of the most impactful vulnerabilities in modern web applications: SQL Injection, CSRF, XSS, SSRF and IDOR. With this in mind, this challenge most certainly wants to test what we studied from this module.

<img width="740" height="129" alt="image" src="https://github.com/user-attachments/assets/83e14fca-c0bc-4a83-bb12-4a35bd47ee5b" />
<img width="1272" height="442" alt="image" src="https://github.com/user-attachments/assets/1d00eaed-e0fb-4dc9-b89d-53733c5da03c" />

Let's start with reconaissance. We'll use nmap to scan for open ports.

<img width="956" height="530" alt="image" src="https://github.com/user-attachments/assets/c4ffcef3-8991-49a6-b070-e123cffefb1f" />

Ports 22, 53 and 80 are open. Let's visit port 80.

<img width="507" height="347" alt="image" src="https://github.com/user-attachments/assets/cba8f774-9853-4c2b-903d-e230cc6ef35a" />

We are greeted with a login page (no user registration button), and an `api.php` page that had information on using the Recruit API. There we find out that we could use `/file.php?cv=<URL>` to access any candidate's CV. I gave a lot of values to the cv parameter but the following error message persisted every time: `Only local files are allowed`.

Let's run a directory enumeration now using gobuster.

<img width="1027" height="614" alt="image" src="https://github.com/user-attachments/assets/d1700dbf-f12b-4910-a545-5b90aea763e8" />

We found some interesting subdirectories. By inspecting each one of them, the one that caught my eye was the `/mail` one. It contains a log file: `mail.log` Let's take a look inside of it:

<img width="964" height="598" alt="image" src="https://github.com/user-attachments/assets/ea874fa9-6900-4a3c-b409-417e8bc479ad" />

Just like that we found the username "hr" and the location of the password being inside `config.php`. We also found out that the administrator credentials were stored within a backend database, confirming my intuition (we'll most likely use SQL Injection).

After pasting the value `file://config.php` in the cv parameter, we get into `config.php`.

<img width="560" height="555" alt="image" src="https://github.com/user-attachments/assets/5811517b-2878-43e9-97f4-69fc1a16a06c" />

Perfect! We have the username "hr" and the password "hrpassword123". Let's grab the first flag by logging in as a normal user!

<img width="1920" height="799" alt="image" src="https://github.com/user-attachments/assets/19dd335b-0dd3-4b86-905d-cd4c2eff963d" />



