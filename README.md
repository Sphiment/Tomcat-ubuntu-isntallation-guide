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

aaaaaaaaaaaaaaaaaaaaaaaaaa


```bash
sudo tar xf /tmp/tomcat/apache-tomcat*.tar.gz -C /opt/tomcat
```
