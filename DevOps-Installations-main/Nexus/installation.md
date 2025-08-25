**✅ 1. Install OpenJDK 17 on Ubuntu 22.04 LTS**
```bash
sudo apt update
sudo apt install openjdk-17-jdk -y
```

**✅ 2. Download and Install Nexus Repository Manager on Ubuntu 22.04 LTS**
```
cd /opt
sudo wget https://download.sonatype.com/nexus/3/nexus-3.82.0-08-linux-x86_64.tar.gz
```

**✅ 3. Extract the tar file**
```
sudo tar -xvzf nexus-3.82.0-08-linux-x86_64.tar.gz
```

**✅ 4. Lets create new user named nexus to run nexus service**
```
sudo adduser nexus
```
**✅ 5. To set no password for nexus user open the visudo file in ubuntu**
```
sudo visudo
```
**✅ 6. Add below line into it , save and exit**
```
nexus ALL=(ALL) NOPASSWD: ALL
```
**✅ 7. Give permission to nexus files and nexus directory to nexus user**
```
sudo chown -R nexus:nexus /opt/nexus
sudo chown -R nexus:nexus /opt/sonatype-work
```
**✅ 8. Open /opt/nexus/bin/nexus.rc file, 
Uncomment it and add nexus user as shown below**
```
sudo vi /opt/nexus/bin/nexus.rc
```
**uncomment and add**
```
run_as_user="nexus"
```
**✅ 9. Open the /opt/nexus/bin/nexus.vmoptions file**
```
sudo vi /opt/nexus/bin/nexus.vmoptions
```
**To Increase the nexus JVM heap size, you can modify the size as shown below**
```
-XX:MaxDirectMemorySize=2703m
-Djava.net.preferIPv4Stack=true
```
**✅ 10. Run Nexus as a service using Systemd**
**To run nexus as service using Systemd**
```
sudo vi /etc/systemd/system/nexus.service
```
**insert the below content**
```
[Unit]
Description=nexus service
After=network.target

[Service]
Type=forking
LimitNOFILE=65536
ExecStart=/opt/nexus/bin/nexus start
ExecStop=/opt/nexus/bin/nexus stop
User=nexus
Restart=on-abort

[Install]
WantedBy=multi-user.target
```
**✅ 11. Start Nexus**
```
sudo systemctl start nexus
```
**Enable Nexus service at system startup**
```
sudo systemctl enable nexus
```

**check nexus service status**
```
sudo systemctl status nexus
```

**✅ 12. Access Nexus Repository on Web Interface**

```
ufw allow 8081/tcp
```
**Use EC2 publicipaddress:8081**

**Default username is admin**

**To find default password run the below command**
```
sudo vi /opt/sonatype-work/nexus3/admin.password
```
