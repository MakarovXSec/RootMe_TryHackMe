 __  __          _  __           _____   ______      _ 
|  \/  |   /\   | |/ /    /\    |  __ \ / __ \ \    / /
| \  / |  /  \  | ' /    /  \   | |__) | |  | \ \  / / 
| |\/| | / /\ \ |  <    / /\ \  |  _  /| |  | |\ \/ /  
| |  | |/ ____ \| . \  / ____ \ | | \ \| |__| | \  /   
|_|  |_/_/    \_\_|\_\/_/    \_\|_|  \_\\____/   \/    

╔════════════════════════════════════════════════════════════════════════╗
║                      SECURITY ASSESSMENT REPORT                        ║
║                        TARGET: RootMe (TryHackMe)                      ║
║                        OPERATOR: makarov_XSec                          ║
╚════════════════════════════════════════════════════════════════════════╝

[+] PHASE 01: ENUMERATION
----------------------------------------------------------------------------------------------------------------------
| STEP              | COMMAND / METHOD                                                   | RESULT                          |
|-------------------|--------------------------------------------------------------------|---------------------------------|
| 1. Connectivity   | ping                                                               | Host reachable                  |
| 2. Port Scan      | nmap -p- -sS -sC -sV --open --min-rate=2000 -vvv -Pn IP            | Ports 22 (SSH), 80 (HTTP) OPEN  |
| 3. Web Analysis   | Manual + Source Code                                               | No useful info found            |
| 4. Directory Scan | gobuster                                                           | /uploads and /panel discovered  |
----------------------------------------------------------------------------------------------------------------------

[+] PHASE 02: INITIAL ACCESS
----------------------------------------------------------------------------------------------------------------------
| STEP              | ACTION                                   | RESULT                                  |
|-------------------|------------------------------------------|-----------------------------------------|
| 1. Upload Vector  | File upload functionality                | Upload allowed                          |
| 2. Bypass Filter  | Change extension (.php → .phtml)         | Upload successful                       |
| 3. Payload Exec   | Access payload via /uploads              | Reverse shell obtained                  |
| 4. Shell Access   | Netcat listener (port 443)               | Remote shell as www-data                |
| 5. Stabilization  | Reverse shell (port 444)                 | Stable shell achieved                   |
----------------------------------------------------------------------------------------------------------------------

[+] PHASE 03: POST-EXPLOITATION
----------------------------------------------------------------------------------------------------------------------
| STEP              | ACTION                                   | RESULT                                  |
|-------------------|------------------------------------------|-----------------------------------------|
| 1. Enumeration    | Manual directory exploration             | Limited access (non-root)               |
| 2. File Discovery | /var/www                                 | user.txt found                          |
| 3. Flag Retrieval | cat user.txt                             | THM{y0u_g0t_a_sh3ll}                    |
----------------------------------------------------------------------------------------------------------------------

[+] PHASE 04: PRIVILEGE ESCALATION
----------------------------------------------------------------------------------------------------------------------
| STEP              | COMMAND / METHOD                         | RESULT                                  |
|-------------------|------------------------------------------|-----------------------------------------|
| 1. SUID Scan      | find / -perm -4000 2>/dev/null           | Multiple SUID binaries found            |
| 2. Analysis       | GTFOBins                                 | Python SUID exploitation identified     |
| 3. Exploitation   | python -c 'import os; os.execl(...)'     | Root shell obtained                     |
| 4. Final Access   | /root directory                          | Root flag retrieved                     |
----------------------------------------------------------------------------------------------------------------------

[+] PHASE 05: VULNERABILITY DISCLOSURE
----------------------------------------------------------------------------------------------------------------------
1. FINDING: Unrestricted File Upload (Critical)
   DETAILS: The application allows arbitrary file uploads with insufficient validation, enabling remote code execution.
   FIX: Implement strict file type validation and disable execution in upload directories.

2. FINDING: Insecure File Extension Filtering (High)
   DETAILS: Upload restrictions can be bypassed by changing file extensions (e.g., .php to .phtml).
   FIX: Use server-side MIME validation and whitelist allowed extensions.

3. FINDING: SUID Misconfiguration (High)
   DETAILS: SUID-enabled Python binary allows privilege escalation to root.
   FIX: Remove unnecessary SUID permissions from binaries.

4. FINDING: Poor Access Control (Medium)
   DETAILS: Sensitive files (user.txt) accessible from web directory.
   FIX: Restrict access permissions and separate web root from sensitive data.

----------------------------------------------------------------------------------------------------------------------
[+] CONCLUSION
----------------------------------------------------------------------------------------------------------------------
The target machine was successfully compromised through a file upload vulnerability, leading to remote code execution. 
Privilege escalation was achieved via misconfigured SUID binaries, resulting in full root access.

----------------------------------------------------------------------------------------------------------------------
[!] END OF DOCUMENT - makarov_XSec - CONFIDENTIAL