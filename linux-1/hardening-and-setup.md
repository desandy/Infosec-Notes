---
description: >-
  A collection of useful programs and configurations for getting your home box
  set up for pre-engagement use.
---

# Hardening & Setup

## System update

The first thing to do after the first boot is to update the system. In most Debian-based flavors of Linux, you achieve this by executing the commands below:

```bash
sudo apt update && sudo apt upgrade -y
```

## Manage installed packages

List all packages installed on your Linux OS and remove the unnecessary ones. Besides installing updates, the next best way to harden a system is to remove or disable applications and services that are vulnerable to attack or are not needed. Here’s an example of how to list the packages installed on Kali Linux: `apt-cache pkgnames`

Remember that disabling unnecessary services will reduce the attack surface, so it is important to remove the following legacy services if you found them installed on the Linux server:

* Telnet server
* RSH server
* NIS server
* TFTP server
* TALK server
* Any other running services that are not needed

`yum` for RedHat based systems. `apt` for debian based systems.

```bash
# yum erase xinetd ypserv tftp-server telnet-server rsh-server
# apt --purge remove xinetd nis yp-tools tftpd atftpd tftpd-hpa telnetd rsh-server rsh-redone-server
```

### **Disable Unnecessary Services**

If you don't want to completely uninstall a service, you can simply disable it until it is needed.  A lot of services and daemons are started during system boot and disabling those that are not being used can help with system hardening and can improve boot time. Since most modern distributions use `systemd` instead of init scripts, you can use `systemctl` to list running services.

```bash
sudo systemctl list-unit-files --type=service
sudo systemctl list-dependencies graphical.target
```

These commands will display such service and daemons. You can disable a specific service by using the below commands.

```bash
sudo systemctl disable service
sudo systemctl disable httpd.service
```

## Check for open ports

Identifying open connections to the internet is critical to understanding your attack surface. In Kali Linux, use the following commands to identify open ports:

```bash
netstat -tulpn
ss -tulpn
lsof -i

...add more info
```

## Secure SSH

First of all, if you do not need SSH, disable it. However, if you want to use it, then you need to ensure the configurations of SSH are secure.

1. Browse to `/etc/ssh` and open the `sshd_config` file using your favorite text editor.
2. Change the default port number 22 to something else, e.g. 2299. \(Caution: this could interfere with some tools which expect SSH to run on port 22\).
3. Make sure that root cannot login remotely through SSH by modifying the following line in `sshd_config`.

    `PermitRootLogin no`

4. Allow only specific users:

    `AllowUsers <username>`

5. Add a banner to discourage attackers from continuing further with:

    `Banner /etc/banner` 

6. Check the manual for SSH to understand all the configurations in `/etc/ssh/sshd_config`. Some other examples of recommended configuration options are:

   ```bash
   AuthorizedKeysFile /etc/ssh/authorized-keys/%u #removes this file from user's folder and puts its in more secure /etc folder
   Protocol2
   IgnoreRhosts to yes
   HostbasedAuthentication no
   PermitEmptyPasswords no
   X11Forwarding no
   MaxAuthTries 5
   Ciphers aes128-ctr,aes192-ctr,aes256-ctr
   HostKeyAlgorithms ecdsa-sha2-nistp256,ecdsa-sha2-nistp384,ecdsa-sha2-nistp521,ssh-rsa,ssh-dss
   KexAlgorithms ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,diffie-hellman-group14-sha1,diffie-hellman-group-exchange-sha256
   MACs hmac-sha2-256,hmac-sha2-512,hmac-sha1
   ClientAliveInterval 900 
   ClientAliveCountMax 0
   UsePAM yes
   ```

7. Finally, set the permissions on the sshd\_config file so that only root users can change its contents:
   * `chown root:root /etc/ssh/sshd_config`
   * `chmod 600 /etc/ssh/sshd_config`

## Lock the boot directory

The boot directory contains important files related to the Linux kernel, so you need to make sure that this directory is locked down to read-only permissions by following the next simple steps. 1. First, open the “fstab” file: `vim /etc/fstab` 2. Then, add the following below the last line: `LABEL=/boot /boot ext2 defaults,ro 1 2` 3. When you finish editing the file, you need to set the owner by executing the following command: `chown root:root /etc/fstab` 4. Next, set permissions for securing the boot settings:

