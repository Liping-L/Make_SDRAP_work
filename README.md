# Making_SDRAP_work
Some requirements of SDRAP (https://github.com/JasperBraun/SDRAP)

IMPORTANT：
 These are just some personal steps that worked for me, but I hope they work for you too.
 If there are any mistakes, please feel free to let me know.

## Step 01: install CentOS 6.7 with apache-2.2.15 pre-installed
 The image can be downloaded from 
 https://vault.centos.org/6.7/isos/x86_64/CentOS-6.7-x86_64-bin-DVD1.iso
 
 ***When configuring the virtual machine network working mode, it's better to use 'Bridged' mode.***

## Step 02: install php 5.3.3
***There are 5 rpm files in total.***
(***php-pdo\* is the prerequisite of php-mysql\****)

` sudo rpm -ivh php-common-*.rpm php-cli-*.rpm php-*.rpm php-pdo-*.rpm`

` sudo rpm -ivh php-mysql-*.rpm`

## Step 03: install mysql 5.6.31

***There are 6 rpm files in total***

<pre>
 sudo yum remove mysql-libs #Remove the pre-installed mysql-libs that conflicts with MySQL-5.6.31

 
 sudo rpm -ivh MySQL-client-5.6.31-1.el6.x86_64.rpm \
  MySQL-devel-5.6.31-1.el6.x86_64.rpm \
  MySQL-embedded-5.6.31-1.el6.x86_64.rpm \
  MySQL-shared-5.6.31-1.el6.x86_64.rpm \
  MySQL-shared-compat-5.6.31-1.el6.x86_64.rpm \
  MySQL-server-5.6.31-1.el6.x86_64.rpm 

 sudo  service mysql start
</pre>

### The temporary root password for MySQL can be found at: 

` sudo cat /root/.mysql_secret`

### Then execute in terminal:
<pre>
   mysql -u root -p   # Input temporary password
</pre>
<pre>
   mysql> SET PASSWORD FOR 'root'@'localhost' = PASSWORD('new_root_password');  # set a new password for root
   mysql> CREATE USER 'new_user'@'localhost' IDENTIFIED BY 'new_user_password';  # create new user
   mysql> GRANT ALL PRIVILEGES ON *.* TO 'new_user'@'localhost';  # set password
   mysql> quit;
</pre>

## Step 04: move BLAST executable files into /bin
<pre>
 wget https://ftp.ncbi.nlm.nih.gov/blast/executables/blast+/2.2.31/ncbi-blast-2.2.31+-x64-linux.tar.gz
 tar -xzf ncbi-blast-2.2.31+-x64-linux.tar.gz
 sudo cp ncbi-blast-2.2.31+/bin/* /bin 
</pre>


## Step 05: enable ServerName: 
***use port 80 or any other***
<pre>
# edit 'ServerName' in /etc/httpd/conf/httpd.conf into: ServerName localhost:80
 sudo vi /etc/httpd/conf/httpd.conf   
  # line 276, delete the '#' and input: ServerName localhost:80
</pre>

## Step 06: configure firewall:
***use the same port as defined above***
<pre>
 sudo vi /etc/sysconfig/iptables 
  # Add the following line: '-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT'
</pre>

## Step 07: reboot firewall and apache:
<pre>
sudo service iptables restart
sudo service httpd restart
</pre>
### Make sure that httpd, MySQL, and iptables are enabled to start on boot.
<pre>
  sudo chkconfig httpd on 
  sudo chkconfig mysqld on 
  sudo chkconfig iptables on
</pre>

### if the firewall is not needed
`chkconfig iptables off && service iptables stop`

## Step 08: configure SDRAP
<pre>
 wget https://github.com/JasperBraun/SDRAP/archive/refs/tags/v1.0.0.tar.gz
 sudo cp SDRAP-1.0.0.tar.gz /var/www/html # or v1.0.0.tar.gz
 cd /var/www/html
 sudo tar -xzf SDRAP-1.0.0.tar.gz
 sudo mv SDRAP-1.0.0/* ./
 sudo sh install.sh
 sudo chcon -Rt httpd_user_content_t tmp # might not be necessary
</pre>

## Step 09: access CentOS web server in LAN
***Access the IP address of the virtual machine through a web browser on other machines in the LAN (e.g. http://192.168.xx.xxx, obtained using 'ifconfig')***

` ifconfig | grep inet`

***When using NAT mode for the virtual machine, 127.0.0.1 or localhost may work for the access from the host***

***Use the right port as defined above (e.g. 127.0.0.1:80)***


## Step 10: to enable the multi-threaded alignment of BLAST:
### modify the codes:
<pre>
  # edit the ***php*** scripts
  sudo vi /var/www/html/php/sdrap/validate_inputes.php
  # add '-num_threads N' to the end of $BLAST_PARAMETERS
</pre>
### enable the system to allow multi-threaded BLAST by php
<pre>
  # install EPEL library（if not installed）
  yum install epel-release -y
  # install policycoreutils-python
  yum install policycoreutils-python -y

  # create an SELinux policy module for BLAST
  sudo grep blastn /var/log/audit/audit.log | audit2allow -M myblast
  sudo semodule -i myblast.pp
  # These steps will create two files: myblast.pp and myblast.te at working directory, they are not needed anymore.
</pre>
