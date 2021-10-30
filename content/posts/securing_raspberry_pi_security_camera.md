---
title: "Securing Raspberry Pi Security Camera (or UIs) using Apache Knox"
date: 2017-06-28
draft: false
tags:
    - Raspberry Pi
keywords:
    - Raspberry Pi
    - Knox
---
I was able to get my hands on the [Raspberry Pi Zero W](https://www.raspberrypi.org/blog/raspberry-pi-zero-w-joins-family/) and decided to put it to use by building a motion activated security camera. I had three simple requirements:
* The camera should have a live feed.
* It should alert me (email/text) when motion is detected.
* It should be secure.

I stumbeled on a great tutorial by Mark West, [Building a Motion Activated Security Camera with Raspberry Pi Zero](https://www.bouvet.no/bouvet-deler/utbrudd/building-a-motion-activated-security-camera-with-the-raspberry-pi-zero). By following the tuorial, I was able to achieve two of my requirements and I was somewhat happy about my setup. There were however some things that I thought needed to be worked out:
* Security
* Network Layout
* Image Detection enhancements

In this blog post I will cover Securing Raspberry Pi Motion Web Interface using Opensource tool called [Apache Knox](http://knox.apache.org/) (disclamer: I am a committer for Apache Knox project)

I am assuming that [Motion](https://motion-project.github.io/) is configure to get live feeds from Raspberry Pi camera module, I will not go into the details as they are covered in detail by Mark West in his excellent blog post mentioned above. It should be noted that any Web Interface can be secured using Apache Knox, I am using Motion as an example here.

### Why Apache Knox ?
* Authentication: Authenticates the users.
* Authorization: Make access decisions i.e. which services are accessible for which user.
* Audit: Gives the ability to determine what actions were taken by whom during some period of time.
* SSL
* Lightweight: Has low memory and disk requirement.

### Setup _user_ and _hostname_
We will create a seperate user for authentication. This user will be created on the system where Apache Knox is installed. Apache Knox uses this user for authentication and access control (using [PAM](http://knox.apache.org/books/knox-0-12-0/user-guide.html#PAM+based+Authentication)) so it makes sense to keep it seperate from other system users. We will use this user exclusively to access our Motion setup from outside.

Setting up user with username - _myuser_ and password - _strongpassword_

```
sudo useradd -m myuser -G sudo
sudo passwd myuser
```

Add user to shadow group
```
sudo usermod -a -G shadow myuser
```

Update hostname so that in multiple node deployments each raspberry pi can be accessed, choose a hostname you like and update in these files

```
# update here
sudo nano /etc/hosts
sudo nano /etc/hostname
```

```
# commit changes
sudo /etc/init.d/hostname.sh
sudo reboot
```

### Setup Apache Knox
Download Apache Knox
```
wget http://www-eu.apache.org/dist/knox/0.12.0/knox-0.12.0.zip
```

Unzip it to /knox directory
```
unzip knox-0.12.0.zip -d /knox
```

Create a link to PAM library under /knox/ext/native
```
cd /knox/knox-0.12.0/ext/native
sudo ln -s /lib/arm-linux-gnueabihf/libpam.so.0.83.1 libpam.so
```

Check gateway.sh
```
nano /knox/knox-0.12.0/bin/gateway.sh
Make sure the APP_JAVA_LIB_PATH parameter is correctly set
APP_JAVA_LIB_PATH="-Djava.library.path=$APP_HOME_DIR/ext/native"
```

_Optional_: Update gateway port from default 8443 to something else in `/knox/conf/gateway-site.xml`

### Update Knox config files
Now we update the knox topology file (sandbox.xml) to use PAM authentication, for Motion running locally on port 8081.
replace `/knox/knox-0.12.0/conf/topologies/sandbox.xml` with `https://github.com/moresandeep/knox4motion/blob/master/conf/sandbox.xml`

_Note_: Update the user (myuser) in authorization provider in `sandbox.xml` file

```
  <provider>
    <role>authorization</role>
    <name>AclsAuthz</name>
    <enabled>true</enabled>
    <param>
        <name>motion.acl</name>
  		  <!-- if you have a different user update here -->
        <value>myuser;*;*</value>
    </param>
  </provider>
```
If Motion is running on another machine then update the following code snippet in sandbox.xml file with the hostname and port where Motion is running.

```
  <service>
          <role>MOTION</role>
          <url>http://<hostname>:<port></url>
  </service>
```

Under `/knox/knox-0.12.0/data/services` create a folder structure as follows
`/knox/knox-0.12.0/data/services/motion/0.0.1/`

Copy `rewrite.xml` and `service.xml` files from `https://github.com/moresandeep/knox4motion/blob/master/services/motion/0.0.1/` to `/knox/data/services/motion/0.0.1/`
i.e.
```
wget https://raw.githubusercontent.com/moresandeep/knox4motion/master/services/motion/0.0.1/rewrite.xml
wget https://raw.githubusercontent.com/moresandeep/knox4motion/master/services/motion/0.0.1/service.xml
```

### Install Java (might not be needed)
Check whether you already have java installed by typing `java -version` if you see something like the following output
```
java version "1.8.0_121"
Java(TM) SE Runtime Environment (build 1.8.0_121-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.121-b13, mixed mode)
```
Then you need not install Java, else run the following command to install Oracle Java 8
```
sudo apt-get update && sudo apt-get install oracle-java8-jdk
```

### Make Knox scripts executable
```
sudo chmod +x /knox/knox-0.12.0/bin/gateway.sh

sudo chmod +x /knox/knox-0.12.0/bin/knoxcl
```

### Create Knox master secret
```
/knox/knox-0.12.0/bin/knoxcli.sh create-master
```
If you are having permissions issues writing to the security directory you can update the permissions as follows
```
sudo chmod -R g=+rwx /knox/knox-0.12.0/bin/../data/security/
```

### Start Apache Knox
To start apache knox run the folowing command
```
/knox/knox-0.12.0/bin/gateway.sh start
```
_Note_: Knox should not be run as root ! if you are having issues with permissions starting knox you can update them as follows
```
sudo chown -R root:users /knox/knox-0.12.0/
```
You can access Knox at `https://<knox-host>:8443/gateway/sandbox/`

Find the relevant Knox logs in `/knox/logs/` directory.