* Set the owner and group of `/etc/grub.conf` to the root user: `chown root:root /etc/grub.conf`
* Set permission on the `/etc/grub.conf` file to read and write for root only: `chmod og-rwx /etc/grub.conf`
* Require authentication for single-user mode:
  * `sed -i "/SINGLE/s/sushell/sulogin/" /etc/sysconfig/init`
  * `sed -i "/PROMPT/s/yes/no/" /etc/sysconfig/init`

## Monitoring and Logging

[https://kali.training/topic/monitoring-and-logging/](https://kali.training/topic/monitoring-and-logging/) TODO: add more info tripwire [https://kali.training/topic/exercise-7-3-securing-the-kali-file-system/](https://kali.training/topic/exercise-7-3-securing-the-kali-file-system/) checksecurity chkrootkit/rkhunter

### **Lock Login Attempts after Failure**

Admins should make sure that users can’t log into their server after a certain number of failed attempts. This increases the overall security of the system by mitigating password attacks. You can use the Linux `faillog` command to see the failed login attempts.

```bash
# faillog
# faillog -m 3
# faillog -l 1800
```

The first command will display the failed login attempts for users from the `/var/log/faillog` database. The second command sets the maximum number of allowed failed login attempts to 3. The third one sets a lock of 1800 seconds or 30 minutes after the allowed number of failed login attempts.

```bash
# faillog -r -u <username>
```

Use this command to unlock a user once they’re prohibited from login. The max number of failed login attempts for the root user should be high or else brute force attacks may leave you locked.

### Fail2Ban

[Fail2Ban](https://www.fail2ban.org/) is one of the most popular IPS solutions for Unix-like systems. It is written using Python and is available on all POSIX-compliant platforms. It will look for obtrusive network requests all the time and block them as soon as possible. Install Fail2Ban using the below command.

```bash
apt install -y fail2ban
yum install -y fail2ban
```

[DenyHosts](https://github.com/denyhosts/denyhosts) is another popular IPS solution for Linux hardening. It will protect your ssh servers from intrusive brute force attempts. Use the following commands to install in on your Debian or Centos servers.

```bash
apt install -y denyhosts
yum install -y denyhosts
```

### Monitoring Logs 

 **Logs to review**

| Log File | Description |
| :--- | :--- |
| /var/log/message | whole system logs or current activity logs |
| /var/log/auth.log | Authentication logs |
| /var/log/kern.log | Kernel logs |
| /var/log/cron.log | Crond logs \(cron job\) |
| /var/log/maillog | Mail server logs |
| /var/log/boot.log | System boot log |
| /var/log/mysqld.log | MySQL database server log file |
| /var/log/secure | Authentication log |
| **/var/run/utmp** | complete picture of users logins: at which terminals, logouts, system events and current status of the system, system boot time \(used by uptime\) etc. |
| **/var/log/wtmp** | gives historical data of utmp |
| **/var/log/btmp** | records failed login attempts |

If you want to read the contents of the binary files `wtmp`, `utmp` or `btmp`, use the command:

```bash
sudo utmpdump /var/run/utmp
sudo utmpdump /var/log/wtmp
sudo utmpdump /var/log/btmp
```

`who`, `w`, and `last <username>` will also give you information about users logged into your machine.

### `logcheck`

**Logcheck** is a powerful tool for monitoring system logs and identifying potential security threats.  It works by monitoring log files for changes (every hour by default) and sends log messages it determines to be 'unusual' to the administrator via email for further analysis. The list of monitored files is stored in `/etc/logcheck/logcheck.logfiles`. The default settings work well unless the `/etc/rsyslog.conf` file has undergone significant customization. 

**logcheck** provides three reporting levels:

- **Paranoid**: Highly detailed and best suited for security-critical servers like firewalls.
- **Server** (default mode): Recommended for most servers.
- **Workstation**: Designed for workstations, offering minimal reporting by filtering out more messages.

Regardless of the chosen mode, customizing **logcheck** to exclude unnecessary messages is advised. Due to its complex message selection mechanism, reading `/usr/share/doc/logcheck-database/README.logcheck-database.gz` is essential to understanding logcheck's Rule Classifications and how to ingore messages.

Here are some ways to use **logcheck** to harden a Linux system:

#### **Fine-Tune Log Monitoring**

- Customize the list of monitored log files in `/etc/logcheck/logcheck.logfiles` to include critical logs like:
  - `/var/log/auth.log` (Authentication logs)
  - `/var/log/kern.log` (Kernel logs)
  - `/var/log/secure` (Security-related logs)
  - `/var/log/btmp` (Failed login attempts)
- Regularly review these logs to detect unauthorized access attempts or system anomalies.

#### **Adjust Filtering Rules**

- Modify rules in `/etc/logcheck/ignore.d.{paranoid,server,workstation}/` to exclude unnecessary messages while ensuring security alerts are not ignored.
- Use `/etc/logcheck/violations.d/` to classify security alerts and `/etc/logcheck/violations.ignore.d/` to filter out false positives.

##### Rule Classification

**logcheck** applies rules to categorize messages into the following types:

- **Cracking attempts** → Stored in `/etc/logcheck/cracking.d/`
- **Ignored cracking attempts** → Stored in `/etc/logcheck/cracking.ignore.d/`
- **Security alerts** → Stored in `/etc/logcheck/violations.d/`
- **Ignored security alerts** → Stored in `/etc/logcheck/violations.ignore.d/`
- **System events** → General logs unless explicitly ignored

##### Ignoring Messages

Messages can only be ignored if rules are defined in specific directories:

- A **cracking attempt** or **security alert** (classified under `/etc/logcheck/violations.d/`) can only be ignored by rules in `/etc/logcheck/violations.ignore.d/`.
- **System events** are always reported unless ignored by rules within `/etc/logcheck/ignore.d.{paranoid,server,workstation}/`, and only those matching or exceeding the selected verbosity level are considered.

#### **Enhance Logcheck with SELinux**

- Enable **SELinux** (`sudo setenforce 1`) to enforce strict access control policies.
- Combine **Logcheck** with SELinux logs (`/var/log/audit/audit.log`) to detect unauthorized access attempts.

#### **Automate Log Analysis**

- Schedule **Logcheck** to run more frequently than the default hourly interval by adjusting the cron job (`/etc/cron.d/logcheck`).
- Set up email alerts to notify administrators immediately of critical security events.

#### **Integrate with Other Security Tools**

- Pair **Logcheck** with **Fail2Ban** to automatically block IPs attempting brute-force attacks.
- Use **Auditd** for deeper system auditing and correlate logs with **Logcheck** for enhanced security insights.

## Enable SELinux

Security Enhanced Linux is a Kernel security mechanism for supporting access control security policy. 

#### **Enable SELinux** <a id="7-enable-selinux"></a>

[SELinux](https://en.wikipedia.org/wiki/Security-Enhanced_Linux), short for Security Enhanced Linux, is a security mechanism that implements various methods for access control at the kernel level. It was developed by Red Hat but has been added to many [modern Linux distributions.](https://ubuntupit.com/best-linux-distro-top-recommendation-to-boost-up-your-linux-experience/) You can check whether SELinux is enabled in your system or not by using the below command.

```bash
sudo getenforce
```

If it returns `enforcing` , your system is protected by SELinux. If the result says `permissive` your system has SELinux but it’s not enforced. It will return `disabled` for systems where SELinux is completely disabled. 

You can enforce SELinux by using the below command:

```bash
sudo setenforce 1
```

Or, using a text editor, open the config file `/etc/selinux/config` and make sure that the policy is enforced by changing the line starting with `SELINUX=`.

```bash
# This file controls the state of SELinux on the system.
# SELINUX can take one of these values:
# enforcing - SELinux security policy is enforced.
# permissive - SELinux prints warnings instead of enforcing.
# disabled - No SELinux policy is loaded.

SELINUX=enforcing
SELINUXTYPE=targeted  # Targeted means only selected processes are protected.
```

SELinux has three configuration mode options:

* Disabled: Turned-off
* Permissive: Prints warnings, but does not block
* Enforcing: Policy is enforced


### Visualizing the Configuration
If you need an image representation of the configuration file, a simple screenshot of a terminal displaying the file’s contents would be the most practical. You can generate this by running:


## Permissions and verifications

Setting permissions is one of the most important and critical tasks to achieve the security goal on a Linux host. Set User/Group Owner and Permissions to `root` on `/etc/anacrontab`, `/etc/crontab` and `/etc/cron.*` by executing the following commands: \(as `root`\)

```bash
chown root:root /etc/anacrontab
chmod og-rwx /etc/anacrontab
chown root:root /etc/crontab
chmod og-rwx /etc/crontab
chown root:root /etc/cron.hourly
chmod og-rwx /etc/cron.hourly
chown root:root /etc/cron.daily
chmod og-rwx /etc/cron.daily
chown root:root /etc/cron.weekly
chmod og-rwx /etc/cron.weekly
chown root:root /etc/cron.monthly
chmod og-rwx /etc/cron.monthly
chown root:root /etc/cron.d
chmod og-rwx /etc/cron.d
```

Set the permissions and owner on `/var/spool/cron` for the `root` crontab.

```bash
chown root:root <crontabfile>
chmod og-rwx <crontabfile>
```

Set permissions and owner on `/etc/passwd` file \(this file must be readable by all users\).

```bash
chmod 644 /etc/passwd
chown root:root /etc/passwd
```

Set permissions and owner on the `/etc/group` file \(this file must be readable by all users\).

```bash
chmod 644 /etc/group
chown root:root /etc/group
```

Set permissions and owner on the `/etc/shadow` file.

```bash
chmod 600 /etc/shadow
chown root:root /etc/shadow
```

Set permissions and owner on the `/etc/gshadow` file.

```bash
chmod 600 /etc/gshadow
chown root:root /etc/gshadow
```



## Misc 

TODO: expand this

* Use strong passwords 
* Set up `fail2ban`, which will make it much harder to brute-force passwords over the network by filtering IP addresses that exceed a limit of failed login attempts. `apt install fail2ban` 
* VPN
* NAT
* Firewall Rules `iptables` and `ip6tables` or GUI `fwbuilder` \([https://kali.training/topic/firewall-or-packet-filtering/](https://kali.training/topic/firewall-or-packet-filtering/)\) 
* Check for default credentials in `README.Debian` files of each respective installed package, as well as `docs.kali.org` and `tools.kali.org` to see if installed services need special care to be secured.

### Add kali repository to other distros

 Any extra repositories needs to be placed into their own file in the directory `/etc/apt/sources.list.d/` with files named as such: `/etc/apt/sources.list.d/repo-name.list` \(replacing `repo-name` with the mirror name\).  This may break things over time so be careful.

```bash
deb   http://http.kali.org/kali   kali-rolling   main non-free contrib
<Archive>   <Mirror>                <Branch>         <Component>
```

To add kali's repository to another distro use the line: `deb http://http.kali.org/kali kali-rolling main non-free contrib`\`

## Useful Programs & Configs Setup

[FireJail](https://firejail.wordpress.com/)

[TOR Browser](https://www.torproject.org/)

[gufw](https://costales.github.io/projects/gufw/)

[chroot jail](https://www.geeksforgeeks.org/linux-virtualization-using-chroot-jail/)



 **lynis** - open source security auditing tool. Comes with Kali

```bash
lynis --update
lynis audit system
```

### Bastille

`bastille --assess` runs [Bastille](http://bastille-linux.sourceforge.net/) in [assessment](http://bastille-linux.sourceforge.net/assessment.htm) mode. The utility will perform an assessment of the current system’s security configuration, grading it according to its relative level of security according to a set of weights you supply. You can use Bastille’s set of default weights for each area being tested or supply your own. This option is only available for the Red Hat and SUSE Linux distributions.

`bastille -x` runs Bastille in its default GUI mode using Perl-Tk to provide the user interface, while `bastille -c` runs the program in console mode.

`bastille --log` logs any changes that Bastille would make to the system without actually applying the changes.

Once Bastille has completed its changes, it creates a TODO list in `/var/log/Bastille/TODO` that describes other actions you should perform on the system to further enhance its security.

If you’re not happy with the changes Bastille makes to your computer, you can run `bastille -r` to reverse its changes. This operation is not guaranteed to work perfectly if it has been a long time since you applied your original Bastille changes, or if you have done a lot of extra security configuration in the meantime.

When using ssh and scp you can deploy Bastille using the commands:

```bash
scp /etc/Bastille/config root@$targetHost:/etc/Bastille
ssh root@$targetHost "bastille -b"
```

where _`$targetHost`_ is the hostname or IP address of the computer you want to connect to. These commands copy the Bastille configuration file to the remote computer and then run Bastille on the remote computer in batch mode, which reads the configuration file to determine the changes Bastille should make and applies them to your system. Obviously Bastille needs to be installed on each of the remote computers first.

### Tmux

Tmux can keep alive sessions if you lose ssh sessions etc, can split panes and more:

Config from [ippsec](https://www.youtube.com/watch?v=Lqehvpe_djs).

```bash
#set prefix
set -g prefix C-a
bind C-a send-prefix
unbind C-b

set -g history-limit 100000
set -g allow-rename off

bind-key j command-prompt -p "Join pane from:" "join-pane -s '%%'"
bind-key s command-prompt -p "Send pane to:" "joian-pane -t '%%'"

set-window-option -g mode-keys vi

run-shell /opt/tmux-logging/logging.tmux
```

First press the prefix `ctrl + b`\(default, Ippsec changes it to Ctrl+a\) then release the buttons and press the combination you want.

Create new named session: `tmux new -s [Name]`

Create new window: `prefix + c`

Rename window: `prefix + ,`

Change panes: `prefix + #`

List windows: `prefix + w`

Vertical split: `prefix + %`

Horizontal split: `prefix + "`

Join panes: `prefix + s #`

Zoom in/out to panes: `prefix + z`

Make sub-terminal its own window: `prefix + !`

Enter vim mode: `prefix + ]` -&gt; Search with `?` in vi mode then press `space` to start copying. Press `prefix + ]` to paste

Kill session by tag:`tmux kill-session -t X`

Kill pane: `prefix + &`

#### tmux plugins:

* tmux logging plugin \(get this!!\) can save log of tmux windows
* [better mouse mode](https://github.com/NHDaly/tmux-better-mouse-mode)

### iptables based wireless access point

Here’s a cool and interesting use of iptables. You can turn any computer with a wireless interface into a wireless access point with hostapd. This solution comes from [https://seravo.fi/2014/create-wireless-access-point-hostapd](https://seravo.fi/2014/create-wireless-access-point-hostapd):

```bash
iptables -t nat -F
iptables -F
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT
echo '1' > /proc/sys/net/ipv4/ip_forward
(DNS, dhcp still required)
```

## References

* [https://forums.kali.org/showthread.php?28027-Hardening-Kali-Linux-Tips-and-Tricks](https://forums.kali.org/showthread.php?28027-Hardening-Kali-Linux-Tips-and-Tricks)
* [https://kali.training/lessons/7-securing-and-monitoring-kali/](https://kali.training/lessons/7-securing-and-monitoring-kali/)
* [https://www.tecmint.com/linux-server-hardening-security-tips/](https://www.tecmint.com/linux-server-hardening-security-tips/)  - TODO: pull more info from this source
* [https://www.ssh.com/ssh/sshd\_config/](https://www.ssh.com/ssh/sshd_config/)
* [https://www.pluralsight.com/blog/it-ops/linux-hardening-secure-server-checklist](https://www.pluralsight.com/blog/it-ops/linux-hardening-secure-server-checklist)
* [https://www.ubuntupit.com/best-linux-hardening-security-tips-a-comprehensive-checklist/](https://www.ubuntupit.com/best-linux-hardening-security-tips-a-comprehensive-checklist/)
* [https://www.thegeekdiary.com/what-is-the-purpose-of-utmp-wtmp-and-btmp-files-in-linux/](https://www.thegeekdiary.com/what-is-the-purpose-of-utmp-wtmp-and-btmp-files-in-linux/)

