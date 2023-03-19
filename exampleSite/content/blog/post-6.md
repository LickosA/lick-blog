---
title: DoS mitigation with Fail2ban
description: meta description
image: images/post/post-6.png
date: 2022-03-12T23:00:00+00:00
categories:
- tutorials
type: featured

---
### What is Denial of Service (DoS)?

A Denial-of-Service (DoS) attack is an attack meant to shut down a machine or network, making it inaccessible to its intended users. DoS attacks accomplish this by flooding the target with traffic, or sending it information that triggers a crash. In both instances, the DoS attack deprives legitimate users (i.e. employees, members, or account holders) of the service or resource they expected.

There are two general methods of DoS attacks: flooding services or crashing services. Flood attacks occur when the system receives too much traffic for the server to buffer, causing them to slow down and eventually stop. Popular flood attacks include:Buffer overflow attacks,ICMP flood and SYN flood. Other DoS attacks simply exploit vulnerabilities that cause the target system or service to crash. In these attacks, input is sent that takes advantage of bugs in the target that subsequently crash or severely destabilize the system, so that it can’t be accessed or used.

An additional type of DoS attack is the Distributed Denial of Service (DDoS) attack. A DDoS attack occurs when multiple systems orchestrate a synchronized DoS attack to a single target. The essential difference is that instead of being attacked from one location, the target is attacked from many locations at once.

### How To Mitigate a DOS Attack?

There is no way to completely avoid becoming a target of a DoS or DDoS attack. But there are proactive steps that administrators can take to reduce the effects of this attack :

* Enroll in a DoS protection service that detects abnormal traffic flows and redirects traffic away from your network. The DoS traffic is filtered out, and clean traffic is passed on to your network.
* Create a disaster recovery plan to ensure successful and efficient communication, mitigation, and recovery in the event of an attack.

It is also important to take steps to strengthen the security posture of all of your internet-connected devices in order to prevent them from being compromised.

* Install and maintain antivirus software.
* Install a firewall and configure it to restrict traffic coming into and leaving your computer.
* Evaluate security settings and follow good security practices in order to minimalize the access other people have to your information, as well as manage unwanted traffic.

### What is Fail2ban?

Fail2Ban is an intrusion prevention system written in the Python language used to block malicious IPs that are trying to breach your system security. It works by scanning various log files and blocking the IPs that are trying to make frequent login attempts for a specified bantime. It also allows you to monitor the strength and frequency of attacks. Due to its simplicity, it is considered the preferred software to secure your server from DOS, DDOS, and brute-force attacks.

_In this post, we will describe how to secure an SSH and Apache server with Fail2Ban on ubuntu 18.04._

### **How to install fail2Ban for securing SSH and Apache Server?**

#### **Pre requirements**

Before installing fail2ban, make sure that your web and ssh server are properly installed and configured.

#### **Installation of Fail2ban?**

To install fail2ban, execute the commands bellow :

    ```bash
    apt-get update &amp;&amp; apt-get upgrade -y # updating system
    apt-get install fail2ban # installing fail2ban
    systemctl start fail2ban 
    systemctl enable fail2ban
    ```

#### **Secure SSH with Fail2Ban**

##### **Configure Fail2Ban for SSH**

By default, all pre-set jails are located inside /etc/fail2ban/jail.conf file. This is not an appropriate way to edit the default jail.conf file. You should create a separate jail.local file for each service that you want to configure.

You can create a jail.local file for SSH with the following command:

    nano /etc/fail2ban/jail.local

| --- |
| nano /etc/fail2ban/jail.local |

Add the following lines:

| --- |
| \[DEFAULT\]ignoreip = your-server-ipbantime = 300findtime = 300maxretry = 3banaction = iptables-multiportbackend = systemd \[sshd\]enabled = true |

Save and close the file when you are finished. Then, restart the Fail2Ban service to apply the changes:

| --- |
| systemctl restart fail2ban |

You can now check the status of SSH jail with the following command:

| --- |
| fail2ban-client status |

You should see that an SSH jail is enabled:

| --- |
| Status\|- Number of jail: 1\`- Jail list: sshd |

A brief explanation of each parameter is shown below:

* ignoreip: Used to define the IP addresses that you want to be ignored.
* bantime: Used to define a number of seconds the IP address will be banned for.
* findtime: Used to define the amount of time between login attempts before the IP is banned.
* maxretry: Used to define the number of attempts to be made before the IP address is banned.
* banaction: Banning action.
* enabled: This option enables the protection for SSH service.

##### **Test SSH Against Password Attacks**

At this point, Fail2Ban is installed and configured. It’s time to test whether it is working or not.

To do so, go to the remote machine and try to SSH to the server IP address:

| --- |
| ssh root@server-ip |

You will be asked to provide the root password. Type the wrong password again and again. Once you reach the login attempt limit, your IP address will be blocked.

You can verify your blocked IP address with the following command:

| --- |
| fail2ban-client status sshd |

You should see your blocked IP in the following output:

| --- |
| Status for the jail: sshd\|- Filter\| \|- Currently failed: 7\| \|- Total failed: 39\| \`- Journal matches: _SYSTEMD_UNIT=sshd.service + _COMM=sshd\`- Actions \|- Currently banned: 1 \|- Total banned: 2 \`- Banned IP list: 190.8.80.42 |

You can also check the SSH log for failed logins:

| --- |
| tail -5 /var/log/secure \| grep 'Failed password' |

You should see the following output:

| --- |
| Mar 1 03:55:03 centos8 sshd\[11196\]: Failed password for invalid user bpadmin from 190.8.80.42 port 55738 ssh2 |

You can also block and unblock a specific IP address manually.

For example, to unblock the IP 190.8.80.42, run the following command:

| --- |
| fail2ban-client set sshd unbanip 190.8.80.42 |

To block the IP **190.8.80.42**, run the following command:

| --- |
| fail2ban-client set sshd banip 190.8.80.42 |

#### **Secure Apache with Fail2Ban**

##### **Configure Fail2Ban for Apache**

You can also secure the Apache web server from different kinds of attacks. You will need to configure jail.local file for Apache as shown below:

| --- |
| nano /etc/fail2ban/jail.local |

Add the following lines at the end of the file:

| --- |
| \[apache-auth\]enabled = trueport = http,httpslogpath = %(apache_error_log)s \[apache-badbots\]enabled = trueport = http,httpslogpath = %(apache_access_log)sbantime = 48hmaxretry = 1 \[apache-noscript\]enabled = trueport = http,httpslogpath = %(apache_error_log)s |

Save and close the file when you are finished. Then, restart the Fail2Ban service to implement the changes:

| --- |
| systemctl restart fail2ban |

You can now verify the status of all jails with the following command:

| --- |
| fail2ban-client status |

You should see the following output:

| --- |
| Status\|- Number of jail: 5\`- Jail list: apache-auth, apache-badbots, apache-noscript, sshd |

A brief explanation of each jail is shown below:

* apache-auth: This jail is used to protect Apache from failed login attempts.
* apache-badbots: This jail is used to ban hosts which agent identifies spammer robots crawling the web for email addresses.
* apache-noscript: Used to block the IP which is trying to search for scripts on the website to execute.

##### **Test Apache Against Dos Attacks**

We will test DOS attack using slowloris. Slowloris is very common tool used in DOS attacks.