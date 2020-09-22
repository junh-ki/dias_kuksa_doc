************************
Step 1: In-vehicle Setup
************************

* To ease the complexity, Virtual CAN is used here as an interface. Therefore you should follow :ref:`virtual-can` prior to this part.



can-utils
#########

* `can-utils` is a Linux specific set of utilities that enables Linux to communicate with the CAN network on the vehicle. The basic tutorial can be found `here <https://sgframework.readthedocs.io/en/latest/cantutorial.html>`_

1. Open a terminal and install `can-utils`::

    $ sudo apt install can-utils

2. To test `can-utils`, command the following in the same terminal::

    $ candump vcan0

`candump` allows you to print all data that is being received by a CAN interface (`vcan0` here).

|

3. Open another terminal and command the following::

    $ cansend vcan0 7DF#DEADBEEF

`cansend` sends a CAN message to the corresponding CAN interface (`vcan0` here).

|

4. Confirm whether `candump` has received the CAN message. You should be able to see output like the following in the previous terminal::

    vcan0   7DF    [4]   DE AD BE EF



The kuksa.val Infrastructure
############################

Something
