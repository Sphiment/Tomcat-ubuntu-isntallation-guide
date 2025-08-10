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
sudo ln -s /opt/tomcat/apache-tomcat-9.0.27 /opt/tomcat/latest
```


```bash
sudo chown -R -H tomcat: /opt/tomcat/latest
sudo sh -c 'chmod +x /opt/tomcat/latest/bin/*.sh'
```


Step 4: Create tomcat service
```bash
sudo nano /etc/systemd/system/tomcat.service
```
and paste this

```
[Unit]
Description=Tomcat 9 servlet container
After=network.target

[Service]
Type=forking

User=tomcat
Group=tomcat

Environment="JAVA_HOME=/usr/lib/jvm/default-java"
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
