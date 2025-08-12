# Tomcat-ubuntu-isntallation-guide

## Prerequisites
- A user with sudo privileges


## 1) Make sure packages are uptodate 

```bash
sudo apt update && sudo apt upgrade -y
```

## 2) Install Java (JRE/JDK)
Tomcat 9 supports Java 8+. Choose between JRE and JDK according to your needs and you can add the -headless for even lighter build
here are all the builds from the lightest (top) to the heaviest (bottom):

```bash
# Check if Java is already installed and its version
java -version
```

If Java is missing, install one of the packages (choose ONE):
you can also change the default to a specific version like 
```bash
sudo apt install openjdk-21-jdk
```
or you can use the default openjdk:
```bash
# Headless JRE
sudo apt install default-jre-headless

# Full JRE
sudo apt install default-jre

# Headless JDK
sudo apt install default-jdk-headless

# Full JDK
sudo apt install default-jdk
```


## 3) Create a dedicated tomcat user and group
Create a system user and group named 'tomcat'
```bash
# -r: system account, -U: create group, -m: create home dir, -d: home dir, -s: shell
sudo useradd -r -U -m -d /opt/tomcat -s /bin/false tomcat
```

## 4) Download and install Tomcat

```bash
tomcatMajor=9 #choose a major version of tomcat here
tomcatMinor=$(curl --silent "https://dlcdn.apache.org/tomcat/tomcat-${tomcatMajor}/" | grep -oP "v${tomcatMajor}\.\d+\.\d+" | sort -V | tail -n 1 | sed 's/^v//')
wget -P /tmp/tomcat https://dlcdn.apache.org/tomcat/tomcat-${tomcatMajor}/v${tomcatMinor}/bin/apache-tomcat-${tomcatMinor}.tar.gz
```

```bash
sudo tar xf /tmp/tomcat/apache-tomcat-${tomcatMinor}.tar.gz -C /opt/tomcat
```


```bash
sudo ln -s /opt/tomcat/apache-tomcat-${tomcatMinor} /opt/tomcat/latest
```


```bash
sudo chown -R -H tomcat:tomcat /opt/tomcat/latest
sudo chmod -R u+x /opt/tomcat/latest/bin
```

Step 4: Create tomcat service
find your java home directory and choose the one you want tomcat to be installed on:
```bash
sudo update-java-alternatives -l
```
or you can print the latest version of java you have only:
```bash
sudo update-java-alternatives -l | awk '{print $NF}' | tail -n 1
```
then open this file 
```bash
sudo nano /etc/systemd/system/tomcat.service
```
and paste this
```/etc/systemd/system/tomcat.service
[Unit]
Description=Tomcat servlet container
After=network.target

[Service]
Type=forking

User=tomcat
Group=tomcat

Environment="JAVA_HOME=CHANGE ME"
Environment="JAVA_OPTS=-Djava.security.egd=file:///dev/urandom -Djava.awt.headless=true"

Environment="CATALINA_BASE=/opt/tomcat/latest"
Environment="CATALINA_HOME=/opt/tomcat/latest"
Environment="CATALINA_PID=/opt/tomcat/latest/temp/tomcat.pid"
Environment="CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC"

ExecStart=/opt/tomcat/latest/bin/startup.sh
ExecStop=/opt/tomcat/latest/bin/shutdown.sh

[Install]
WantedBy=multi-user.target
```
and you have to change the ```JAVA_HOME``` environment variable to the JAVA_HOME you got from the above command


then reload system daemon
```
sudo systemctl daemon-reload
```
start tomcat
```
sudo systemctl start tomcat
```
check if is it actually active
```
sudo systemctl status tomcat
```
the output should look something like this 
```
● tomcat.service - Tomcat 9 servlet container
     Loaded: loaded (/etc/systemd/system/tomcat.service; enabled; preset: enabled)
     Active: active (running) 
```
Congrats you installed tomcat and created a systemd service for it, now here are some extra configs you will probely need:

making a tomcat user:

in this file  ```/opt/tomcat/latest/conf/tomcat-users.xml```  add these lines:

manager account (has access to Server Status and Manager App):
```
<user username="tomcat_manager" password="s3cret" roles="manager-gui" />
```
admin account (has access to Server Status, Manager App and Host Manager):
```
<user username="tomcat_admin" password="s3cret" roles="admin-gui,manager-gui"/>
```

enable remote access:
in this file ```/opt/tomcat/latest/webapps/manager/META-INF/context.xml``` configure the ```valve``` definition
and add your ip address (```YOUR_IP_ADDRESS```is where your machine's ip address should be)
you can add multiple ip addresses by separting them with a pipe "|" (don't add any spaces)
```
<Context antiResourceLocking="false" privileged="true" >
  <CookieProcessor className="org.apache.tomcat.util.http.Rfc6265CookieProcessor"
                   sameSiteCookies="strict" />
  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1|YOUR_IP_ADDRESS" />
  <Manager sessionAttributeValueClassNameFilter="java\.lang\.(?:Boolean|Integer|Long|Number|String)|org\.apache\.catali></Context>
```
or you can allow unrestricted access from any ip address (less secure) by commenting the ```valve``` definition
```
<Context antiResourceLocking="false" privileged="true" >
  <CookieProcessor className="org.apache.tomcat.util.http.Rfc6265CookieProcessor"
                   sameSiteCookies="strict" />
<!--
  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />
-->
  <Manager sessionAttributeValueClassNameFilter="java\.lang\.(?:Boolean|Integer|Long|Number|String)|org\.apache\.catali></Context>
```
