<h1>Multi Tier Web Application Stack Setup Locally(VM).</h1>

<img src="https://i.imgur.com/rBWqVGC.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

<h2>Description</h2>
The "VPROFILE PROJECT SETUP" is a guide to set up a multi-tier web application stack locally using VirtualBox and Vagrant. It requires Oracle VM VirtualBox and Vagrant, along with specific plugins. This setup allows developers to work on a local development environment to test and develop web applications without affecting production systems.
<br />


<h2>Prerequisite</h2>

- <b> Oracle VM Virtualbox
- Vagrant
- Vagrant plugins
- vagrant plugin install vagrant-hostmanager(to ping other services)
- vagrant plugin install vagrant-vbguest
- Git bash or equivalent editor </b> 


<h2>Program walk-through:</h2>

<p align="center">
Follow the steps:
  
VM Setup

clone the repository on git bash for windows and terminal for mac and linux. 

 <br/>
<img src="https://i.imgur.com/K93NuB2.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
  <br />
<br />

2. I went to the directory that has my Vagrantfile. I installed vagrant host manager with the command below before powering up my Virtual Box.:  

 <br/>
<img src="https://i.imgur.com/iW9FzwN.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />

3. After installing the plugin, I ran the command below to power up my VMs.

 <br/>
<img src="https://i.imgur.com/nknH4WO.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />

4. We can check our VMs from Oracle VM VirtualBox Manager.

 <br/>
<img src="https://i.imgur.com/81ORaZb.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />

5. Next, I validated my VMs one by one with command vagrant ssh <name_of_VM_given_in_Vagrantfile>

 <br/>
<img src="https://i.imgur.com/0eROVyV.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />

6. I checked /etc/hosts file. Since plugin have have been installed, /etc/hosts file will be updated automatically.

 <br/>
<img src="https://i.imgur.com/isDh46o.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />

7. Now I will try to ping app01 from web01 vbox.

 <br/>
<img src="https://i.imgur.com/Qb1KzAW.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

<img src="https://i.imgur.com/JXyz9uu.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />

8. I was able to connect app01 successfully. I will check other services similarly.

<br/>
<br />

9. Lets connect to app01 vbox and check connectivity of app01 with rmq01, db01 and mc01.

 <br/>
<img src="https://i.imgur.com/LQ912lv.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />

10. There are 6 different services for my application.

<br/>
<img src="https://i.imgur.com/8GLquSv.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />

11. I need to set up my services in the order mentioned below.

<br/>
<img src="https://i.imgur.com/xN9bYQq.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
<br />

12. Provisioning MySQL

<br/>
<img src="https://i.imgur.com/K8t3bXD.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />

20. Now, we will create a CI/CD pipeline to fetch the code from GitHub.

21. Switch to root user and update all packages.

22. It is always a good practice to update OS with the latest patches when logged into the VM for the first time.

- sudo su 
- yum update -y

23. Set up the DB password using the DATABASE_PASS environment variable and add it to the /etc/profile file

- DATABASE_PASS='admin123'

24. The above variable will be temporary, so I need to add the variable above to/etc/profile file and update it to make it permanent.

- vi /etc/profile
- source /etc/profile

25. Next, I Installed the EPEL(Extra Packages for Enterprise Linux) repository.
You can look up EPEL here

- yum install epel-release -y

26. Install Maria DB Package

- yum install git mariadb-server -y

With Mariadb installed, start and enable Mariadb service. We can also check the status of MariaDB service to make sure it is actively(running).

- systemctl start mariadb
- systemctl enable mariadb
- systemctl status mariadb

27.  RUN mysql secure installation script.

- mysql_secure_installation

<img src="https://i.imgur.com/jc7KBo0.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

  <br/>
<br />

28. Check connectivity to db with command blow: Once it asks password, enter admin123. If the connection is successful exit from DB.

- mysql -u root -p
- exit

<br/>
<img src="https://i.imgur.com/NliE2JN.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />

<br />

