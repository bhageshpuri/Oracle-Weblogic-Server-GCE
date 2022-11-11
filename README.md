# Oracle-Weblogic-Server-GCE

Create a GCE VM with following configuration:
Machine type - n2-standard-2
centos-7
Open the port 80 on your VPC
Install and configure Oracle weblogic server on the VM.
1] Install JAVA package into our admin server.
sudo su –
yum update
yum install wget
cd /opt/
wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.rpm
yum localinstall -y jdk-8u131-linux-x64.rpm
2] Create JAVA_HOME variable inside the server node and ensure java is installed properly.
vi /root/.bash_profile
export JAVA_HOME=/usr/java/jdk1.8.0_131 
PATH=$JAVA_HOME/bin:$PATH:$HOME/bin 
export PATH

source /root/.bash_profile
java -version

java version "1.8.0_131"
Java(TM) SE Runtime Environment (build 1.8.0_131-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.131-b11, mixed mode)
3] For Oracle weblogic installation it’s a requirement that the installation must be done using non-root user. Let’s proceed to create an additional user for Oracle weblogic server installation.
useradd -s /bin/bash oracle
usermod -aG wheel oracle
passwd oracle

Changing password for user oracle.
New password: 
BAD PASSWORD: The password contains the user name in some form
Retype new password: 
passwd: all authentication tokens updated 

su - oracle
pwd

/home/oracle
4] Configure the environment variables for the Oracle weblogic user.
ORACLE_BASE :: Default Oracle installer directory location
ORACLE_HOME :: Default Oracle database directory location / optional if have Oracle client inside
MW_HOME :: Default Middleware installer directory location
WLS_HOME :: Default Oracle Weblogic managed server directory location
WL_HOME :: Default Oracle Weblogic admin server directory location
DOMAIN_BASE :: Default Oracle Weblogic global domain
DOMAIN_HOME :: Default Oracle Weblogic specific domain
Execute the below commands to set the environment variables.
mkdir wls
cd wls
vi /home/oracle/.bash_profile
export ORACLE_BASE=/home/oracle/wls/oracle 
export ORACLE_HOME=$ORACLE_BASE/product/fmw12 
export MW_HOME=$ORACLE_HOME 
export WLS_HOME=$MW_HOME/wlserver 
export WL_HOME=$WLS_HOME 
export DOMAIN_BASE=$ORACLE_BASE/config/domains 
export DOMAIN_HOME=$DOMAIN_BASE/TEST 
 
export JAVA_HOME=/usr/java/jdk1.8.0_131 
PATH=$JAVA_HOME/bin:$PATH:$HOME/bin 
export PATH

source /home/oracle/.bash_profile 
mkdir -p $ORACLE_BASE 
mkdir -p $DOMAIN_BASE 
mkdir -p $ORACLE_HOME 
mkdir -p $ORACLE_BASE/config/applications 
mkdir -p /home/oracle/wls/oraInventory
5] Create a file called oraInst.loc and wls.rsp .
— The oraInst.loc file is required to define an inventory location during the Oracle weblogic installation.
— The wls.rsp file as it acts as a response file that will be used during the installation.
vi oraInst.loc
inventory_loc=/home/oracle/wls/oraInventory 
inst_group=oracle

vi wls.rsp
[ENGINE] 
Response File Version=1.0.0.0.0 
[GENERIC] 
ORACLE_HOME=/home/oracle/wls/oracle/product/fmw12 
INSTALL_TYPE=WebLogic Server 
DECLINE_SECURITY_UPDATES=true 
SECURITY_UPDATES_VIA_MYORACLESUPPORT=false
6] Proceed with downloading the Oracle Weblogic installer. Visit the URL here to get the Oracle Weblogic version of our choice.
Note: For this article, we have used the Oracle Weblogic version 12.1.3.
https://download.oracle.com/otn/nt/middleware/12c/wls/1213/fmw_12.1.3.0.0_wls.jar

cd $ORACLE_BASE
ls

config  fmw_12.1.3.0.0_wls.jar  product
7] Minimum of 512 MB of swap space is required for Oracle weblogic server.
•	Check if the system has any configured swap by using swapon utility.

swapon -s
If the response is null by the command, then the summary was empty and no swap file exists
Another way of checking for swap space is with the free utility, which shows us the system’s overall memory usage.

free -m
              total        used        free      shared  buff/cache   available
Mem:           3788         251        2630           8         906        3287
Swap:             0           0           0
As you can see, our total swap space in the system is 0.
•	Enable a swapfile.

sudo dd if=/dev/zero of=/swapfile count=1024 bs=1MiB

1024+0 records in
1024+0 records out
1073741824 bytes (1.1 GB) copied, 1.36359 s, 787 MB/s

ls -lh /swapfile
-rw-r--r--. 1 root root 1.0G Oct 21 09:56 

sudo chmod 600 /swapfile

ls -lh /swapfile
-rw-------. 1 root root 1.0G Oct 21 09:56 

sudo mkswap /swapfile

Setting up swapspace version 1, size = 1048572 KiB
no label, UUID=fbaff8fe-522c-46f1-9013-d96e0acc2ab5

sudo swapon /swapfile
free -m

              total        used        free      shared  buff/cache   available
Mem:           3788         236        2734           8         817        3306
Swap:          1023           0        1023
•	Make the swap permanent.
Currently, we have configured the swap space in the above steps but if the system is restarted it will not mount the swap file partitions automatically for use. To make the swap space available permanently we need to create a fstab entry to ensure it will automatically mount the filesystems and partitions.
Add the swapfile entry at the bottom of the page.
sudo vi /etc/fstab

/swapfile   swap    swap    sw  0   0
8] Proceed with the installation.
java -jar /home/oracle/wls/oracle/fmw_12.1.3.0.0_wls.jar -silent -responseFile /home/oracle/wls/wls.rsp -invPtrLoc /home/oracle/wls/oraInst.loc
 
