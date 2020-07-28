# Blue Team: Summary of Operations

## Table of Contents
- Network Topology
- Description of Targets
- Monitoring the Targets
- Patterns of Traffic & Behavior
- Suggestions for Going Further

### Network Topology
_TODO: link map_

The following machines were identified on the network:
- Target 1
  - Linux
  - Hosting Wordpress Site - Vulnerable
  - 192.168.1.110
- Target 2
  - Linux
  - Hosting Wordpress Site - Less Vulnerable
  - 192.168.1.115
- ELK Stack
  - Linux
  - Hosts ELK SEIM and Kibana Site
  - 192.168.1.100
- Capstone VM
  - Linux
  - Hosts Company Site and WebDAV (Project 2)
  - 192.168.1.105
- Kali
  - Linux
  - "Malicious" Machine - Enumeration and attack of othe rmachines done on this machine
  - 192.168.1.90
- Windows RDP
  - Windows
  - Hosts the virtual network and is the domain controller
  - 192.168.1.1

### Description of Targets

Two VMs on the network were vulnerable to attack: `Target 1` 192.168.1.110 and `Target 2` 192.168.1.115.

Each VM functions as an Apache web server and has SSH enabled, so ports 80 and 22 are possible ports of entry for attackers. As such, the following alerts have been implemented:

### Monitoring the Targets

Traffic to these services should be carefully monitored. To this end, we have implemented the alerts below:

#### CPU Usage Monitor

Alert 1 is implemented as follows:
  - metricbeat
  - Threshold is set to 50% of max CPU resources over 5 minutes
  - Can be utilized to detect machines comprimised with resource sequestering malware.
  - CPU utility is constantly flucuating with application initiation and manipulating large file transfers and computation, so it may be triggered often if deployed across multiple machines.
![CPU Usage Alert](/Images/CPUusagealert)

#### HTTP Request Size Monitor
Alert 2 is implemented as follows:
  - packetbeat
  - Theshold was set to 3500 HTTP requests per minute.
  - Can be utilized to detect DDOS attacks or brute force login attempts.
  - This threshold should be designed to not be triggered by daily common use and therefore shuold change with the upscaling of the website. At the recommended 3500 requests, it is expected common usage of this website should not trigger the alert.
![HTTP Request Alert](/Images/HTTPSizealert)

#### Excessive HTTP Errors
Alert 3 is implemented as follows:
  - packetbeat
  - Threshold was set to alert if the top 5 HTTP response codes detects code 400 responses (client errors)
  - Can be utilized to detects a high influx of client errors, common to brute force attacks and spidering a website.
  - This threshold should not create false triggers unless there are bugs in the webpages
![Top 5 HTTP Errors 400 Alert](/Images/httperrors)

### Suggestions for Going Further

The logs and alerts generated during the assessment suggest that this network is susceptible to several active threats, identified by the alerts above. In addition to watching for occurrences of such threats, the network should be hardened against them. The Blue Team suggests that IT implement the fixes below to protect the network:
- Vulnerability 1
  - The best solution to prevent malware is to have good governance and poliocy to prevent reckless behavior on company resources in addition to keeping machine updated as often as possible. Otherwise, establish firewall rules to block the download of specific file types and to deny outgoing traffic through ports not commonly used.
  - These patches will work because prevention of human error can solve a plethora of preventable issues, and prevention of specific file type downloads and traffic from non-determined ports will stop the download and functionality of malware.
- Vulnerability 2
  - Block high frequency traffic and blacklist IPs that are notable offenders of reconnecting at unusually high rates. These connection specifications can be made on the firewall configuration.
  - DDOS machines will be restricted from initiating connection to the server and will adapt to be IPs attempting to DDOS.
- Vulnerability 3
  - Establish a robust password policy and lockout policy. This can be done on the hosting machine and wordpress configuration files. 
  - Bruteforce attempts will be halted immediately due to the obstacle of a login timeout.
