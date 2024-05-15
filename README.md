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
  * - Install your Debian OS as '.iso' file. [Debian OS Download Link] (https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/)
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

  *  Bonus part:

Explain how to use the project. Provide examples, code snippets, or screenshots to illustrate usage. Include any important commands or options users need 
to know.

## Contributing

If you would like to contribute to this repository by adding new solutions or improving existing ones, please follow these steps:

1. Fork the repository.
2. Create a new branch for your changes: `git checkout -b feature/new-solutions`.
3. Make your changes and commit them: `git commit -m 'Add new solutions'`.
4. Push to the branch: `git push origin feature/new-solutions`.
5. Open a pull request.

## License

This repository is licensed under the license. See the [LICENSE](LICENSE) file for details.
