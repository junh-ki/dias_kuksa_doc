**********************
Step 1: Hardware Setup
**********************

Raspberry-Pi (Data Publisher)
#############################

* For development, you will be using Raspberry-Pi 3 or 4 (preferably 4 since it is faster and has more RAM capacity).

* Raspberry-Pi is not a regular micro-controller but rather a single-board computer. This means that you can run an OS (Operating System; Raspbian, Ubuntu, etc.) on it, and connect it to other IO devices such as monitor, mouse and keyboard. This way, you can use your Raspberry-Pi in the similar way you use your PC, which eases the entire in-vehicle development process.

* You can kick-start with your Raspberry-Pi by following this `instruction <https://projects.raspberrypi.org/en/projects/raspberry-pi-setting-up>`_.

* In this documentation, the following hardware and OS are used. 
    * HW: Raspberry-Pi 4 
    * OS: Raspberry-Pi OS (32-bit) with desktop / `Download <https://www.raspberrypi.org/downloads/raspberry-pi-os/>`_



CAN Interface for Raspberry-Pi
******************************

For Raspberry-Pi to be interactive with CAN, a CAN interface is required. Since Raspberry-Pi doesn't have the in-built CAN interface, user has to configure it manually. There are several ways for configuring the interface on Raspberry-Pi and only three options with different purposes are introduced here.


.. _virtual-can:

CAN Interface Option 1 - Virtual CAN (Only for simululation)
************************************************************

* A virtual CAN interface emulates a physical CAN interface and is capable of behaving nearly identically with less limitations. A virtual CAN interface is appropriate when user just wants to play a CAN log for testing applications in the development phase.

1. Open a terminal and command::

    $ sudo modprobe vcan
    $ sudo ip link add dev vcan0 type vcan
    $ sudo ip link set up vcan0

2. Now you should be able to see the interface, `vcan0`, when commanding::

    $ ifconfig



.. _skpang-pican2:

CAN Interface Option 2 - SKPang PiCan2
**************************************

.. figure:: /_images/hw/pican2.jpg
    :width: 700
    :align: center

* SKPang PiCan2 is a shield that provides a physical CAN interface between Raspberry-Pi and the actual CAN bus. A physical CAN interface is required for a field test.

1. For your Raspberry-Pi to recognize the connected PiCan2, you need to go through a setup process. After physically connecting a PiCan2 shield to your Raspberry-Pi, follow the "Software Installation (p.6)" part of the `instruction <http://skpang.co.uk/catalog/images/raspberrypi/pi_2/PICAN2UG13.pdf>`_ from Raspberry-Pi.

2. When installation is done, open a terminal and confirm whether the `can0` interface is present by commanding::

    $ ifconfig -a

3. If `can0` is shown, configure and bring the interface up by commanding::

    $ sudo ip link set can0 up type can bitrate 500000

* `bitrate` shall be set as the same as the CAN baudrate of the target vehicle.

4. Now you should be able to see the interface, `can0`, when commanding::

    $ ifconfig

5. If you want to bring the interface down, command the following::

    $ sudo ip link set can0 down



.. _seeed-2-channel:

CAN Interface Option 3 - Seeed 2-Channel Shield
***********************************************

.. figure:: /_images/hw/seed_2_channel.png
    :width: 800
    :align: center

* Seeed 2-Channel CAN-BUS(FD) Shield serves the same purpose as SKPang PiCan2 does but with two different CAN interfaces. Because a lot of vehicles use more than one CAN channel, it is required to use a dual-channel shield when data from two different CAN channels need to be analyzed in real-time.

* A detailed setup description can be found `here <https://wiki.seeedstudio.com/2-Channel-CAN-BUS-FD-Shield-for-Raspberry-Pi/#install-can-hat>`_.

1. Get the CAN-HAT source code and install all linux kernel drivers::

    $ git clone https://github.com/seeed-Studio/pi-hats
    $ cd pi-hats/CAN-HAT
    $ sudo ./install.sh 
    $ sudo reboot

2. After the reboot, confirm if `can0` and `can1` interfaces are successfully initialized by commanding::

    $ dmesg | grep spi

3. You should be able to see output like the following::

    [ 3.725586] mcp25xxfd spi0.0 can0: MCP2517 successfully initialized.
    [ 3.757376] mcp25xxfd spi1.0 can1: MCP2517 successfully initialized.

4. Open a terminal and double-check whether the `can0` and `can1` interfaces are present by commanding::

    $ ifconfig -a

5-A. (CAN Classic) If `can0` and `can1` are shown, configure and bring the interfaces up by commanding::

    $ sudo ip link set can0 up type can bitrate 1000000 restart-ms 1000 fd off
    $ sudo ip link set can1 up type can bitrate 1000000 restart-ms 1000 fd off

* `bitrate` shall be set as the same as the CAN baudrate of the target vehicle.

5-B. (CAN FD) If `can0` and `can1` are shown, configure and bring the interface up by commanding::

    $ sudo ip link set can0 up type can bitrate 1000000 dbitrate 2000000 restart-ms 1000 fd on
    $ sudo ip link set can1 up type can bitrate 1000000 dbitrate 2000000 restart-ms 1000 fd on

* `bitrate` shall be set as the same as the CAN baudrate of the target vehicle.

6. If you want to bring the interface down, command the following::

    $ sudo ip link set can0 down
    $ sudo ip link set can1 down



Linux Machine (Data Consumer)
#############################

* A data consumer machine is intended to use the data produced by the connected vehicle's Raspberry-Pi. For development, you can use a virtual machine on your PC that is later expected to be replaceable with a VM instance from cloud service providers to ensure scalability. Please note that it is not required to use virtual machine if the default OS is already Ubuntu.

1. Set up an Ubuntu virtual machine. A detailed tutorial to how to set up Ubuntu with VirtualBox is explained `here <https://brb.nci.nih.gov/seqtools/installUbuntu.html>`_.

    * The image file used (Ubuntu 18.04 LTS - Bionic Beaver) for this documentation can be downloaded `here <http://nl.releases.ubuntu.com/18.04.4/>`_.

2. Open a terminal and install Git on Ubuntu::

    $ sudo apt update
    $ sudo apt install git
    $ git --version
