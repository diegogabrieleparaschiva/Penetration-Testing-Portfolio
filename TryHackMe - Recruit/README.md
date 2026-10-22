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

Now let's get to serious business and get the second flag, trying to use SQL Injection.

<img width="1339" height="318" alt="image" src="https://github.com/user-attachments/assets/865f10a2-7407-434f-a3b0-8a83ae9d9ff6" />

Well.. there we go. We are dealing with a MySQL backend. Next let's see how many columns we have. I typed `' union select 1-- -`, then `' union select 1,2-- -`, then `' union select 1,2,3-- -` and finally `' union select 1,2,3,4-- -` confirming there were 4 columns.

<img width="1331" height="419" alt="image" src="https://github.com/user-attachments/assets/c18250b5-55fd-45f8-b626-f4adbaf3959f" />

Next, I got the tables from the database using `' union select table_name,2,3,4 from information_schema.tables-- -`. We got a really big list of tables, and we found one particular database that looked interesting: "users"

<img width="1404" height="126" alt="image" src="https://github.com/user-attachments/assets/4f84097d-16a9-4f37-a92f-47e9df119ffb" />

Let's find the columns in the users table using this command `' union select column_name,2,3,4 from information_schema.columns where table_name='users'-- -`.

<img width="1350" height="614" alt="image" src="https://github.com/user-attachments/assets/e52b3066-57b1-4bd4-84f5-4eaaa1459a69" />

Usernames and passwords.. That should get us closer to our flag. Let's run our last SQL Injection `' union select 1,password,3,username from users-- -`!

<img width="1353" height="342" alt="image" src="https://github.com/user-attachments/assets/8a6b4515-2603-46bf-a9cb-cd40f594f192" />

And just like that we have the admin credentials. Now we'll login as the admin, and get our second and final flag of this room!

<img width="1333" height="565" alt="image" src="https://github.com/user-attachments/assets/35cd67ba-2d71-472d-9603-8e1411207414" />

And that's the finale of this CTF room. I want to mention that I know we could have used SQLMAP to skip the whole process of manual exploitation of SQLi, but I'm still a newbie at penetration testing (I switched from SOC because I was not sure which path to choose in cybersecurity and I wanted to test some other paths) and I don't have the experience yet to use SQLMAP. So far I really enjoy breaking systems and I'm excited to learn so much more!

Let's now talk about some remediation.
Firstly, let's talk about the SQL Injection problem. The login mechanism is currently vulnerable to SQL Injection because it concatenates unsanitized user input directly into the SQL query. The solution is to use prepared statements (parameterized queries): This is the primary and most effective defense against SQLi. It forces the database to treat user input strictly as data, rather than executable code, neutralizing payloads like `' OR 1=1 --`. Also let's implement strict validation to ensure that fields like "username" only accept expected characters (e.g., alphanumeric) and reject SQL meta-characters.
Finally let's talk about Local File Inclusion (LFI) (which we'll learn in the next module "Web Application Vulnerabilities II"): The API endpoint (`file.php?cv=`) attempts to implement filters (blocking `file://` or external URLs), but relying on blacklist filters is prone to bypasses. First, you should avoid direct file paths: never allow user input to directly dictate the file path or URL that the server reads. Second, implement indirect references: map downloadable CVs to a database ID or a secure hash (e.g., `file.php?cv_id=5`). The backend should then look up the exact, hardcoded internal path associated with that ID. And third, implement strict whitelisting: if filenames must be passed, validate the input against a strict whitelist (e.g., only allowing alphanumeric characters and `.pdf` extensions) and enforce that files can only be read from a dedicated, restricted `uploads/` directory.

That's all for this room! See ya in the next writeups! :)
