<h1>Simple-CTF-THM-Writeup</h1>
This repository is a complete walkthrough of the Simple CTF challenge on TryHackMe, featuring Nmap scanning, directory enumeration with Gobuster, exploitation of CVE-2019-9053, SSH access, and privilege escalation via sudo permissions.<br><br>
<img width="940" height="410" alt="image" src="https://github.com/user-attachments/assets/6b449216-b022-40ea-befa-adf53c13aabd" /><br><br>
Tools Used:<br>
•	Nmap<br>
•	Gobuster<br>
•	SQLmap (failed attempt)<br><br>
I started with a nmap scan and found 3 open ports:<br><br>
<img width="939" height="354" alt="image" src="https://github.com/user-attachments/assets/a57a3a89-803c-47ed-b373-97f692234c9f" /><br><br>
Q: How many services are running under port 1000?<br>
A: 2 (ftp and http)<br><br>
The highest port number has ssh open.<br>
Q: What is running on the higher port?<br>
A: ssh <br><br>
Q: What's the CVE you're using against the application?<br>
This was confusing because there were multiple possible vulnerabilities associated with the services running on the machine. The system was running OpenSSH and Apache (httpd), so my first approach was to explore known vulnerabilities related to OpenSSH. However, none of the commonly referenced CVEs matched the expected answer.<br>
So I took the hint and it said it was discovered in “CMS Made Simple 2.2.8”, which led me to research its known exploits. This version of CMS Made Simple is affected by a critical vulnerability identified as:<br>
A: CVE-2019-9053<br><br>
This clarified the exploit path. The target application exposed through the web server was vulnerable to SQL Injection, enabling attackers to retrieve sensitive information through crafted requests.<br>
Upon visiting the IP address in a browser, I was initially met with a default Apache page, which masked the presence of the CMS backend.<br><br>
<img width="939" height="836" alt="image" src="https://github.com/user-attachments/assets/24b0ffae-b3c4-4f09-bce6-8fb2476e639f" /><br><br>
While searching for what to do next, I landed on the sqlmap tools documentation where they have given the examples of the attack.<br>
https://www.kali.org/tools/sqlmap/<br>
I performed the first attack thing given which finds what’s in the database apparently. I tried it but it said my sqlmap is outdated so I upgraded it.<br><br>
sqlmap -u "http://10.201.61.220/?p=1&forumaction=search" –dbs<br><br>
<img width="941" height="429" alt="image" src="https://github.com/user-attachments/assets/51abdc49-0d9f-496f-804a-5bda7c48a46e" /><br><br>
Did not really work.<br><br>
“Furthermore, compared to 2022, in 2023, SQL injection vulnerabilities were identified as CVEs 2159 times. And in the latest OWASP Top 10, which lists the most critical and common vulnerabilities in web applications, they rank third.” <br>
(https://www.vaadata.com/blog/sqlmap-the-tool-for-detecting-and-exploiting-sql-injections/)<br><br>
After a lot of trial and error, I learned an important lesson. When you visit an IP address and it shows you a web page, whether it’s just one page or several, it’s always worth checking for any hidden or additional pages. These extra directories can hold important clues that aren’t visible at first glance, and they often guide you toward the real entry points of the application.
And to do so, you need to use gobuster, a tool used to check for hidden directories in a web page. <br><br>
gobuster dir --url http://10.201.40.124/ --wordlist /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt --output gobuster_80.txt <br><br>
Gobuster found something: /simple and this site has a search bar!<br><br>
<img width="938" height="357" alt="image" src="https://github.com/user-attachments/assets/b5064cca-d57d-4a22-ae42-23fbcb667a47" /><br><br>
I found this github repo where there’s a CVE 2019-9053 exploit. So I downloaded in on my machine and ran it using this cmd:<br>
Python exploit.py -u http://10.201.40.124/simple --crack -w /usr/share/wordlists/rockyou.txt<br><br>
<img width="939" height="461" alt="image" src="https://github.com/user-attachments/assets/63b5627b-3fb3-4ec0-8607-018d4d52a1f0" /><br><br>
Got the password!<br><br>
Now I got all the credentials. Remember ssh was open? So let’s enter there.<br>
Format : ssh username@ip<br>
ssh mitch@10.201.40.124<br><br>
<img width="939" height="650" alt="image" src="https://github.com/user-attachments/assets/300fb8dd-78ec-4d25-bc0c-8fe0eeb24749" /><br><br>
Found another user other than mitch, sunbath. Now we need to escalate our privileges.<br>
For that, we need to know what thing mitch can do rn. To check that:<br>
sudo -l<br><br>
<img width="731" height="347" alt="image" src="https://github.com/user-attachments/assets/78e73ef8-c345-4c22-a344-8c9ec43e8fa7" /><br><br>
So we can access vim without password. So let’s search how.<br><br>
<img width="940" height="222" alt="image" src="https://github.com/user-attachments/assets/1329aa5e-b1a9-4045-aa8b-1d8ae2cf9cb5" /><br><br>
(https://gtfobins.github.io/gtfobins/vim/)<br><br>
Became root!<br><br>
<img width="759" height="618" alt="image" src="https://github.com/user-attachments/assets/eaf501b2-900f-4a5e-b0db-e43c163a571c" /><br><br>
Hooray! I did it!
