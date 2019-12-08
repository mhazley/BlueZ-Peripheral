# Setting up a BLE Peripheral with BlueZ: Intro

### Get to grips with Bluetooth Low Energy on a Linux Platform

***B***luetooth ***L***ow ***E***nergy (***BLE***, formerly known as Bluetooth Smart) has been around since 2010 when it was integrated into version 4.0 of the Core Bluetooth Specification. 

While possibly a slow starter, it is undeniable the effect the BLE has had on the ***I***nternet ***o***f ***T***hings (***IoT***) industry and today its use can be found in everything from Wearables to Smart-Home devices to location tracking beacons. 

Nowadays, it is fairly common to find a large array of IoT development platforms shipping with Bluetooth hardware on board including, possibly, the most celebrated; the Raspberry Pi.

This series of articles will focus on outlining what is required to bring up a BLE Peripheral with a GATT Server on a Raspberry Pi (or any other Linux based device). 

In the interests of scope, the assumption is made that the reader will already be familiar with the mechanics of BLE, including Central & Peripheral roles as well as GAP and GATT. If not, I cannot recommend [the O’Reilly book](http://shop.oreilly.com/product/0636920033011.do) enough.

##### BlueZ

Seeing as this series of articles will focus on Linux platforms, its important to talk about the all important Bluetooth Stack in Linux, which is called [BlueZ](http://www.bluez.org). This is split between userspace and kernel and provides support for the core Bluetooth layers and protocols. 

In the kernel, BlueZ takes care of device management, L2CAP, RFCOMM and Hardware Drivers (to name just a few) and in userspace, there is `bluetoothd`, which is a central daemon that provides a user interface (via D-Bus), thus reducing exposure to the low level details.

BlueZ also provides a number of essential tools for using Bluetooth in Linux, namely `bluetoothctl`, which is a command line agent, and `btmon` which is a tool to see the commands travelling between the kernel and the Bluetooth hardware. 

##### D-Bus

It is worth taking a moment to talk about D-Bus before we get started because, as mentioned above, this is the type of interface exposed by BlueZ as an API.

D-Bus is essentially an Inter-Process Communication (IPC) and a Remote Procedure Call (RPC) mechanism that allows various programs running on a computer to communicate with each other. This basically means that, as a user, you don't need to care about how things are implemented under the hood, you just need communicate on the bus to call the functions that you want to.

![Processes with D-Bus](https://upload.wikimedia.org/wikipedia/commons/thumb/6/6d/Processes_with_D-Bus.svg/1280px-Processes_with_D-Bus.svg.png)

Going into the depths of how D-Bus works is outside the scope of this series of articles, but it is worth noting that while the examples outlined in the upcoming articles will primarily use Python, there are wrappers for most languages that you would find in use on Linux.

##### Prerequisites 

Before starting, it is important to set up your development environment. (Note, these articles assume debian based).

First, lets make sure BlueZ is installed and up to date. At the time of writing, v5.52 is the latest release but anything over v5.50 will suffice for the examples in these articles.

BlueZ version can be checked by using the`bluetoothctl` command:

```
$ bluetoothctl
Agent registered
[bluetooth]# version
Version 5.50
[bluetooth]# exit
```

If this tool fails to run, or the version reads lower, you'll need to install a newer version of BlueZ. This can be done via package manager or by building and installing from source. First, lets try the package manager.

```
sudo apt-get install -y bluetooth bluez-tools
```

Once complete, check the version again. If this still hasn't got you to at least v5.50, then you can build from source as follows.

First, install some prerequisites to enable you to build BlueZ:

```
sudo apt-get install -y build-essential autoconf glib2.0 libglib2.0-dev libdbus-1-dev libudev-dev libical-dev libreadline-dev
```

Then, grab the latest version of BlueZ and untar:

```
wget https://mirrors.edge.kernel.org/pub/linux/bluetooth/bluez-5.52.tar.xz
tar xf bluez-5.52.tar.xz
cd bluez-5.52
```

Then configure:

```
./configure --prefix=/usr --mandir=/usr/share/man --sysconfdir=/etc --localstatedir=/var --enable-experimental --enable-maintainer-mode --enable-tools
```

Finally, build and install:

```
make && sudo make install
```

First start by installing the following:

```
sudo apt install -y libgirepository1.0-dev libdbus-1-dev libcairo2-dev python-cairo 
```

Then, either reboot, or simply restart the daemon:

```
sudo systemctl daemon-reload
sudo service bluetooth restart
```

You can then check Bluetooth is running by using the following command:

```
service bluetooth status
```

Once you have confirmed BlueZ is working, next step is to simply set up the environment for the examples in these articles.

Firstly install:

```
sudo apt install -y libgirepository1.0-dev libdbus-1-dev libcairo2-devbl python-cairo
```

Then, the examples assume a Python v3.7 environment, with the following libs:

```
pip install PyGObject==3.30.4 dbus-python
```

