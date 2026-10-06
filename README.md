# ibgateway-raspberry-64
Running Interactive Brokers Gateway on Raspberry 4B + Debian 64-bit or Raspberry 5 + Raspberry Pi OS 64-bit

&nbsp;

Read this and other Raspberry Pi guides here
* [How to get Argon One v2 fan control working on Raspberry 4B and Debian 64](https://nemozny.github.io/argonone-debian-64/)
* [Running Interactive Brokers gateway on Raspberry 4B and Debian 64-bit](https://nemozny.github.io/ibgateway-raspberry-64/)
* [Share your Raspberry Debian 64-bit physical desktop remotely with x11vnc](https://nemozny.github.io/vnc-share-physical-monitor/)

&nbsp;

### Credits
This thread helped me immensely - https://groups.io/g/twsapi/topic/install_tws_or_ib_gateway_on/25165590

&nbsp;

### Install OS
* Raspberry 4B - tested on Debian 64-bit from https://raspi.debian.net/tested-images/
* Raspberry 5 - tested on regular RPI OS 64-bit

&nbsp;

### Download ibgateway/tws
There is a new [download portal](https://www.interactivebrokers.com/en/trading/download-tws.php?p=offline-latest).

Direct links:
```
$ wget https://download2.interactivebrokers.com/installers/ibgateway/latest-standalone/ibgateway-latest-standalone-linux-x64.sh
$ wget https://download2.interactivebrokers.com/installers/tws/latest-standalone/tws-latest-standalone-linux-x64.sh
```

&nbsp;

### Bellsoft Liberica JDK
Download [Bellsoft Liberica JDK](https://bell-sw.com/pages/downloads/), which bundles all Java modules that IB gateway needed.

* Select version "JDK 25 LTS"
* Linux
* Under Linux select ARM
* Dropdown "Package" select "Full JDK"
* Download .DEB installer

In 2026 for TWS/IBGateway versions 10.50+ you need Liberica JDK 25 - for me it was [bellsoft-jdk25.0.4.1+1-linux-aarch64-full.deb](https://download.bell-sw.com/java/25.0.4.1+1/bellsoft-jdk25.0.4.1+1-linux-aarch64-full.deb).  
In 2025 I have downloaded JDK 17 LTS / 64-bit / Linux / ARM / Package: Full JDK - for me it was [bellsoft-jdk17.0.14+10-linux-aarch64-full.deb](https://download.bell-sw.com/java/17.0.14+10/bellsoft-jdk17.0.14+10-linux-aarch64-full.deb).  
In 2023 I have downloaded JDK 11 LTS / 64-bit / Linux / ARM / Package: Full JDK - for me it was [bellsoft-jdk11.0.20+8-linux-aarch64-full.deb](https://download.bell-sw.com/java/11.0.20+8/bellsoft-jdk11.0.20+8-linux-aarch64-full.deb).  

Don't forget to switch to "Full JDK"!

![bellsoft](https://github.com/user-attachments/assets/c011b324-ec14-4ed0-8825-1eb728142b13)


Install it:
```
$ sudo dpkg -i bellsoft-jdk25.0.4.1+1-linux-aarch64-full.deb
Selecting previously unselected package bellsoft-java25-full.
(Reading database ... 226787 files and directories currently installed.)
Preparing to unpack bellsoft-jdk25.0.4.1+1-linux-aarch64-full.deb ...
Unpacking bellsoft-java25-full (25.0.4.1+1) ...
Setting up bellsoft-java25-full (25.0.4.1+1) ...
update-alternatives: using /usr/lib/jvm/bellsoft-java25-full-aarch64/bin/jar to provide /usr/bin/jar (jar) in auto mode
... omitted ...
update-alternatives: using /usr/lib/jvm/bellsoft-java25-full-aarch64/bin/serialver to provide /usr/bin/serialver (serialver) in auto mode
```

After a successful installation you can find your new JDK in /usr/lib/jvm/bellsoft-java25-full-aarch64/bin.

&nbsp;

Update October 2025: Back in 2023 I had to run the installer using OpenJDK and then run the actual gateway/TWS using Bellsoft Java. That was no longer necessary in 2025, you can use Bellsoft for both. If you run into problems with the installer, you can try to install with OpenJDK (ie. app_java_home="/opt/jdk1.8.0_441") and then run with Bellsoft.

&nbsp;

### Run the Gateway/TWS installer
Updates in 2026:
Installer was stubbornly using the bundled JRE, not the one I provided with Bellsoft Java. I needed to explicitly disable the bundled JRE using "INSTALL4J_DISABLE_BUNDLED_JRE=true".  
Then it was trying to run the GUI installer, although I was in CLI. Disable GUI with "-c".  

Run the installer like this:
```
$ INSTALL4J_DISABLE_BUNDLED_JRE=true  app_java_home="/usr/lib/jvm/bellsoft-java25-full-aarch64" sh ./tws-latest-standalone-linux-x64.sh -c
```
...while passing your Bellsoft JDK folder as the "app_java_home" parameter, disabling the bundled JRE and enabling the command line mode.

You might need to change "sh" to "bash", based on your circumstances.

The same applies to the TWS installer.

&nbsp;

### Running TWS / Gateway in GUI
Installers create desktop shortcuts by default. To start the app using the shortcut, prepend
```
env app_java_home="/usr/lib/jvm/bellsoft-java25-full-aarch64"
```
to the shortcut command. For example the full command will look like this
```
env app_java_home="/usr/lib/jvm/bellsoft-java25-full-aarch64" "/home/nemozny/Jts/1051/tws" -J-DjtsConfigDir="/home/nemozny/Jts" %U
```

### Configuring the gateway in headless mode
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


Edit your ibc/gatewaystart.sh script and change JAVA_PATH to
```
JAVA_PATH=/usr/lib/jvm/bellsoft-java25-full-aarch64/bin
```
or whichever version you have used.

### Running the gateway

```
$ cd ibc
$ ./gatewaystart.sh
```
IBC should fire up your gateway after a short delay.

&nbsp;

### Start the gateway from a script or cron
In my python scripts I am starting the gateway before any actual processing starts. However the gateway/TWS can only run in GUI, it cannot run headless, in shell. For graphical environment you can use whatever GNOME / KDE came with your desktop. On server without a physical screen you can replace X server with "xvfb" - Virtual Framebuffer 'fake' X server. There are plenty of HOWTO's over the internet.

In any case, if you execute your scripts from a remote shell, not from shell on your physical/virtual screen, you need to pass the DISPLAY variable to project the application on the X display. You also need to avoid the modern Wayland, since Wayland is a communication protocol and not a server like X11, so AFAIK you can NOT do the above with Wayland. Wayland does not provide the DISPLAY variable and WAYLAND_DISPLAY does not work.

For further reading see [https://github.com/nemozny/vnc-share-physical-monitor](https://github.com/nemozny/vnc-share-physical-monitor).

Update your IBC scripts (twsstart.sh and gatewaystart.sh), at the very end of the script you can see:
```
if [[ "$1" == "-inline" ]]; then
    exec "${IBC_PATH}/scripts/displaybannerandlaunch.sh"
else
    title="IBC ($APP $TWS_MAJOR_VRSN)"
    xterm $iconic -T "$title" -e "${IBC_PATH}/scripts/displaybannerandlaunch.sh" &
fi
```
First make sure you actually have "xterm". Change xterm for whatever GUI console app you had installed. It happened to me I had "lxterminal" and no "xterm" on a fresh install.

Add "DISPLAY=:0" to the second to last line:
```
    DISPLAY=:0 xterm $iconic -T "$title" -e "${IBC_PATH}/scripts/displaybannerandlaunch.sh" &
fi
```
This will run the app in your X session.

You can then kill the app by running a shell command "pkill java".

