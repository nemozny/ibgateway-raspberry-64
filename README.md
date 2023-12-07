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

#### Install Debian 64-bit on RPI 4B
Source - https://raspi.debian.net/tested-images/

or a regular RPI OS 64-bit on RPI 5.

&nbsp;

#### Download ibgateway/tws
```
$ wget https://download2.interactivebrokers.com/installers/ibgateway/latest-standalone/ibgateway-latest-standalone-linux-x64.sh
$ wget https://download2.interactivebrokers.com/installers/tws/latest-standalone/tws-latest-standalone-linux-x64.sh
```

&nbsp;

#### Download Oracle JDK
Download [Java SE Development Kit 8uXXX](https://www.oracle.com/java/technologies/downloads/#java8) (Java 8). You may find it towards the end of the page.

Specifically **Linux / ARM64 Compressed Archive**, in my case it was **jdk-8u381-linux-aarch64.tar.gz**.

Yeah, you need to register for an account.

Unpack the JDK somewhere, for example to /opt.
```
$ cd /opt/
$ wget <whatever_url>
$ tar -xf jdk-8u381-linux-aarch64.tar.gz
```

&nbsp;

#### Run the installer
...like this:
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
...while passing your Oracle JDK folder as the "app_java_home" parameter.

You might need to change "sh" to "bash", based on your circumstances.

The same argument applies for the TWS installer.

&nbsp;

#### Running the gateway
I could not make it work **without** [IBC](https://github.com/IbcAlpha/IBC). [IBC](https://github.com/IbcAlpha/IBC) passes some additional arguments to Java and I have always tried to keep my distance from Java.

Download, install and configure your [IBC](https://github.com/IbcAlpha/IBC).

...
out of scope
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

You can try and set JAVA_PATH to our earlier JDK
```
JAVA_PATH=/opt/jdk1.8.0_381/bin
```
but I had no luck with this.

You can always check ibc/logs or Jts/launcher.log for errors and in this case it was:

```
Error: VM option 'UseG1GC' is experimental and must be enabled via -XX:+UnlockExperimentalVMOptions.
Error: Could not create the Java Virtual Machine.
Error: A fatal exception has occurred. Program will exit.
IBC returned exit status 1
autorestart file not found

Gateway finished
```
I have managed to pass "-XX:+UnlockExperimentalVMOptions" to java by editing the ibc/scripts/ibcstart.sh file, but it did not work either. Gateway was complaining about JavaFX and did not start.

&nbsp;

#### Bellsoft Liberica JDK
The solution for me was to download [Bellsoft Liberica JDK](https://bell-sw.com/pages/downloads/), which bundles all modules that IB gateway needed.

I have downloaded JDK 11 LTS / 64-bit / Linux / ARM / Package: Full JDK. For me it was https://download.bell-sw.com/java/11.0.20+8/bellsoft-jdk11.0.20+8-linux-aarch64-full.deb.

Note: I have used JDK 17 LTS for RPI 5.

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
I have encountered a small problem on RPI 4B (no error on RPI 5), but there was a quick fix.

```
$ apt-get install libasound2
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
You might want to run 'apt --fix-broken install' to correct these.
The following packages have unmet dependencies:
 libasound2 : Depends: libasound2-data (>= 1.2.8-1) but it is not going to be installed
E: Unmet dependencies. Try 'apt --fix-broken install' with no packages (or specify a solution).
```
```
$ apt --fix-broken install
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Correcting dependencies... Done
The following additional packages will be installed:
  alsa-topology-conf alsa-ucm-conf libasound2 libasound2-data
Suggested packages:
  libasound2-plugins alsa-utils
The following NEW packages will be installed:
  alsa-topology-conf alsa-ucm-conf libasound2 libasound2-data
0 upgraded, 4 newly installed, 0 to remove and 1 not upgraded.
1 not fully installed or removed.
Need to get 414 kB of archives.
After this operation, 2566 kB of additional disk space will be used.
Do you want to continue? [Y/n]
```
libasound2 unblocked bellsoft-java11-full and it finished its setup.

After a successful installation you can find your new JDK in /usr/lib/jvm/bellsoft-java11-full-aarch64/bin.

&nbsp;

#### Running the gateway once more
Edit your ibc/gatewaystart.sh script and change JAVA_PATH to
```
JAVA_PATH=/usr/lib/jvm/bellsoft-java11-full-aarch64/bin
```
and it worked! For me. Both on RPI 4B and RPI 5.

```
$ cd ibc
$ ./gatewaystart.sh
```
IBC should fire up your gateway after a short delay.

&nbsp;

#### Conclusion for RPI 5

TODO

&nbsp;

#### Conclusion for RPI 4B

I am NOT using Raspberry OS and I am not running a window manager, only xvfb / virtual framebuffer at the moment ("headless" gateway).

TWS is not great, but bearably responsive. Mind you I am connecting over internet to my RPI using xrdp and through a SSH jumpbox on top.

I am running from SD card.

Please let me know if you made it to work easier, better or faster.