We’ve successfully installed Oracle weblogic. Next, we will proceed with the configuration phase.
9] Configure the Oracle weblogic server.
Let’s proceed with the configuration of weblogic and domain configuration for admin server. We will only create single domain called TEST.
cd $WL_HOME
cd /home/oracle/wls/oracle/product/fmw12/wlserver/ 
cd common/bin/ 
./commEnv.sh 
./wlst.sh 

Java HotSpot(TM) 64-Bit Server VM warning: ignoring option MaxPermSize=256m; support was removed in 8.0
Initializing WebLogic Scripting Tool (WLST) ...
Jython scans all the jar files it can find at first startup. Depending on the system, this process may take a few minutes to complete, and WLST may not return a prompt right away.
Welcome to WebLogic Server Administration Scripting Shell
Type help() for help on available commands
wls:/offline>readTemplate('/home/oracle/wls/oracle/product/fmw12/wlserver/common/templates/wls/wls.jar')
wls:/offline/base_domain>cd('Servers/AdminServer')
 
wls:/offline/base_domain/Server/AdminServer>set('ListenPort',7001)
wls:/offline/base_domain/Server/AdminServer>create('AdminServer','SSL') 
Proxy for AdminServer: Name=AdminServer, Type=SSL
wls:/offline/base_domain/Server/AdminServer>cd('SSL/AdminServer')
wls:/offline/base_domain/Server/AdminServer/SSL/AdminServer>set('Enabled','True')
wls:/offline/base_domain/Server/AdminServer/SSL/AdminServer>set('ListenPort',7002)
wls:/offline/base_domain/Server/AdminServer/SSL/AdminServer>cd('/')
wls:/offline/base_domain>cd('Security/base_domain/User/weblogic')
wls:/offline/base_domain/Security/base_domain/User/weblogic>cmo.setPassword('kV5iP7yS5jP2jY3T')
wls:/offline/base_domain/Security/base_domain/User/weblogic>setOption('OverwriteDomain','true')
wls:/offline/base_domain/Security/base_domain/User/weblogic>writeDomain('/home/oracle/wls/oracle/config/domains/TEST')
wls:/offline/TEST/Security/TEST/User/weblogic>closeTemplate()
wls:/offline>exit()
Exiting WebLogic Scripting Tool.
10] Start the weblogic on the admin server.
cd $DOMAIN_HOME 
cd bin/ 
pwd

cd /home/oracle/wls/oracle/config/domains/TEST/bin

./startWebLogic.sh
Once verified the Oracle weblogic service is running. Let’s go ahead to create a systemd service for it.
11] Create a systemd service for the Oracle weblogic server.
Sometimes we want to run our application automatically if the service is crashed or underlying server restarts. In such cases systemd in Linux helps to configure services which can be managed easily using comands. We will create a systemd service for our weblogic server.
•	Create a service file.

sudo su
cd /etc/systemd/system
vi wls.service

[Unit]
Description=Oracle Weblogic Server Service
After=network.target

[Service]
Type=simple
User=root
ExecStart=/bin/bash /home/oracle/wls/oracle/config/domains/TEST/startWebLogic.sh
Restart=on-abort

[Install]
WantedBy=multi-user.target

sudo systemctl daemon-reload
sudo systemctl start wls.service
sudo systemctl enable wls.service

Created symlink from /etc/systemd/system/multi-user.target.wants/wls.service to /etc/systemd/system/wls.service

sudo systemctl status wls.service
•	Check the status.
 
12] Access the Oracle weblogic admin dashboard.
http://[GCE_INSTANCE_PUBLIC_IP]:7001/console
For this test, we have created domain called TEST. Once you have launched the URL in the browser, you should see the console. Provide the username and password that we’ve defined during configuration above. For this test, the username will be weblogic and password which you we have set earlier which is kV5iP7yS5jP2jY3T .
 
From the Oracle weblogic console click on Environment > Servers tab . You will see the the Weblogic Admin server already included inside TEST domain and it is HEALTHY.
 
1.4. Deploying Spring MVC on Oracle webLogic server
Now, as have our Oracle webLogic server running let’s create sample deployment through which we can compare the results post migration.
1] Let’s install maven utility to package our spring mvc application on your local system.
sudo yum install maven

sudo yum install git
2] Clone the GitHub repository for spring mvc application code.
git clone https://github.com/luvvero/m4a

3] Change the current working directory
cd m4a/sample-app1
4] Build the package
mvn clean // Clears the target directory into which Maven normally builds your project.

mvn install // Builds the project described by your Maven POM file and installs the resulting artifact (WAR) into your local Maven repository
5] A HelloSpringMVC.war will be created in your target/ directory.
 
6] Deploy on Oracle weblogic server.
•	Click on Deployments under the Domain Structure on the left panel.
 
•	To install a new application or module for deployment to targets in this domain, click Install button.
 
•	To upload the HelloSpringMVC.war which was generated earlier and click Next . On the next page, click the Browse button below to select an application or module on the machine from which you are currently browsing. When you have located the file, click the Next button to upload this deployment to the Admin Server.
•	Once the file is successfully uploaded you will receive a success message on top of the panel. Click Next button to prepare a deployment.
•	Else update the path to /etc/systemd/system/m4a/sample-app1/target
 
•	Select the targeting style as Install this deployment as an application and click Next button.
 
•	Provide name for your deployment and click on Finish button to proceed.
 
•	Once the deployment in successful under Control section you will see your application listed and it is HEALTHY.
 
http://[GCE_INSTANCE_PUBLIC_IP]:7001/testpage
 
We have successfully created an Oracle weblogic server running on GCE server.

