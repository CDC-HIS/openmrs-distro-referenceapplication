# EthiOHRI Implementation on Ubuntu 20.04

## System Requirements
- **OS**: Ubuntu 20.04
- **OpenMRS Platform**: 2.6.0

## Prerequisites installation
- Java 1.8
- MySQL 5.7
- Tomcat 8
- wget

## Required files
- modules.zip
- openmrs.war (*should be* `version 2.6.0`)
- frontend.zip
- configuration.zip
- js.zip
---


## 1. Install Java 1.8
```bash
sudo apt-get update
```
```bash
sudo apt install -y openjdk-8-jdk
```
```bash
java -version
```

## 2. Install MySQL - Recommended version: MySQL  8
```bash
sudo apt install mysql-server -y
```
```bash
sudo mysql
```
```bash
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'newpassword';
```
```bash
FLUSH PRIVILEGES;
```
```bash
exit;
```
```bash
sudo mysql_secure_installation
```
Accept all requests by answering yes while running the above command.
```bash
mysql -u root -p
# Enter you new password for root user
```
#### Create mysql user with username `openmrs` and database `openmrs`
```bash
CREATE USER 'openmrs'@'%' IDENTIFIED BY 'Pd123#@!';
GRANT ALL PRIVILEGES ON *.* TO 'openmrs'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
 CREATE DATABASE openmrs;
Exit;
```

## 3. Install Apache Tomcat 8.5.79

### Install wget package
```bash
sudo apt install wget
```
####  Download tomcat 8
```bash
wget https://archive.apache.org/dist/tomcat/tomcat-8/v8.5.79/bin/apache-tomcat-8.5.79.tar.gz
```
#### Extract `apache-tomcat-8.5.79.tar.gz`
```bash
sudo tar xzf apache-tomcat-8.5.79.tar.gz
```
#### Move the extracted tomcat file to the following directory
```bash
sudo mv apache-tomcat-8.5.79 /usr/share/tomcat/tomcat8
```
### Create tomcat8  user
```bash
sudo useradd -m -d /usr/share/tomcat/tomcat8 tomcat8
```
```bash
sudo chown tomcat8:tomcat8 -R /usr/share/tomcat/tomcat8
```
### Add tomcat service
```bash
sudo nano /etc/systemd/system/tomcat.service
```
#### Past the following configuration inside the `tomcat.service`
```
# copy and paste below script
[Unit]
Description: Apache Tomcat 8.5 Server
After=syslog.target network.target

[Service]
Type=forking
User=tomcat8
Group=tomcat8
LimitNOFILE = 65536

Environment=CATALINA_HOME=/usr/share/tomcat/tomcat8
Environment=CATALINA_BASE=/usr/share/tomcat/tomcat8

ExecStart=/usr/share/tomcat/tomcat8/bin/catalina.sh start
ExecStop=/usr/share/tomcat/tomcat8/bin/catalina.sh stop

RestartSec=10
Restart=always
[Install]
WantedBy=multi-user.target
```
#### Press `ctr+s`, `ctrl+x`

### Run the following commands to register and start the service
```bash
sudo systemctl daemon-reload
```
```bash
sudo systemctl start tomcat
```
## 4 Openmrs Installation
#### 4.1 Copy *openmrs.war* file to `/usr/share/tomcat/tomcat8/webapps/`
   ```bash
   cp openmrs.war /usr/share/tomcat/tomcat8/webapps/
   ```
#### 4.2 unzip all zip files mentioned under [Required files](#Required-files): 
   ```bash
   unzip modules.zip -d /usr/share/tomcat/tomcat8/.OpenMRS/
   ```
   ```bash
   unzip frontend.zip -d /usr/share/tomcat/tomcat8/.OpenMRS/
  ```
   ```bash
   unzip configuration.zip -d /usr/share/tomcat/tomcat8/.OpenMRS/
   ```
   ```bash
   unzip js.zip -d /usr/share/tomcat/tomcat8/webapps/openmrs/WEB-INF/views/scripts/
   ````
#### 4.3 Change permissions of the OpenMRS directory
```bash
sudo chmod -R 755 /usr/share/tomcat/tomcat8/.OpenMRS/
```
#### 4.4 Change ownership of the tomcat directory
```bash
sudo chown -R tomcat8:tomcat8 /usr/share/tomcat/tomcat8/
```
#### 4.5 Remove configuration_checksum
```bash
sudo rm -rf /usr/share/tomcat/tomcat8/.OpenMRS/configuration_checksum
```
#### 4.6 Restart tomcat service
```bash
sudo systemctl restart tomcat
```
#### 4.7 Access OpenMRS
Open your web browser and navigate to:
```
http://localhost:8080/openmrs
```
#### 4.8 Follow the setup wizard
- **Database Type**: MySQL
- **Database Hostname**: localhost
- **Database Port**: 3306
- **Database Name**: openmrs
- **Database Username**: openmrs
- **Database Password**: Pd123#@!
- **Admin Username**: admin
- **Admin Password**: Admin123
- **OpenMRS Version**: 2.6.0

## 5. Post-Installation
### 5.1 Configure OpenMRS
- Navigate to the OpenMRS administration page.
- Configure the modules as per your requirements.
- Set up user roles and permissions.

### 5.2 Backup OpenMRS
- Regularly back up your OpenMRS database and configuration files.
- Use the following command to back up the MySQL database:
```bash
mysqldump -u openmrs -p openmrs > openmrs_backup.sql
```
### 5.3 Monitor Logs
- Monitor the OpenMRS logs for any issues:
```bash
tail -f /usr/share/tomcat/tomcat8/logs/catalina.out
```
### 5.4 Update OpenMRS
- Regularly check for updates to OpenMRS and its modules.
- To update, download the latest `openmrs.war` file and replace the existing one in `/usr/share/tomcat/tomcat8/webapps/`.
- Restart the Tomcat service after replacing the WAR file:
```bash
sudo systemctl restart tomcat
```



      
    


