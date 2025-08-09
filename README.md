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
# Headless JRE
sudo apt install default-jre-headless

# Full JRE
sudo apt install default-jre

# Headless JDK
sudo apt install default-jdk-headless

# Full JDK
sudo apt install default-jdk
```
