# MMA Leathernecks Team Linux Checklist
## https://cpxvii.s3.amazonaws.com/cp17_tr/CP17_Ubuntu22_Training_Answer_Key.pdf
## Note

Assume root permissions are needed for most commands. You can use `sudo` or become root with `su`.

I would no longer recommend running `apt-get dist-upgrade` in competition. They may call it intelligent, but it has a bad track record of breaking critical services.

This script heavily borrows from [Forty-Bot Linux Checklist](https://github.com/Forty-Bot/linux-checklist)

## Checklist

1. Read the readme

	Take notes on neccessary services, users, and any other important information.

1. Do the Forensics Questions

	Forensics questions can point you towards other vulnerabilities. Keep this in mind. (ex: a media file, find a hidden message, find a backdoor, etc)
	1. Locate files
    		`locate *.mp3`
    		`find -size 12934c (c is for bytes`
	1. Find Machine ID
        	`hostnamctl`
1. Install clamtk
	`apt-get install clamtk`
	Run the scan
	`freshclam`

1. Install required services found in README.md

1. Account Configuration
	1. Secure the /etc/shadow file
              `chmod 640 /etc/shadow`
	1. Lock the root account

		`$ passwd -l root`
		
	1. set `PermitRootLogin no` in `/etc/ssh/sshd_config`

	1. Disable the guest account in `/etc/lightdm/lightdm.conf`

		```
		allow-guest=false
		greeter-hide-users=true
		greeter-show-manual-login=true
		autologin-user=none
		```

	1. Compare `/etc/passwd` and `/etc/group` to the readme

		Look out for uid 0 and hidden users!
	   	Look for any repeating UID or GID
		Make sure no programs have a /bin/sh or /bin/bash
		Only root should have a UID and GID of 0


	1. Delete unauthorized users

		```
		$ userdel -r $user
		$ groupdel $user
		```

	1. Add users

		```
		$ useradd -G $group1,$group2 $user
		$ passwd $user
		```

	1. Remove unauthorized users from adm and groups

		`$ gpasswd -d $user $group`

	1. Change unsecure password for users

		`passwd $user`

	1. Add authorized users to groups

		`$ gpasswd -a $user $group`

	1. Check `/etc/sudoers` and `/etc/sudoers.d` for unauthorized users and groups.

		1. Remove any instances of `nopasswd` and `!authenticate`, these allow sudo use without authentication

		1. Any commands listed can be run without a password (ex: /bin/chmod)

		1. Group lines are preceded by `%`

	1. Wait to change user passwords until after password policy!


1. Password Policy

	1. Change password expiration requirements in `/etc/login.defs`

		```
		FAILLOG_ENAB YES
		LOG_UNKFAIL_ENAB YES
		SYSLOG_SU_ENAB YES
		SYSLOG_SG_ENAB YES
		PASS_MAX_DAYS 30
		PASS_MIN_DAYS 7
		PASS_WARN_AGE 12
		```

	1. Add password history, minimum password length, and password complexity requirements in `/etc/pam.d/common-password`
		**PASSWORD LENGTH**
		1. The file that controls password complexity is:
     			**INSTALL CRACKLIB PRIOR TO CHANGING COMMON-PASSWORD**

			`$ apt-get install libpam-cracklib`
	
			```
			password	required	pam_unix.so obscure sha512 remember=12 use_authtok
			password	required	pam_cracklib.so retry=3 minlen=13 difok=4 dcredit=-1 ucredit=-1 ocredit=-1 lcredit=-1 maxrepeat=3
			```

			`/etc/pam.d/common-password`

			1. There is a line:

			`password [success=1 default=ignore] pam_unix.so obscure sha512`

			1. Which defines the basic rules for password complexity. You can add a minimum length override by changing it to:

			`password [success=1 default=ignore] pam_unix.so obscure sha512 minlen=12`

		1. REMOVE nullok after pam_unix.so since Null passwords do not authenticate:
			
			1. Go to `sudo gedit /etc/pam.d/common-auth`
			
			1. Remove nullok 

			`auth [success=2 default=ignore]pam_unix.so nullok`

			1. It should look like this:
			
			`auth [success=2 default=ignore]pam_unix.so`

	1. Enforce account lockout policy in `/etc/pam.d/common-auth`

		**MUST COME FIRST**

		`auth	required	pam_tally2.so deny=5 audit unlock_time=1800 onerr=fail even_deny_root`

		In a terminal, type `sudo touch /usr/share/pam-configs/faillock`, then sudo nano /usr/share/pam-configs/faillock`

		```
  		Name: Enforce failed login attempt counter
		Default: no
		Priority: 0
		Auth-Type: Primary
		Auth:
		 [default=die] pam_faillock.so authfail
		 sufficient pam_faillock.so authsucc
  		```
  
		type `sudo touch /usr/share/pam-configs/faillock_notify`, then `sudo nano /usr/share/pam-configs/faillock_notify`

		```
		Name: Notify on failed login attempts
		Default: no
		Priority: 1024
		Auth-Type: Primary
		Auth:
		 requisite pam_faillock.so preauth
  		```
  
		Type `sudo pam-auth-update`.
		Select, with the spacebar, Notify on failed login attempts, and Enforce failed login attempt counter, and then select
		`<Ok>.`
	1. Change account expiry defaults in `/etc/default/useradd`

		```
		EXPIRE=30
		INACTIVE=30
		```

	1. Check minimum and maximum password ages in `/etc/shadow`

		Use `chage` to change password expiration.

		`$ chage -m $MIN -M $MAX $user`

	1. **CHANGE PASSWORDS---YOU WILL BE LOCKED OUT IF YOU DON'T!**

		Be sure to record new user passwords!

		`$ passwd $user`


1. Check for unauthorized media

	1. Find media files

		`$ find / -iname "*.$extension"`

	1. Look through user home directories for any unauthorized media

		`$ ls -alR /home`

		**There also may be unauthorized network shares not under the /home directory**

1. Network Security

	1. Enable and configure UFW

		```
		$ ufw default deny incoming
		$ ufw default allow outgoing
		$ ufw allow $port/service
		$ ufw delete $rule
		$ ufw logging on
		$ ufw logging high
		$ ufw enable
		```

	1. Check `/etc/hosts` file for suspicious entries

	1. Prevent IP Spoofing

		`$ echo "nospoof on" >> /etc/host.conf`

1. Package Management

	1. Verify the repositories listed in `/etc/apt/sources.list`

	1. Verify Repositories

		1. Check apt repository policy

			`$ apt-cache policy`

		1. Check apt trusted keys

			`$ apt-key list`

	1. Updates

		```
		$ apt-get update
		$ apt-get -y upgrade
		$ apt-get -y dist-upgrade
		```

	1. Enable automatic updates
		1. Enable automatic updates: Click the Show Applications button on the bottom of the Launcher and click Software & Updates
(you may select Settings on the pop-up box). In the Updates tab, select the dropdown box next
to Automatically check for updates, and choose Daily. If prompted type the password of your
current user account. The password for your current user account can be found in the README.
Click Close (or Cancel if prompted to apply updates)

		1. Install `unattended-upgrades`

			`$ apt-get install unattended-upgrades`

		1. Reconfigure `unattended-upgrades`

			`$ dpkg-reconfigure unattended-upgrades`

		1. Edit `/etc/apt/apt.conf.d/20auto-upgrades`

			```
			APT::Periodic::Update-Package-Lists "1";
			APT::Periodic::Download-Upgradeable-Packages "1";
			APT::Periodic::AutocleanInterval "7";
			APT::Periodic::Unattended-Upgrade "1";
			```

		1. Edit `/etc/apt/apt.conf.d/50auto-upgrades`

			```
			Unattended-Upgrade::Allowed-Origins {
				"${distro_id} stable";
				"${distro_id} ${distro_codename}-security";
				"${distro_id} ${distro_codename}-updates";
			};

			Unattended-Upgrade::Package-Blacklist {
				"libproxy1v5";		# since the school filter blocks the word proxy
			};
			```

		**Look for points for packages mentioned in the README, along with bash (if vulnerable to Shellshock), the kernel, sudo, and sshd**

	1. Verify binaries match with `debsums`

		1. Install `debsums`

			`$ apt-get install debsums`

		1. Generate checksums for packages that don't come with them

			`$ debsums -g`

		1. Verify checksums for all binaries

			`$ debsums -c`

		1. Verify checksums for binaries and config files *(false positives for legitimate changes by us)*

			`$ debsums -a`

	1. Remove unauthorized and unused packages

		1. Use deborphan to detect unneccessary packages

			1. Install deborphan

				`$ apt-get install deborphan`

			1. Search for unneccessary packages

				`$ deborphan --guess-all`

			1. Delete unneccessary data packages

				`$ deborphan --guess-data | xargs sudo apt-get -y remove --purge`

			1. Delete unneccessary libraries

				`$ deborphan | xargs sudo apt-get -y remove --purge`

		1. Look for hacking tools, games, and other unwanted/unneccessary packages

			```
			$ apt-cache policy $package
			$ which $package
			$ dpkg-query -l | grep -E '^ii' | less
			```
			**OR**
			
			```
			systemctl list-units --type=service --state=active
			sudo systemctl stop nginx
			sudo systemctl disable nginx
			```

		1. Ensure all services are required

			`service --status-all`

		REMOVE ANY GAMES NOT SPECIFIED

		BAD STUFF

		`john, nmap, vuze, frostwire, kismet, freeciv, minetest, minetest-server, medusa, hydra, truecrack, ophcrack, nikto, cryptcat, nc, netcat, tightvncserver, x11vnc, nfs, xinetd`

		POSSIBLY BAD STUFF

		`samba, postgresql, sftpd, vsftpd, apache, ftp, mysql, php, snmp, pop3, icmp, sendmail/postfix, dovecot, bind9, nginx, AisleRiot, manaplus, JTR,`

		MEGA BAD STUFF

		`telnet, rlogind, rshd, rcmd, rexecd, rbootd, rquotad, rstatd, rusersd, rwalld, rexd, fingerd, tftpd, telnet, snmp, netcat, nc, nginx,apache2`

  		IF FTP REQUIRED INSTALLED SECURE IT:
		`sudo nano /etc/vsftpd.conf`

		`
		anonymous_enable=ON
		local_enable=YES
		write_enable=YES
		chroot_local_user=YES
		`

1. Service & Application Hardening

	1. Configure OpenSSH Server in `/etc/ssh/sshd_config`

		```
		Protocol 2
		LogLevel VERBOSE
		X11Forwarding no
		MaxAuthTries 4
		IgnoreRhosts yes
		HostbasedAuthentication no
		PermitRootLogin no
		PermitEmptyPasswords no
		```

	1. Harden Firefox

		1. Block Popups

	1. Configure apache2 in `/etc/apache2/apache2.conf`

		```
		ServerSignature Off
		ServerTokens Prod
		```

1. Backdoor Detection and Removal

	1. `ss -ln`

	1. If a port has `127.0.0.1:$port` in its line, that means it's connected to loopback and isn't exposed. Otherwise, there should only be ports which are specified in the readme open (but there probably will be tons more).

	1. For each open port which should be closed

		1. Find the program using the port

			`$ lsof -i $port`

		1. Locate where the program is running from

			`$ whereis $program`

		1. Find what package owns the file

			`$ dpkg -S $location`

		1. Remove the responsible package

			`$ apt-get purge $package`

		1. If there is no package, delete the file and kill the processes

			`$ rm $location; killall -9 $program`

		1. Verify the port is closed

			`$ ss -l`

1. Cron

	1. Check your user's crontabs

		`$ crontab -e`

	1. Check `/etc/cron.*/`, `/etc/crontab`, and `/var/spool/cron/crontabs/`

	1. Check init files in `/etc/init/` and `/etc/init.d/`

	1. Remove contents of `/etc/rc.local`

		`$ echo "exit 0" > /etc/rc.local`

	1. Check user crontabs

		`$ crontab -u $user -l`

	1. Deny users use of cron jobs

		`$ echo "ALL" >> /etc/cron.deny`
1. Kernel Securing
   
	`Sysctl -p`

   	1. Add this to the bottom of the `/etc/sysctl.conf` file

   		1. Disable ICMP redirects
   	 
			`1.	net.ipv4.conf.all.accept_redirects = 0`

   		1. Disable IP redirecting
   	    
			```
   			net.ipv4.ip_forward = 0
   	 		net.ipv4.tcp_syncookies=1
			net.ipv4.conf.all.send_redirects = 0
			net.ipv4.conf.default.send_redirects = 0
			```
   
   		1. Disable IP spoofing
   	    
			`net.ipv4.conf.all.rp_filter=1`

   		1. Disable IP source routing
   	    
			`net.ipv4.conf.all.accept_source_route=0`

   		1. SYN Flood Protection
   	    
			```
   			net.ipv4.tcp_max_syn_backlog = 2048
			net.ipv4.tcp_synack_retries = 2
			net.ipv4.tcp_syn_retries = 5
			net.ipv4.tcp_syncookies = 1
			```
   
   		1. Disable IPV6
   	    
			```
			net.ipv6.conf.all.disable_ipv6 = 1
			net.ipv6.conf.default.disable_ipv6
			net.ipv6.conf.lo.disable_ipv6
			```
   		1. APPLY CHANGES **IMPORTANT**
			 `sudo sysctl --system`

1. Kernel Debugging
   	1.  The file: `/etc/sysctl.conf` should have `kernel.sysrq = 0`
1. Kernel Hardening

	1. Edit the `/etc/sysctl.conf` file

		```
		fs.file-max = 65535
		fs.protected_fifos = 2
		fs.protected_regular = 2
		fs.suid_dumpable = 0
		kernel.core_uses_pid = 1
		kernel.dmesg_restrict = 1
		kernel.exec-shield = 1
		kernel.sysrq = 0
		kernel.randomize_va_space = 2
		kernel.pid_max = 65536
		net.core.rmem_max = 8388608
		net.core.wmem_max = 8388608
		net.core.netdev_max_backlog = 5000
		net.ipv4.tcp_rmem = 10240 87380 12582912
		net.ipv4.tcp_window_scaling = 1
		net.ipv4.tcp_wmem = 10240 87380 12582912
		net.ipv4.conf.all.accept_redirects = 0
		net.ipv4.conf.all.accept_source_route = 0
		net.ipv4.conf.all.log_martians = 1
		net.ipv4.conf.all.redirects = 0
		net.ipv4.conf.all.rp_filter = 1
		net.ipv4.conf.all.secure_redirects = 0
		net.ipv4.conf.all.send_redirects = 0
		net.ipv4.conf.default.accept_redirects = 0
		net.ipv4.conf.default.accept_source_route = 0
		net.ipv4.conf.default.log_martians = 1
		net.ipv4.conf.default.rp_filter = 1
		net.ipv4.conf.default.secure_redirects = 0
		net.ipv4.conf.default.send_redirects = 0
		net.ipv4.icmp_echo_ignore_all = 1
		net.ipv4.icmp_echo_ignore_broadcasts = 1
		net.ipv4.icmp_ignore_bogus_error_responses = 1
		net.ipv4.ip_forward = 0
		net.ipv4.ip_local_port_range = 2000 65000
		net.ipv4.tcp_max_syn_backlog = 2048
		net.ipv4.tcp_synack_retries = 2
		net.ipv4.tcp_syncookies = 1
		net.ipv4.tcp_syn_retries = 5
		net.ipv4.tcp_timestamps = 9

		# Disable IPv6
		net.ipv6.conf.all.disable_ipv6 = 1
		net.ipv6.conf.default.disable_ipv6 = 1
		net.ipv6.conf.lo.disable_ipv6 = 1

		# Incase IPv6 is necessary
		net.ipv6.conf.default.router_solicitations = 0
		net.ipv6.conf.default.accept_ra_rtr_pref = 0
		net.ipv6.conf.default.accept_ra_pinfo = 0
		net.ipv6.conf.default.accept_ra_defrtr = 0
		net.ipv6.conf.default.autoconf = 0
		net.ipv6.conf.default.dad_transmits = 0
		net.ipv6.conf.default.max_addresses = 1
		```

	1. Load new sysctl settings

		`$ sysctl -p`

1. Antivirus

	1. Install `clamav`, `chkrootkit`, and `rkhunter`

		`$ apt-get install clamav chkrootkit rkhunter`

	1. Run ClamAV

		```
		$ freshclam
		$ freshclam --help
		```

	1. Run chkrootkit

		`$ chkrootkit -l`

	1. Run RKHunter

		```
		$ rkhunter --update
		$ rkhunter --propupd
		$ rkhunter -c --enable all --disable none
		```

	1. Look through `/var/log/rkhunter.log`

1. Audit the System with Lynis

	1. Install

		```
		$ cd /usr/local
		$ git clone https://github.com/CISOfy/lynis
		$ chown -R 0:0 /usr/local/lynis
		```

	1. Audit the system with Lynis

		```
		$ cd /usr/local/lynis
		$ lynis audit system
		```

	1. Look through `/var/log/lynis-report.dat` for warnings and suggestions

		`$ grep -E 'warning|suggestion' | sed -e 's/warning\[\]\=//g' | sed -e 's/suggestion\[\]\=//g'`

1. Configure Auditd

	1. Install

		`$ apt-get install auditd`

	1. Enable

		`$ auditctl -e 1`

	1. Configure with `/etc/audit/auditd.conf`
    
1. Check cronjobs
   
	1. Check these folders

	```
	/etc/cron.*
	/etc/crontab
	/var/spool/cron/crontabs
 	```
 
	1. Check the init files

	```
	/etc/init
	/etc/init.d
 	```
 
	1. Check for each user
    
	`crontab –u {USER} -l`

1.  Check the runlevels if unable to boot into GUI
   
	1. To check the run level
    
	`runlevel`

	1. Runlevels
    

	```
 	0-System halt;No activity
	1-Single user
	2-Multi-user, no filesystem
	3-Multi-user, commandline only
	4-user defineable
	5-multi-users,GUI
	6-Reboot
 	```
 
	1. To change the run level
	 `Telinit {level}`
1.  APACHE
	1. Hide Apache Version number.
    
		Add the following lines to the bottom of /etc/apache2/apache2.conf
	
		```
	 	ServerSignature Off
	 	ServerTokens Prod
	 	```
 
 	1. Make sure Apache is running under its own user account and group.
     
		Add a separate user “apache”

		Edit the `/etc/apache2/apache2.conf` file

		```
		User apache
		Group apache
  		```
  
	1. Ensure that file outside the web root directory are not accessed. /etc/apache2/apache2.conf

		```
		<Directory />
		Order Deny,Allow
		Dent from all
		Options -Indexes
		AllowOverride None
		</Directory>
		<Directory /html>
		Order Allow,Deny
		Allow from all
		</Directory>
  		```
	1. Turn off directory browsing, Follow symbolic links and CGI execution
    
		Add Options `None` to a `<Directory /html>` tag

	1. Install modsecurity
    
    		```
    		apt-get install mod_security
    		service httpd restart
    		```
  		
	1. Lower the Timeout value in `/etc/apache2/apache2.conf`
    
		`Timeout 45`

1.  MySQL
	1. Restrict remote MySQL access
    
		Edit `/etc/mysql/my.cnf`

		`Bind-address=127.0.0.1`

	1. Disable use of LOCAL INFILE
		Edit `/etc/mysql/my.cnf`

		```
		[mysqld]
	   	local-infile=0
  		```
  
	1. Create Application Specific user

		```
		root@Ubuntu:~# mysql –u root –p
		mysql> CREATE USER ‘myusr’@’localhost’ IDENTIFIED BY ‘password’;
		mysql> GRANT SELECT,INSERT,UPDATE,DELETE ON mydb.* TO ‘myusr’@’localhost’ IDENTIFIED BY ‘password’;
		mysql> FLUSH PRIVILEGES;
  		```
  
	1. Improve Security with mysql_secure-installation
  
    		```
    		root@Ubuntu:~# mysql_secure_installation
    		change the root password?: y
    		Remove anonymous users?: y
    		Disallow root login remotely?: y
    		Remove test database and access to it?: y
    		Reload privilege tables now?: y
		```

1.  PHP
	1. Restrict PHP Information Leakage
 
		Edit `/etc/php5/apaceh2/php.ini`

		`expose_php = off`

	1. Disable Remote Code Execution
		Edit `/etc/php5/apache2/php.ini`

		```
		allow_url_fopen=Off
		allow_url_include=Off
  		```

	1. Disable dangerous PHP Functions

		Edit `/etc/php5/apache2/php.ini`

		`disable_functions=exec,shell_exec,passthru,system,popen,curl_exec,curl_multi_exec,parse_ini_file,show_source,proc_open,pcntl_exec`

	1. Enable Limits in PHP

		Edit `/etc/php5/apache2/php.ini`

		```
		upload_max_filesize = 2M
	   	max_execution_time = 30
		max_input_time = 60
  		```

## Other Checklists

[SANS Hardening the Linux System](https://www.sans.org/media/score/checklists/LinuxCheatsheet_2.pdf)

[Awesome Security Hardening](https://github.com/decalage2/awesome-security-hardening)
