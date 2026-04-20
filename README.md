# Lian_Yu TryHackMe Writeup

---

## 1. Initial Target (Before Switching to Kali Linux)



- Initial IP address obtained from TryHackMe: `10.48.176.202`
- This IP was used during the initial enumeration phase. 



## 2. Gobuster Scan (First Enumeration)

### Command:
```bash
gobuster dir -u http://10.48.176.202 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```
![](images/1.png)

Explanation:

-	gobuster
→ A tool used for brute-force enumeration of hidden directories and files on a web server. 
- dir
→ Specifies directory enumeration mode, meaning it searches for folders instead of DNS or virtual hosts. 
- -u http://10.48.176.202
→ Defines the target URL to scan. 
- -w /usr/share/wordlists/...
→ Specifies the wordlist file used to guess directory names. 

Function:
- Used to discover hidden directories that are not visible on the main webpage. 

Result:
- /island (accessible)
- /server-status (403 – forbidden)




## 3. Accessing /island Page
- URL: http://10.48.176.202/island
 
![](images/2.png)

Explanation:
- The page displays a message with a hidden clue:
- “The Code Word is:”
- This indicates that further enumeration is required.


## 4. Gobuster Scan (Second Level)

### Command:
```bash
gobuster dir -u http://10.48.176.202/island -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```
![](images/3.png)

Explanation:
- Same command as before but targeting /island.
  
Function:
- To find subdirectories inside /island.

Result:
- /2100 
 
## 5. Accessing /2100 Page
- URL: http://10.48.176.202/island/2100

 ![](images/4.png)

Explanation:
- The page appears mostly empty but contains hints.
- Suggests checking the page source.

 
## 6. Viewing Page Source
Action:
- Right-click → “View Page Source” 

![](images/5.png)

Explanation:
- Reveals hidden HTML comment:
you can avail your ticket here but how?

Function:
- Indicates the presence of a hidden file with extension .ticket.


## 7. Gobuster with File Extension
### Command:
```bash
gobuster dir -u http://10.48.176.202/island/2100 -x ticket -w
/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```
![](images/6.png)

Explanation:
- -x ticket
        → Instructs Gobuster to search for files ending with .ticket.
   
Function:
- Helps identify hidden files with specific extensions.
  
Result:
- green_arrow.ticket



## 8. Accessing Ticket File
URL: http://10.48.176.202/island/2100/green_arrow.ticket

![](images/7.png)

Explanation:
- The file contains an encoded string:  RTy8yHQdsxC
 


## 9. Decoding Using CyberChef
![](images/8.png)

Explanation:
- The encoded string is decoded using Base58 decoding.
  
Function:
- Converts encoded data into readable format.

Result:
- !#th3h00d →  This is used as a password.


## 10. Code Word Discovery
![](images/9.png)
- From the earlier page:  vigilante
- Used as username later.



## 11. OpenVPN Connection (Kali Linux)
### Command: 
```bash
sudo openvpn Lian_Yu.ovpn
```
![](images/10.png)
![](images/11.png)

Explanation:
- sudo
→ Runs the command with superuser (root) privileges, required to modify network settings.
- openvpn
→ A VPN client used to establish a secure connection to the TryHackMe network. 
- Lian_Yu.ovpn
→ Configuration file that contains the VPN connection details. 

Function:
- Establishes a secure tunnel between Kali Linux and the TryHackMe lab environment.

Additional Explanation (IP Change):
- Initially, the target IP address used was:
`10.48.176.202`
- After switching to Kali Linux and reconnecting using OpenVPN, a new IP address was assigned:
`10.49.157.190`
- This happens because the VPN session was restarted and the machine was redeployed. 
- Therefore, all subsequent steps were performed using the updated IP address.



## 12. FTP Connection
### Command: 
```bash
ftp 10.49.157.190
```
![](images/12.png)

Explanation:
- ftp
→ A command used to connect to a remote server using the File Transfer Protocol (FTP).
- `10.49.157.190`
→ The target IP address obtained after reconnecting VPN in Kali Linux.
Output Explanation:
- ftp> o
→ o stands for open & Used to initiate a connection to a remote FTP server.
- (to) 10.49.157.190
→ Indicates the destination IP address that the FTP client is trying to connect to.

Function:
- Establishes a connection to the remote FTP server
- Allows the user to access, view, and download files from the target machine
  
Additional Note:
- The connection is performed using the updated IP address (10.49.157.190) after switching to Kali Linux and reconnecting the VPN.
- The previous IP is no longer valid, therefore all interactions must use the new IP.


## 13. Downloading Files from FTP
### Command:
```bash
get Leave_me_alone.png
get Queen's_Gambit.png
get aa.jpg
```
![](images/13.png)

Explanation:
- get
     → Downloads files from FTP server to local machine. 

Function:
- Retrieve files for further analysis.


## 14. Steghide Extraction
### Command: 
```bash
steghide extract -sf aa.jpg
```
![](images/14.png)

Explanation:
- steghide
→ A tool used for steganography (hiding data in files). 
- extract
→ Extract hidden data. 
- -sf aa.jpg
→ Specifies the source file. 

Function:
- Extract hidden content embedded inside the image.
 
Result:
- ss.zip


## 15. Unzip Extracted File
### Command:
```bash
unzip ss.zip
```
![](images/15.png)

Function:
- Extract compressed files.
  
Result:
- passwd.txt
- shado
 


## 16. Reading Files
### Command:'
```bash
cat passwd.txt
cat shado
```
![](images/16.png)
![](images/17.png)

Explanation:
- cat
→ Displays file content. 

Result:
- SSH password: M3tahuman


## 17. SSH Login
### Command: 
```bash
ssh slade@10.49.157.190
```
![](images/18.png)

Explanation:
- ssh
→ Secure Shell protocol for remote login.
- slade@IP
→ Username and target machine. 

Function:
- Gain remote shell access. 


## 18. Retrieve User Flag
### Command: 
```bash
cat user.txt
```
![](images/19.png)
Function:
- Display user flag.


## 19. Privilege Escalation Check
### Command: 
```bash
sudo -l
```
![](images/20.png)

Explanation:
- Lists commands that can be run with sudo privileges. 

Result:
- `/usr/bin/pkexec` allowed 
 

## 20. Root Access
### Command: 
```bash
sudo /usr/bin/pkexec /bin/sh
```
![](images/21.png)

Explanation:
- Executes a shell with root privileges. 

Function:
- Escalate privileges to root user.
 

## 21. Retrieve Root Flag
### Command: 
```bash
cat root.txt
```
![](images/22.png)
![](images/23.png)

Function:
- Display final flag.

---

## 👩‍💻 Author

**Lyana Yasmin**




 
