.. _can-traces:

*****************************
Reference: Logging CAN traces
*****************************

* It would be tedius if you have to get inside the target vehicle, set up the simulation environment and test everytime there is a new update on your implementation. Which is why having a CAN trace log that is properly logged is important because it eases the development process. With a log file, you can develop and test your application on your desk without having to go to the vehicle and this would save time and energy unnecessarily spent on setting up the environment.

* Although `canplayer` from the `can-utils` library in Raspberry-Pi is compliant only with the .log format, it is recommended to use Vector tools to get CAN traces since they can provide the traces in a variety of formats such that the traces can be used with not only `canplayer` but also several other means in different environments. Therefore two ways, with and without Vector tools, to log CAN traces in the target vehicle are introduced in this part.



Option A: with Vector Tools
===========================

Hardware Prerequisites
**********************

* Laptop installed with Vector Software
* Licensed Vector CANcase (VN1630 is used here)
* USB Interface for CAN and I/O (comes with CANcase)
* CAN cable (D-sub /D-sub) x 1
* CAN adapter (Open cable to D-sub) x 1



Software Prerequisites
**********************

* Vector Software (CANalyzer Version 13.0 SP2 is used here)


Logging with Vector Tools
*************************

1. Connect the Vector CANcase to the CAN H/L from an ECU in the vehicle by using the CAN cable and adapter. For this you also need to refer to the ECU hardware specification to get to know which ports of the ECU stand for CAN-high and -low of what number of CAN channel.

2. Connect the Vector CANcase to your laptop and check if the device manager recognizes the CANcase.

.. figure:: /_images/canalyzer/0-device_manager.PNG
    :width: 700
    :align: center

* Because the CANcase used here is Vector VN1630, it shows the exact name of the CANcase.

3. Run CANalyzer 13.0 SP2.

.. figure:: /_images/canalyzer/1-license.PNG
    :width: 700
    :align: center

* The capture shows when your CANcase is properly licensed with `CANalyzer PRO 13.0`. Press "OK" to proceed.

.. figure:: /_images/canalyzer/2-license.PNG
    :width: 700
    :align: center

* The capture shows when your CANcase is not licensed. You can not proceed further in this case.

4. The first thing you would see in CANalyzer is the "Trace" tab. Here you can see the incoming CAN traces when they are being read.

.. figure:: /_images/canalyzer/3-trace.PNG
    :width: 700
    :align: center

5. To synchronize your CANcase with the target vehicle's baudrate, you have to configure manually in CANalyzer. To do this, switch to the "Configuration" tab.

.. figure:: /_images/canalyzer/4-configuration.PNG
    :width: 700
    :align: center

6. When you double-click the CANcase icon, a window named "Network Hardware Configuration" would show up. Select the CAN channel (VN1630: written on the back side of CANcase) that you connected to the CAN ports of the target vehicle and set the baudrate the same as that of the vehicle. Then click "OK".

.. figure:: /_images/canalyzer/5-configuration_baudrate.PNG
    :width: 700
    :align: center

7. 

.. figure:: /_images/canalyzer/6-logging.PNG
    :width: 700
    :align: center

8.

.. figure:: /_images/canalyzer/7-logging.PNG
    :width: 700
    :align: center

9.

.. figure:: /_images/canalyzer/8-logformat.PNG
    :width: 700
    :align: center

10.

.. figure:: /_images/canalyzer/9-start.PNG
    :width: 700
    :align: center



Option B: with Raspberry-Pi and CAN Shield
==========================================

Hardware Prerequisites
**********************





Logging with Raspberry-Pi and CAN Shield
****************************************



candump -l vcan0


