# Linux-Server-Config
This project enables the user to access the files from any location using a secured source like amazon aws cloud service.

# Prerequisites 
* Virtual Server Amazon EC2
* Item Catalog Project, created previously.
* PostgreSQL
* RSA Private Key for grader :
     https://github.com/KSM1598/Linux-Server-Config/blob/master/Linux_server_31_01_2019.pem
     Password: ```grader``` 

# 1. Creating New Ubuntu Linux server instance on Amazon EC2
  * Login to ```aws.amazon.com``` and login to default user (ubuntu)
  * Choose EC2 and Launch Instance.
  * Edit Inbound rules and add 3 rules i.e,add port ssh 2200, Http port 80, NTP port 123 and save.
  * Check for instance ```IPv4 public IP - 3.90.89.13``` we can download a ```.pem``` File and connect with following command:
  * ssh -i Linux_server_31_01_2019.pem -p 2200 grader@3.91.12.127
  * 22 is Port by Default, Later we need to Change 2200.
  
# 2: Update and upgrade installed packages
  * sudo apt-get update
  * sudo apt-get upgrade
  * sudo apt-get install unattended-upgrades
  * sudo apt-get dist-upgrade