29. Next, clone the source code to the db VM then change the directory to src/main/resources/ to get the `SQL queries.

- git clone https://github.com/devopshydclub/vprofile-project.git
- cd vprofile-project/src/main/resources/

<br/>
<img src="https://i.imgur.com/qDYPhiK.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />

<br />

30. Run the command below before initializing DB.

- mysql -u root -p"$DATABASE_PASS" -e "create database accounts"
- mysql -u root -p"$DATABASE_PASS" -e "grant all privileges on accounts.* TO 'admin'@'app01' identified by 'admin123' "
- cd ../../..
- mysql -u root -p"$DATABASE_PASS" accounts < src/main/resources/db_backup.sql
- mysql -u root -p"$DATABASE_PASS" -e "FLUSH PRIVILEGES"

31. Login to a database and do a quick verification to see if SQL queries start a database with role , user and user_role tables.

- mysql -u root -p"$DATABASE_PASS"
- MariaDB [(none)]> show databases;
- MariaDB [(none)]> use accounts;
- MariaDB [(none)]> show tables;
- exit

32. Restart our MariaDB server and log out.

- systemctl restart mariadb
- logout

Provisioning Memcache

33. Login to memcached server, and switch to root user.

- vagrant ssh mc01
- sudo su -

34. Similar to MySQL provisioning, start with updating OS with latest patches and download epel repository.

- yum update -y
- yum install epel-release -y

35. Install memcached package.

- yum install memcached -y

36. Start/enable the memcached service and check the status of service.

- systemctl start Memcached
- systemctl enable memcached
- systemctl status memcache

37. Run command below to enablememcached listen on TCP port 11211 and UDP port 11111.

- memcached -p 11211 -U 11111 -u memcached -d

Validate if port is running with command below:

- ss -tunlp | grep 11211

All good, exit from server with exit command.
 <br/>

<img src="https://i.imgur.com/e5Orx7n.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />

<br />
Provisioning RabbitMQ

38. Login to Rabbit MQ server and switch to root user.

- vagrant ssh rmq01
- sudo su -

39. Updating OS with latest patches and download epel repository.

- yum update -y
- yum install epel-release -y

41. Before installing RabbitMQ, we will install some dependencies first.

- yum install wget -y
- cd /tmp/
- wget http://packages.erlang-solutions.com/erlang-solutions-2.0-1.noarch.rpm
- sudo rpm -Uvh erlang-solutions-2.0-1.noarch.rpm

43. Now Install RabbitMQ server. With command below, install the script and pipe with shell to execute the script.

- curl -s https://packagecloud.io/install/repositories/rabbitmq/rabbitmq-server/script.rpm.sh | sudo bash
- sudo yum install rabbitmq-server -y

44. Lets start/enable the rabbitmq service and check the status of service.

- systemctl start rabbitmq-server
- systemctl enable rabbitmq-server
- systemctl status rabbitmq-server

45. Create a test user with password test

46. create user_tag for test user as administrator.

47. If completed config changes, restartrabbitmq service

48. If completed config changes, restartrabbitmq service

- cd ~
- echo "[{rabbit, [{loopback_users, []}]}]." > /etc/rabbitmq/rabbitmq.config
- rabbitmqctl add_user test test
- rabbitmqctl set_user_tags test administrator
- systemctl restart rabbitmq-server

50. If your service active/running after restart, then we can move to the next service.

  <br/>
<img src="https://i.imgur.com/xPubFVZ.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />

<br />
Provisioning Tomcat

51. Login to app01 server first, and switch to root user.

- vagrant ssh app01
- sudo su -

52. Update OS with latest patches and download epel repository.

- yum update -y
- yum install epel-release -y
 
53. Install dependencies for Tomcat server.

- yum install java-1.8.0-openjdk -y
- yum install git maven wget -y

54. To download Tomcat, switch to /tmp/ directory.

- cd /tmp
- wget https://archive.apache.org/dist/tomcat/tomcat-8/v8.5.37/bin/apache-tomcat-8.5.37.tar.
- tar xzvf apache-tomcat-8.5.37.tar.gz

55. Add tomcat user and copy data to tomcat home directory.

56. Check the new user tomcat with id tomcat command.

- useradd --home-dir /usr/local/tomcat8 --shell /sbin/nologin tomcat

57. Copy your data to /usr/local/tomcat8 directory which is the home-directory for tomcat user.

- cp -r /tmp/apache-tomcat-8.5.37/* /usr/local/tomcat8/
- ls /usr/local/tomcat8

58. Currently root user has ownership of all files under /usr/local/tomcat8/ directory. change it to tomcat user.

- ls -l /usr/local/tomcat8/
- chown -R tomcat.tomcat /usr/local/tomcat8
- ls -l /usr/local/tomcat8/

59. Setup systemd for tomcat, create a file with below content. After creating this file, start tomcat service with systemctl start tomcat and stop tomcat with systemctl stop tomcat commands.

60. Any changes made to file under /etc/systemd/system/ directory, we need to run below command to be effective:
  
<br/>
<img src="https://i.imgur.com/M6xkaL0.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />

<br />

61. Enable tomcat service. The service name tomcat has to be same as given /etc/systemd/system/tomcat.service directory.

- systemctl daemon-reload
- systemctl enable tomcat
- systemctl start tomcat
- systemctl status tomcat <br/>
<img src="https://i.imgur.com/5NNPrmX.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />

<br />

62. In /tmp directory, we will clone our source code here.

- git clone https://github.com/devopshydclub/vprofile-project.git
- ls
- cd vprofile-project/

63. Before building your artifact, you need to update our configuration file that will be connect to our backend services db, memcached and rabbitmq service.

- vi src/main/resources/application.properties

64. In application.properties file: make sure the settings are correct. First check DB configuration. Our db server is db01 , and we have admin user with password admin123 as we setup. For memcached service, hostname is mc01 and we validated it is listening on tcp port 11211. Fort rabbitMQ, hostname is rmq01 and we have created user test with pwd test.

- #JDBC Configutation for Database Connection
- jdbc.driverClassName=com.mysql.jdbc.Driver
- jdbc.url=jdbc:mysql://db01:3306/accounts?useUnicode=true&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull
- jdbc.username=admin
- jdbc.password=admin123
- #Memcached Configuration For Active and StandBy Host
- #For Active Host
- memcached.active.host=mc01
- memcached.active.port=11211
- #For StandBy Host
- memcached.standBy.host=127.0.0.2
- memcached.standBy.port=11211
- #RabbitMq Configuration
- rabbitmq.address=rmq01
- rabbitmq.port=5672
- rabbitmq.username=test
- rabbitmq.password=test

65. Run mvn install command to create our artifact. Artifact will be created in/tmp/vprofile-project/target/vprofile-v2.war
  <br/>
<img src="https://i.imgur.com/cQ59UF2.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />

<br />

66. Deploy our artifact vprofile-v2.war to Tomcat server. But before that, remove default app from our server. For that reason first we will shutdown server. The default app will be in /usr/local/tomcat8/webapps/ROOT directory.

- systemctl stop tomcat
- systemctl status tomcat
- rm -rf /usr/local/tomcat8/webapps/ROOT

67. Our artifact is under vprofile-project/target directory. Now we will copy our artifact to /usr/local/tomcat8/webapps/ directory as ROOT.war and start tomcat server. Once we start the server, it will extract our artifact ROOT.war under ROOT directory.
cd ..

- cp target/vprofile-v2.war /usr/local/tomcat8/webapps/ROOT.war
- systemctl start tomcat
- ls /usr/local/tomcat8/webapps/

By the time, our application is coming up we can provision our Nginx server.

Provisioning Nginx

68. Nginx server is Ubuntu, despite our servers are RedHat. To update OS with latest patches run below command:

- sudo apt update && sudo apt upgrade
- Install nginx.
- sudo su -
- apt install nginx -y

69. Create a Nginx configuration file under directory /etc/nginx/sites-available/ with below content:

vi /etc/nginx/sites-available/vproapp
Content to add:
upstream vproapp {
server app01:8080;
}
server {
listen 80;
location / {
proxy_pass http://vproapp;
}
}

70. Remove default nginx config file:

- rm -rf /etc/nginx/sites-enabled/default

71. Create a symbolic link for our configuration file using default config location as below to enable our site. Then restart nginx server.

- ln -s /etc/nginx/sites-available/vproapp /etc/nginx/sites-enabled/vproapp
- systemctl restart nginx
<br/>
<br />
<br />

Validate Application from Browser

In web01 server, run ifconfig to get its IP address. So the IP address of our web01 is : 192.168.56.11

- enp0s8: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.56.11

72. First validate Nginx is running on browser http://<IP_of_Nginx_server>.

<img src="https://i.imgur.com/uXfzzkO.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
<br />

73. Validate Db connection using credentials admin_vp for both username and password.

74. Validate app is running from Tomcat server
 <br/>
<img src="https://i.imgur.com/wBQ97Kv.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
<br />

75. Validate RabbitMQ connection by clicking RabbitMQ

<br/>
<img src="https://i.imgur.com/vW7fg3o.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
<br />

76. Validate Memcache connection by clicking MemCache

<br/>
<img src="https://i.imgur.com/AD1ble1_d.webp?maxwidth=760&fidelity=grand" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
<br />

77. Validate data is coming from Database when user first time requests it.

<img src="https://i.imgur.com/hE6tT88.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
<br />

78. Validate data is coming from Memcached when user second time requests it.
    <br/>
</p>
- <b>~Hassanat Hussain</b>
<!--
 ```diff
- text in red
+ text in green
! text in orange
# text in gray
@@ text in purple (and bold)@@
```
