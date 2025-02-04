This project is designed to develop ethical hacking skills, including information gathering, exploiting web application vulnerabilities, bypassing system restrictions, and privilege escalation. The attack process involves analyzing the exposed infrastructure, identifying potential entry points, and using various penetration techniques such as directory brute-forcing, SQL injection, LFI vulnerabilities, and file upload bypass methods.

The reconnaissance phase began with a full port scan of the target system using nmap. The results showed two web servers running on ports 80 and 1337, as well as FTP, SSH, and an MQTT service. Checking the FTP service revealed that anonymous login was allowed, but no useful files were found.

Next, the focus shifted to the web server on port 1337. A simple page visit yielded no useful information, but a directory brute-force scan using ffuf uncovered hidden routes, including /admin and /admin_101. The first one turned out to be a decoy, while the second presented a login form with a pre-filled username: hacker@root.thm. Attempts to log in showed a response to incorrect credentials, indicating the possibility of SQL injection.

The login request was intercepted in Burp Suite and passed to sqlmap, which confirmed that the email parameter was vulnerable. Further analysis extracted the expose database, which contained the user and config tables. As a result, administrator credentials were found and successfully used to log into /admin_101.

The admin panel was an AI chat system with a hint suggesting hidden parameters. Analyzing the page source revealed a clue pointing to possible file or view parameters. Testing confirmed that the file parameter was vulnerable to LFI, allowing access to /etc/passwd and revealing system usernames.

Further investigation of the database uncovered another restricted section, /upload-cv00101011/index.php, which required a password. A hint suggested that the password was the name of a system user starting with "z", which matched zeamkish from /etc/passwd. Entering this username granted access to a file upload page.

Analyzing the page source revealed that file validation only checked for extensions (jpg and png). This allowed bypassing the restriction by first naming the file with a .jpg extension and then modifying the request in Burp Suite to .php. This method successfully uploaded a php-reverse-shell script from pentestmonkey. After checking the page source for the uploaded file’s location, it was executed on the server while a netcat listener was running, providing shell access.

Once inside the system, the home directory was explored, revealing two users: ubuntu and zeamkish. In zeamkish's home directory, a file named ssh_creds.txt contained login credentials. Using these credentials, SSH access was successfully obtained.

Next, privilege escalation methods were explored. Checking sudo -l showed that the user had no elevated privileges. However, searching for binaries with the SUID bit set revealed find. Using a known technique from GTFOBins, the command /usr/bin/find . -exec /bin/sh -p \; -quit was executed, granting root access. This resulted in obtaining superuser privileges and access to the final flag.
