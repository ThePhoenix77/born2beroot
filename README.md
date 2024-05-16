# Born2beroot

Welcome to the "born2beroot" project! This README.md file provides an overview of the project, its purpose, features, and how to get started with it.

## Table of Contents

- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Usage](#usage)
- [Contributing](#contributing)
- [License](#license)

## Introduction

The "born2beroot" project is an assignment given as part of the School 42 curriculum. It is designed to introduce students to the fundamentals of system administration and server management on a Linux environment.

In this project, we are tasked with setting up and configuring a Virtual Machine (VM) running a minimal installation of a Linux distribution 
(specifically, Debian). We are required to configure various aspects of the system, including user management, network configuration, security measures, and 
system monitoring.


## Prerequisites

  * You need a hypervizor type 2 (VirtualBox) to create your Debian VM. [VirtualBox Download Link](https://www.virtualbox.org//wiki/Downloads)  
  * - Install your Debian OS as ".iso" file. [Debian OS Download Link] (https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/)
    - Scroll to the bottom of the website and click debian-xx.x.x-amd64-netinst.iso
  * Partition your Debian VM and set it up as told by the subject guidelines.

## Installation

  *  Installing Git into your VM:
    
      -  Type: ``` apt-get install git -y```

      -  Type: ```git --version to check the Git Version```
        
  *  Installing and Configuring SSH (Secure Shell Host):
    
      -  Type: ```apt-get install ufw``` to install UFW
        
      -  Type: ```sudo ufw enable``` to enable
        
      -  Type: ```sudo ufw status numbered``` to check the status of UFW
        
      -  Type: ```sudo ufw allow ssh``` to configure the Rules
        
      -  Type: ```sudo ufw allow 4242``` to configure the Port Rules
        
      -  Type: ```sudo ufw status numbered``` to check the status of UFW 4242 Port
        
  *  Installing libpam-pwquality
    
      -  Type: ```sudo apt-get install libpam-pwquality``` to install Password Quality Checking Library

## Usage
 ### Mandatory part:

  *  Changing of the Host and Guest Port to 4242:
    
      -  Click on your Virtual Machine on the VirtusalBox interface and select Settings
        
      -  Click Network then Adapter 1 then Advanced and then click on Port Forwarding
        
      -  Click on the green plus button on the top right to add a new rule
        
      -  Add a new rule with Host and Guest Port to 4242
        
      -  Then head back to your Virtual Machine
        
      -  Type: ```sudo systemctl restart ssh``` to restart your SSH Server
        
      -  Type: ```sudo service sshd status``` to check your SSH Status
        
      -  Open an iTerm and type the following ```ssh your_username@127.0.0.1 -p 4242``` or ```ssh localhost -p 4242```
        
      - In case an error occurs, then type ```rm ~/.ssh/known_hosts``` in your iTerm and then retype ```ssh your_username@127.0.0.1 -p 4242```
    
      - Lastly type ```exit``` to quit your SSH iTerm Connection
        
  *  Setting Password Policy:
    
      - Type: ```sudo vim /etc/pam.d/common-password``` to accesss the password configuration file.
    
      - Find the line with: ```password		requisite		pam_deny.so``` or ```password		requisite		pam_pwquality.so retry=3```
    
      - Add this line: ```minlen=10 ucredit=-1 lcredit=-1 dcredit=-1 maxrepeat=3 reject_username difok=7 enforce_for_root```
    
      - Save and Exit Vim
    
      - Type in your Virtual Machine ```sudo vim /etc/login.defs```
    
      - Find this part: ```PASS_MAX_DAYS 9999 PASS_MIN_DAYS 0 PASS_WARN_AGE 7```
    
      - Change that part to ```PASS_MAX_DAYS 30 and PASS_MIN_DAYS 2 keep PASS_WARN_AGE 7 ```
    
      - Type: ```sudo reboot``` to reboot the change affects
        
  *  Creating a Group:
    
      - Type: ```sudo groupadd user42``` to create a group
    
      - Type: ```sudo groupadd evaluating``` to create an evaluating group
    
      - Type: ```getent group``` to check if the group has been created


  *  Creating a User and Assigning Them Into The Group:
    
      -  Type: ```cut -d: -f1 /etc/passwd``` to check all local users
        
      -  Type: ```sudo adduser new_username``` to create a username - write down your new_username, as you will need this later on
        
      -  Type: ```sudo usermod -aG user42 your_username``` to add your username to the user42 group
        
      -  Type: ```getent group user42``` to check if the user is the group
        
      -  Type: ```groups``` to see which groups the user account belongs to
        
      -  Type: ```chage -l your_new_username``` to check if the password rules are working in users

  *  Creating sudo.log:
    
      -  Type: ```cd /var/log```
        
      -  Type: ```mkdir sudo``` (if it already exists, then continue to the next step).
        
      -  Type: ```cd sudo && touch sudo.log```

  *  Configuring Sudoers Group:
    
      -  Type: ```sudo nano /etc/sudoers``` to go the sudoers file
        
      -  Edit your sudoers file to look like the following by adding in all of the defaults in the text below:
        ```
          ╔══════════════════════════════════════════════════════════════════════════════╗
          ║ Defaults	env_reset							 ║
          ║ Defaults	mail_badpass                                                     ║
          ║ Defaults	secure_path="/usr/local/sbin:/usr/local/bin:/usr/bin:/sbin:/bin" ║
          ║ Defaults	badpass_message="Password is wrong, please try again!"           ║
          ║ Defaults	passwd_tries=3                                                   ║
          ║ Defaults	logfile="/var/log/sudo/sudo.log"                                 ║
          ║ Defaults	log_input, log_output                                            ║
          ║ Defaults	requiretty                                                       ║
          ╚══════════════════════════════════════════════════════════════════════════════╝
        ```


         
  *   Crontab Configuation:
    
      -  Type:``` apt-get install -y net-tools``` to install the netstat tools
        
      -  Type: ```cd /usr/local/bin/```
        
      -  Type: ```touch monitoring.sh```
        
      -  Type: ```chmod 777 monitoring.sh```
        
      -  Edit the monitoring script (type: ```sudo vi monitoring.sh```)
    
      -  Paste this text below:
        
       ``` 
        #!/bin/bash
        arc=$(uname -a)
        pcpu=$(grep "physical id" /proc/cpuinfo | sort | uniq | wc -l) 
        vcpu=$(grep "^processor" /proc/cpuinfo | wc -l)
        fram=$(free -m | awk '$1 == "Mem:" {print $2}')
        uram=$(free -m | awk '$1 == "Mem:" {print $3}')
        pram=$(free | awk '$1 == "Mem:" {printf("%.2f"), $3/$2*100}')
        fdisk=$(df -BG | grep '^/dev/' | grep -v '/boot$' | awk '{ft += $2} END {print ft}')
        udisk=$(df -BM | grep '^/dev/' | grep -v '/boot$' | awk '{ut += $3} END {print ut}')
        pdisk=$(df -BM | grep '^/dev/' | grep -v '/boot$' | awk '{ut += $3} {ft+= $2} END {printf("%d"), ut/ft*100}')
        cpul=$(top -bn1 | grep '^%Cpu' | cut -c 9- | xargs | awk '{printf("%.1f%%"), $1 + $3}')
        lb=$(who -b | awk '$1 == "system" {print $3 " " $4}')
        lvmu=$(if [ $(lsblk | grep "lvm" | wc -l) -eq 0 ]; then echo no; else echo yes; fi)
        ctcp=$(ss -neopt state established | wc -l)
        ulog=$(users | wc -w)
        ip=$(hostname -I)
        mac=$(ip link show | grep "ether" | awk '{print $2}')
        cmds=$(journalctl _COMM=sudo | grep COMMAND | wc -l)
        wall "	#Architecture: $arc
	        #CPU physical: $pcpu
	        #vCPU: $vcpu
	        #Memory Usage: $uram/${fram}MB ($pram%)
	        #Disk Usage: $udisk/${fdisk}Gb ($pdisk%)
	        #CPU load: $cpul
	        #Last boot: $lb
	        #LVM use: $lvmu
	        #Connections TCP: $ctcp ESTABLISHED
	        #User log: $ulog
	        #Network: IP $ip ($mac)
	        #Sudo: $cmds cmd"
       ```

      -  Type: ```sudo visudo``` to open your sudoers file
        
      -  Add in this line: ```your_username ALL=(ALL) NOPASSWD: /usr/local/bin/monitoring.sh``` under where its written ```%sudo ALL=(ALL:ALL) ALL```
        
      -  Exit and save your sudoers file
        
      -  Type: ```sudo reboot``` in your Virtual Machine to reboot sudo
        
      -  Type: ```sudo /usr/local/bin/monitoring.sh``` to execute your script as su (super user)
        
      -  Type: ```sudo crontab -u root -e``` to open the crontab and add the rule
        
      -  Type the following: ```*/10 * * * * /usr/local/bin/monitoring.sh``` this means that every 10 mins, this script will show

  
   ### Mandatory part:
   
   *  Installing curl:
      -	To get the latest version of PHP, we need to add a different APT repository, Sury's repository.
     
      -	Type: ```$ sudo apt install curl``` to install curl
        
   *  Retrieving packages with curl
     
      - Type: ```$ sudo curl -sSL https://packages.sury.org/php/README.txt | sudo bash -x```

   *  Installing PHP version 8.1:
     
        ```
        $ sudo apt install php8.1
        $ sudo apt install php-common php-cgi php-cli php-mysql
        ```
      - Check php version
        ```
        $ php -v
        ```

   *  Installing Lighttpd
      - Apache may be installed due to PHP dependencies. Uninstall it if it is to avoid conflicts with lighttpd:
        ```
        $ systemctl status apache2
        $ sudo apt purge apache2
        ```
   * Install lighttpd:
      - Type: $ sudo apt install lighttpd
      - Check version, start, enable lighttpd and check status:
        ```
        $ sudo lighttpd -v
        $ sudo systemctl start lighttpd
        $ sudo systemctl enable lighttpd
        $ sudo systemctl status lighttpd
        ```
      - Allow http port (port 80) through UFW:
        ```
        $ sudo ufw allow http
        $ sudo ufw status
        ```
       - Then forward host port 8080 to guest port 80 in VirtualBox:
         
         Go to VM >> Settings >> Network >> Adapter 1 >> Port Forwarding
         Add rule for host port 8080 to forward to guest port 80
     
       - Test Lighttpd, by going to host machine browser and type in address http://127.0.0.1:8080 or http://localhost:8080. You should see a Lighttpd
         "placeholder page".

        - Get back in VM, activate lighttpd FastCGI module:
          ```
          $ sudo lighty-enable-mod fastcgi
          $ sudo lighty-enable-mod fastcgi-php
          $ sudo service lighttpd force-reload
          ```
       
        - Test php is working with lighttpd, create a file in /var/www/html named info.php. In that php file, write:
          ```
          <?php
          phpinfo();
          ?>
          ```
        - Save and go to host browser and type in the address http://127.0.0.1:8080/info.php. You should get a page with PHP information.

   * Installing MariaDB
      - Type: ```$ sudo apt install mariadb-server```
        
      - Start, enable and check MariaDB status:
        ```
        $ sudo systemctl start mariadb
        $ sudo systemctl enable mariadb
        $ systemctl status mariadb
        ```
        
   * MySQL secure installation:
     - Type: ```$ sudo mysql_secure_installation```
     - Answer the questions like so (root here does not mean root user of VM, it's the root user of the databases!):
       ```
       Enter current password for root (enter for none): <Enter>
       Switch to unix_socket authentication [Y/n]: Y
       Set root password? [Y/n]: Y
       New password: 101Asterix!
       Re-enter new password: 101Asterix!
       Remove anonymous users? [Y/n]: Y
       Disallow root login remotely? [Y/n]: Y
       Remove test database and access to it? [Y/n]:  Y
       Reload privilege tables now? [Y/n]:  Y

       ```
   * Restart MariaDB service:

     - Type: ```$ sudo systemctl restart mariadb```
     - Enter MariaDB interface
     - Type: ```$ mysql -u root -p```
     - Enter MariaDB root password, then create a database for WordPress:
       ```
       MariaDB [(none)]> CREATE DATABASE wordpress_db;
       MariaDB [(none)]> CREATE USER 'admin'@'localhost' IDENTIFIED BY 'WPpassw0rd';
       MariaDB [(none)]> GRANT ALL ON wordpress_db.* TO 'admin'@'localhost' IDENTIFIED BY 'WPpassw0rd' WITH GRANT OPTION;
       MariaDB [(none)]> FLUSH PRIVILEGES;
       MariaDB [(none)]> EXIT;
       ```
       
      - Check that the database was created successfully, go back into MariaDB interface
      - Type: ```$ mysql -u root -p```
        
      - Show databases:
        ```
        MariaDB [(none)]> show databases;
        ```
        
      - You should see something like this:
        ```
        +--------------------+
        | Database           |
        +--------------------+
        | information_schema |
        | mysql              |
        | performance_schema |
        | wordpress_db       |
        +--------------------+
        ```
        
      - If the database is there, everything's good!

  * Installing WordPress
    - We need to install two tools(wget && tar):
      ```
      $ sudo apt install wget
      $ sudo apt install tar
      ```
      
    - Download the latest version of Wordpress, extract it and place the contents in /var/www/html/ directory. Then clean up archive and extraction
      directory:
      ```
      $ wget http://wordpress.org/latest.tar.gz
      $ tar -xzvf latest.tar.gz
      $ sudo mv wordpress/* /var/www/html/
      $ rm -rf latest.tar.gz wordpress/
      ```
      
    - Create WordPress configuration file:
      ```
      $ sudo mv /var/www/html/wp-config-sample.php /var/www/html/wp-config.php
      ```

    - Edit /var/www/html/wp-config.php with database info:
      ```
      <?php
      /* ... */
      /** The name of the database for WordPress */
      define( 'DB_NAME', 'wordpress_db' );

      /** Database username */
      define( 'DB_USER', 'admin' );

      /** Database password */
      define( 'DB_PASSWORD', 'WPpassw0rd' );

      /** Database host */
      define( 'DB_HOST', 'localhost' );
      ```

    - Change permissions of WordPress directory to grant rights to web server and restart lighttpd:
      ```
      $ sudo chown -R www-data:www-data /var/www/html/
      $ sudo chmod -R 755 /var/www/html/
      $ sudo systemctl restart lighttpd
      ```

    - In host browser, connect to http://127.0.0.1:8080 and finish WordPress installation.

  * Postfix Service:
    
     - Postfix definition:
       
       ```
       Postfix is a free and open-source Mail Transfer Agent (MTA).
       In simpler terms, it's the software onyour server that handles
       the routing and delivery of emails.
       
       Here's a breakdown of what an MTA does:
       
       	-> Receives Incoming Emails: When someone sends an email to an address
       on your domain (e.g., [email address removed]), Postfix receives that email.
       
       	-> Routes Emails: Postfix figures out where to send the email based on
       the recipient's address. It might deliver the email directly to a mailbox on
       your server, or it might send it on to another server.
       
       	-> Delivers Emails: Postfix uses protocols like SMTP (Simple Mail Transfer Protocol)
       to send emails to their final destination.

       ```
      - Postfix advantages:
        ```
        Postfix is known for being:
        
        	->Fast and Efficient: It can handle large volumes of email traffic efficiently.
        
        	->Secure: It incorporates security features to protect your server from spam and
        other threats.
        
        	-> Easy to Administer: It's known for being relatively easy to set up and manage
        compared to some other MTAs.
        
        So, if you want your Debian VM to send and receive emails, installing Postfix
        is a great option!
        ```
      - Postfix installation steps:
      - Update and Install Packages:
      - Type: ```$ sudo apt update```
      - Install Postfix and the mailutils package
        ```
        sudo apt install postfix mailutils
        ```

      - Postfix Configuration:
      - During installation, Postfix will prompt you with configuration options. Here's what to choose:
        Visit and follow this video tuorial steps on configuring and testing postfix service on Debian VM [link](https://www.youtube.com/watch?v=oRxk8KWlNRc&t=4s)
    
       - Editing main.cf (Optional):
         While the basic configuration is done, you might want to edit the main Postfix configuration
         file /etc/postfix/main.cf for further customization.
         
       - Type: ```$ sudo nano /etc/postfix/main.cf```
       - Common customizations in main.cf include:
         
         	/*/ Relay restrictions: Define which networks are allowed to send emails through your server.

         	/*/ Mailbox location: Specify where incoming emails are stored.

         	/*/ Alias settings: Create email aliases for forwarding.
    
       - Restart Postfix:
         ```
         $ sudo systemctl restart postfix
         ```
       
       - Once you've made any edits (or left it as is), save the main.cf file and restart Postfix for the changes to take effect
       - Type: $ sudo systemctl restart postfix

    * Testing:
       - You can test your Postfix setup by sending a test email. Here's an example using the mail command:
         ```echo "This is a test email" | mail -s "Postfix Test" root```

       - This sends an email with the subject "Postfix Test" to the root user on your VM.  Check the root user's mailbox
          to see if the email arrived.



## Contributing

If you would like to contribute to this repository by adding new solutions or improving existing ones, please follow these steps:

1. Fork the repository.
2. Create a new branch for your changes: `git checkout -b feature/new-solutions`.
3. Make your changes and commit them: `git commit -m 'Add new solutions'`.
4. Push to the branch: `git push origin feature/new-solutions`.
5. Open a pull request.

## License

This repository is licensed under the license. See the [LICENSE](LICENSE) file for details.
