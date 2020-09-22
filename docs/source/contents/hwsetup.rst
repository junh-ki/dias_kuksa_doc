**************
Hardware Setup
**************

Raspberry Pi
############

- For development, you will be using Raspberry-Pi 3 or 4 (preferably 4 since it is faster and has more RAM capacity).

- Raspberry-Pi is not a regular micro-controller board but rather a single-board computer. This means that you can run an OS (Operating System; Raspbian, Ubuntu, etc.) on it, and connect other devices (such as monitor, keyboard) to it. This way, you can use your RPi in the similar way you use your PC which eases your in-vehicle analysis as well as in-vehicle development.

- You can kick-start with your Raspberry-Pi by following this `instruction <https://projects.raspberrypi.org/en/projects/raspberry-pi-setting-up>`_.

- In this documentation, the following hardware and OS are used. 
    * HW: Raspberry-Pi 4 
    * OS: Raspberry-Pi OS (32-bit) with desktop / `Download <https://www.raspberrypi.org/downloads/raspberry-pi-os/>`_



.. _virtual-can:

CAN Interface Option 1 - Virtual CAN (Only for simululation)
############################################################

1. Open a terminal and command::

    $ sudo modprobe vcan
    $ sudo ip link add dev vcan0 type vcan
    $ sudo ip link set up vcan0

2. Now you should be able to see the interface, `vcan0`, when commanding::

    $ ifconfig



CAN Interface Option 2 - SKPang PiCan2
######################################

.. figure:: /_images/hw/pican2.jpg
    :width: 700
    :align: center

1. SKPang PiCan2 is a CAN shield that gives your Raspberry-Pi a CAN interface. So that you can develop any CAN-related projects with your Raspberry-Pi.

2. For your Raspberry-Pi to recognize the connected PiCan2, you need to go through a setup process. After physically connecting a PiCan2 to your Raspberry-Pi, follow the "Software Installation (p.6 )" part of the `instruction <http://skpang.co.uk/catalog/images/raspberrypi/pi_2/PICAN2UG13.pdf>`_ from your Raspberry-Pi perspective.

3. When installation is done, open a terminal and confirm whether the `can0` interface is present by commanding::

    $ ifconfig -a

4. If `can0` is shown, configure and bring the interface up by commanding::

    $ sudo ip link set can0 up type can bitrate 500000

5. Now you should be able to see the interface, `can0`, when commanding::

    $ ifconfig

6. If you want to bring the interface down, command the following::

    $ sudo ip link set can0 down



CAN Interface Option 3 - Seeed 2 Channel CAN
############################################

.. figure:: /_images/hw/seed_2_channel.png
    :width: 800
    :align: center

* The detailed description can be found `here <https://wiki.seeedstudio.com/2-Channel-CAN-BUS-FD-Shield-for-Raspberry-Pi/#install-can-hat>`_.

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

5-A. (CAN Classic) If `can0` and `can1` are shown, configure and bring the interface up by commanding::

    $ sudo ip link set can0 up type can bitrate 1000000 restart-ms 1000 fd off

5-B. (CAN FD) If `can0` and `can1` are shown, configure and bring the interface up by commanding::

    $ sudo ip link set can0 up type can bitrate 1000000 dbitrate 2000000 restart-ms 1000 fd on

6. If you want to bring the interface down, command the following::

    $ sudo ip link set can0 down
