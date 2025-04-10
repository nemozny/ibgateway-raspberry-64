# ibgateway-raspberry-64
Running Interactive Brokers Gateway on Raspberry 4B + Debian 64-bit or Raspberry 5 + Raspberry Pi OS 64-bit

&nbsp;

Read this and other Raspberry Pi guides here
* [How to get Argon One v2 fan control working on Raspberry 4B and Debian 64](https://nemozny.github.io/argonone-debian-64/)
* [Running Interactive Brokers gateway on Raspberry 4B and Debian 64-bit](https://nemozny.github.io/ibgateway-raspberry-64/)
* [Share your Raspberry Debian 64-bit physical desktop remotely with x11vnc](https://nemozny.github.io/vnc-share-physical-monitor/)

&nbsp;

#### Credits
This thread helped me immensely - https://groups.io/g/twsapi/topic/install_tws_or_ib_gateway_on/25165590

&nbsp;

#### Install OS
* Raspberry 4B - tested on Debian 64-bit from https://raspi.debian.net/tested-images/
* Raspberry 5 - tested on regular RPI OS 64-bit

&nbsp;

#### Download ibgateway/tws
```
$ wget https://download2.interactivebrokers.com/installers/ibgateway/latest-standalone/ibgateway-latest-standalone-linux-x64.sh
$ wget https://download2.interactivebrokers.com/installers/tws/latest-standalone/tws-latest-standalone-linux-x64.sh
```

&nbsp;

#### Bellsoft Liberica JDK
Download [Bellsoft Liberica JDK](https://bell-sw.com/pages/downloads/), which bundles all Java modules that IB gateway needed.

(2025) I have downloaded JDK 17 LTS / 64-bit / Linux / ARM / Package: Full JDK. 

For me it was https://download.bell-sw.com/java/17.0.14+10/bellsoft-jdk17.0.14+10-linux-aarch64-full.deb.

(2023) I have downloaded JDK 11 LTS / 64-bit / Linux / ARM / Package: Full JDK. 

For me it was https://download.bell-sw.com/java/11.0.20+8/bellsoft-jdk11.0.20+8-linux-aarch64-full.deb.

Don't forget to switch to "Full JDK"!

![bellsoft](https://github.com/user-attachments/assets/c011b324-ec14-4ed0-8825-1eb728142b13)


Install it:
```
$ dpkg -i bellsoft-jdk11.0.20+8-linux-aarch64-full.deb
Selecting previously unselected package bellsoft-java11-full.
(Reading database ... 28164 files and directories currently installed.)
Preparing to unpack bellsoft-jdk11.0.20+8-linux-aarch64-full.deb ...
Unpacking bellsoft-java11-full (11.0.20+8) ...
dpkg: dependency problems prevent configuration of bellsoft-java11-full:
 bellsoft-java11-full depends on libasound2; however:
  Package libasound2 is not installed.

dpkg: error processing package bellsoft-java11-full (--install):
 dependency problems - leaving unconfigured
Errors were encountered while processing:
 bellsoft-java11-full
```

After a successful installation you can find your new JDK in /usr/lib/jvm/bellsoft-java11-full-aarch64/bin.

&nbsp;

#### Alternatively download Oracle JDK (Raspberry 4B)
**_As of 2025/04, using Oracle JDK v8 failed to start the TWS installer due to "Unrecognized option: --add-opens" error. 
Later in the process TWS requested "The version of the JVM must be 17.0.10.0.101", so maybe you need to download Oracle JDK v17 instead. I don't know, since I used the Bellsoft JDK for both installation and runtime._**


Download [Java SE Development Kit 8uXXX](https://www.oracle.com/java/technologies/downloads/#java8) (Java 8). You may find it towards the end of the page.

Specifically **Linux / ARM64 Compressed Archive**, in my case it was **jdk-8u381-linux-aarch64.tar.gz**.

Yeah, you need to register for an account.

Unpack the JDK somewhere, for example to /opt. There is no installation.
```
$ cd /opt/
$ wget <whatever_url>
$ tar -xf jdk-8u381-linux-aarch64.tar.gz
```


&nbsp;

#### Run the installer
...like this:
```
$ app_java_home="/usr/lib/jvm/bellsoft-java11-aarch64" sh ibgateway-latest-standalone-linux-x64.sh


```
...while passing your Bellsoft JDK folder as the "app_java_home" parameter.

With Bellsoft JDK, I had to run this installer in Raspberry GUI / Window Manager, not just remotely in the shell, or else it failed looking for some Java GUI components.

With Oracle JDK it is (if unpacked to /opt)
```
$ app_java_home="/opt/jdk1.8.0_381" sh ibgateway-latest-standalone-linux-x64.sh

Starting Installer ...
Welcome to the IB Gateway 10.23 Setup Wizard
This will install IB Gateway 10.23 on your computer. The wizard will lead
you step by step through the installation.

Click Next to continue, or Cancel to exit Setup.
Select the folder where you would like IB Gateway 10.23 to be installed,
then click Next.
Where should IB Gateway 10.23 be installed?
```


You might need to change "sh" to "bash", based on your circumstances.

The same argument applies for the TWS installer.

&nbsp;

#### Running the gateway
I could not make it work **without** [IBC](https://github.com/IbcAlpha/IBC). [IBC](https://github.com/IbcAlpha/IBC) passes some additional arguments to Java and I have always tried to keep my distance from Java.

Download, install and configure your [IBC](https://github.com/IbcAlpha/IBC).

...
IBC configuration is out of scope
...

After you have set up your IBC, do not forget to make all scripts executable. I made that mistake several times.
```
$ cd ibc
$ chmod +x *.sh
$ chmod +x scripts/*.sh
```

Edit ibc/gatewaystart.sh or twsstart.sh and at the head of the file there are some basic configuration parameters:
```
TWS_MAJOR_VRSN=1019
IBC_INI=~/ibc/config.ini
TRADING_MODE=
TWOFA_TIMEOUT_ACTION=exit
IBC_PATH=/opt/ibc
TWS_PATH=~/Jts
TWS_SETTINGS_PATH=
LOG_PATH=~/ibc/logs
TWSUSERID=
TWSPASSWORD=
FIXUSERID=
FIXPASSWORD=
JAVA_PATH=
HIDE=
```

You obviously need to enter your values, such as TWS_MAJOR_VRSN=1023, not 1019.

BTW, you can use these values / scripts to run several gateways in parallel, with different configurations, IB logins and on different ports.

The single most important argument is the **JAVA_PATH**, though.



#### Running the gateway
Edit your ibc/gatewaystart.sh script and change JAVA_PATH to
```
JAVA_PATH=/usr/lib/jvm/bellsoft-java11-full-aarch64/bin
```
or whichever version you have used.

```
$ cd ibc
$ ./gatewaystart.sh
```
IBC should fire up your gateway after a short delay.

