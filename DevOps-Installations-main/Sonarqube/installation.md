**✅ 1. Install OpenJDK 17 or above**


Install OpenJDK 17.
```bash
apt-get install openjdk-17-jdk -y
```

**----------------------------------------------------------------------------------------**

**✅ 2. Import the repository signing key**
```bash
sudo apt install curl ca-certificates
sudo install -d /usr/share/postgresql-common/pgdg
sudo curl -o /usr/share/postgresql-common/pgdg/apt.postgresql.org.asc --fail https://www.postgresql.org/media/keys/ACCC4CF8.asc
```
 
**✅ 3. Create the repository configuration file**

```bash
. /etc/os-release
sudo sh -c "echo 'deb [signed-by=/usr/share/postgresql-common/pgdg/apt.postgresql.org.asc] https://apt.postgresql.org/pub/repos/apt $VERSION_CODENAME-pgdg main' > /etc/apt/sources.list.d/pgdg.list"
```

**✅ 4. Update the package lists**
```bash
sudo apt update
```

**✅ 5.Install the latest version of PostgreSQL**
 **If you want a specific version, use 'postgresql-17' or similar instead of 'postgresql'**
```bash
sudo apt -y install postgresql
```

**✅ 6. Enable the database server to start automatically on reboot**
```bash
systemctl enable postgresql
```
**✅ 7. Start the database server**
```bash
systemctl start postgresql
```
**✅ 8. Change the default PostgreSQL password**
**use the same password in step 12**
```bash
passwd postgres
```
**✅ 9. Switch to the postgres user**
```bash
su - postgres
```
**✅ 10. Create a user named sonar**
```bash
createuser sonar
```
**✅ 11. Log in to PostgreSQL**
```bash
psql
```
**✅ 12. Set a password for the sonar user**
```bash
ALTER USER sonar WITH ENCRYPTED password 'my_strong_password';
```
**✅ 13. Create a sonarqube database and set the owner to sonar**
```bash
CREATE DATABASE sonarqube OWNER sonar;
```
**✅ 14. Grant all the privileges on the sonarqube database to the sonar user**
```bash
GRANT ALL PRIVILEGES ON DATABASE sonarqube to sonar;
```
**✅ 15. Exit PostgreSQL**
```bash
\q
```
**✅ 16.Return to your non-root sudo user account**
```bash
exit
```
**---------------------------------------------------------------------------------------**


**✅ 17.Download and Install SonarQube**
  **install the zip utility, which is needed to unzip the SonarQube files**
```bash
sudo apt-get install zip -y
```
 **✅ 18. Locate the latest download URL from the SonarQube official download page**

**Download the SonarQube distribution files**
```bash
sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.9.3.79811.zip
```
**✅ 19. Unzip the downloaded file**
```bash
sudo unzip sonarqube-9.9.3.79811.zip
```
**✅ 20. Move the unzipped files to /opt/sonarqube directory**
```bash
cd /opt
sudo mv sonarqube-9.9.3.79811.zip sonarqube
sudo mv sonarqube /opt/
```
**✅ 21. Add SonarQube Group and User**
**Create a dedicated user and group for SonarQube**
**Create a sonar group**
```bash
sudo groupadd sonar
```
**✅ 22.Create a sonar user and set /opt/sonarqube as the home directory**
```bash
sudo useradd -d /opt/sonarqube -g sonar sonar
```
**✅ 22. Grant the sonar user access to the /opt/sonarqube directory**
```bash
sudo chown sonar:sonar /opt/sonarqube -R
```

**---------------------------------------------------------------------------------------**

**✅ 23. Configure SonarQube**
 **Edit the SonarQube configuration file**
```bash
sudo vi /opt/sonarqube/conf/sonar.properties
```
**✅ 24. Find the following lines**
**
**#sonar.jdbc.username=**
**#sonar.jdbc.password=**
**Uncomment the lines, and add the database user and password you created in Step 2**
```bash
sonar.jdbc.username=sonar
sonar.jdbc.password=my_strong_password
```
**✅ 25. Below those two lines, add the sonar.jdbc.url**
```bash
sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonarqube
```
Save and exit the file


**✅ 27. Edit the sonar script file**
```bash
vi /opt/sonarqube/bin/linux-x86-64/sonar.sh
```
add the below line, in the first line itself
```bash
RUN_AS_USER=“sonar”
```


**Save and exit the file**


**---------------------------------------------------------------------------------------**

**✅ 28. Setup Systemd service**
 **Create a systemd service file to start SonarQube at system boot**
```bash
sudo vi /etc/systemd/system/sonar.service
```

**Paste the following lines to the file**
```bash
[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking

ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop

User=sonar
Group=sonar
Restart=always

LimitNOFILE=65536
LimitNPROC=4096

[Install]
WantedBy=multi-user.target
```
**Save and exit the file**
**Enable the SonarQube service to run at system startup**
```bash
sudo systemctl enable sonar
 ```
**Start the SonarQube service**
```bash
sudo systemctl start sonar
```
**Check the service status**
```bash
sudo systemctl status sonar
```
**---------------------------------------------------------------------------------------**

**✅ 29. Modify Kernel System Limits**
**SonarQube uses Elasticsearch to store its indices in an MMap FS directory**
**It requires some changes to the system defaults**

 **Edit the sysctl configuration file**
```bash
sudo vi /etc/sysctl.conf
```
**Add the following lines**
```bash
vm.max_map_count=262144
fs.file-max=65536
ulimit -n 65536
ulimit -u 4096
```
**Save and exit the file**
**Reboot the system to apply the changes**


After rebooting, go to security group of EC2 instance and edit inbound rules and add Custom TCP of 9000, 
because by default sonarqube runs on 9000 port. 
Now ping with public ip address of EC2 instance,
you will be redirected to login page
by default username is admin & password is also admin.
