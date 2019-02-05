# Linux-Server-Config
This project enables the user to access the files from any location using a secured source like amazon aws cloud service.

# Prerequisites 
   * Virtual Server Amazon EC2
   * Item Catalog Project, created previously.
   * PostgreSQL
   * RSA Private Key for grader :
         https://github.com/KSM1598/Linux-Server-Config/blob/master/Linux_server_31_01_2019.pem
   * Password: ```grader``` 

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

# 3. Change SSH port from 22 to 2200, also remove the comments for all the properties changed.
   * Edit File using: ``` sudo vi /etc/ssh/sshd_config ```
   * Change Port number 22 to 2200
   * PubkeyAuthentication ```yes```
   * PasswordAuthentication ```no```
   * Save and exit using ```esc``` and confirm with ```:wq```
   * Restart: ```sudo service ssh restart```
   * Change inbound rules in Amazon EC2
   * Type : Custom TCP Rule as 2200
   * To check port 2200: Working or Not
   * ```ssh -i linux_31.pem -p 2200 ubuntu@3.90.89.13```
   
# 4: Configure Firewall (UFW)
   Configure the default firewall allow incoming connections for SSH (port 2200) , HTTP (port 80) and NTP (port 123).
   Now Restart ```sudo service ssh restart```
   * ```sudo ufw status```
   * ```sudo ufw default deny incoming```
   * ```sudo ufw default allow outgoing```
   * ```sudo ufw allow 2200/tcp```
   * ```sudo ufw allow www```
   * ```sudo ufw allow 123/udp```
   * ```sudo ufw deny 22```
   * ```sudo ufw enable```
     * After Proceed Option Y/n: ```Y```
   * To know the status of port updations: ``` sudo ufw status```
        
       ```grader@ip-172-31-42-127:~$ sudo ufw status
          Status: active
          To                         Action      From
          --                         ------      ----
          2200/tcp                   ALLOW       Anywhere
          80/tcp                     ALLOW       Anywhere
          22                         DENY        Anywhere
          123/udp                    ALLOW       Anywhere
          2200/tcp (v6)              ALLOW       Anywhere (v6)
          80/tcp (v6)                ALLOW       Anywhere (v6)
          22 (v6)                    DENY        Anywhere (v6)
          123/udp (v6)               ALLOW       Anywhere (v6)```
  
# Creating grader:
  * Create a user with password: ```sudo adduser grader```
    * password: ```grader```
  * Give the grader premissions ```sudo visudo```.
  * Now below the root add grader as ```grader ALL= (ALL:ALL) ALL```
  * Verify the grader permissions ```su -grader``` and password: ```grader```
  * Configure keypairs for grader Create .ssh folder by mkdir /home/grader/.ssh
  * Config key pair authentication to grader 
    * ```create .ssh folder```
    * ```mkdir /home/grader/.ssh```
    * ```su grader```
  * Change from ubuntu to grader
    * ```sudo cp /home/ubuntu/.ssh/authorized_keys /home/grader/.ssh/authorized_keys```
  * Change ownership ```chown grader.grader /home/grader/.ssh```
  * Add grader to sudogroup
       ```sudo su
          usermod -aG sudo grader```
  * Change permissions to ```.ssh``` folder
      ```chmod 700 /home/grader/.ssh
         chmod 644 /home/grader/.ssh/authorized_keys
         vi /etc/ssh/sshd_config```
  * Edit authentications as permit root login ```no``` and pubkeyauthentication ```yes```(esc + :wq)
  * Restart the service :
      * ```sudo service ssh restart```
  * Open the server with in the grader:
      * ```ssh -i Linux_server_31_01_2019.pem -p 2200 grader@3.91.12.127```
  * Set the time zone for grader
      * ```sudo dpkg-reconfigure tzdata```
