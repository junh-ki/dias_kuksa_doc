******************
Step 4: Simulation
******************

* When everything from step 1 to 3 is set up, you can finally test to see whether or not they communicate each other and work in a correct way.

* If your CAN interface is physical one (e.g., `can0`), you can use either a simulation tool such as Vector CANalyzer, CANoe and etc, or connect to the actual CAN in a vehicle.
	
	* When connected to the actual CAN in a vehicle, you can use `ssh` from your laptop to access your Raspberry-Pi. If your laptop's OS is Windows, you can simply use `Putty <https://www.chiark.greenend.org.uk/~sgtatham/putty/>`_. A video tutorial to how to remotely access your Raspberry-Pi with `Putty <https://www.chiark.greenend.org.uk/~sgtatham/putty/>`_ is available `here <https://youtu.be/IDqQIDL3LKg>`_.

* If your CAN interface is virtual one (e.g., `vcan0`), you can use `canplayer` from the `can-utils` library. Before running `canplayer`, you need to prepare a proper .log file that should be used as an argument to run `canplayer`. The .log file that is used here was originally logged with CANalyzer in the .asc format, and converted to the .log format with a Python script.

* To log CAN traces with CANalyzer and get the .asc file (that should be converted to the .log format later) or get the .log file directly with Raspberry-Pi tools, you can follow :ref:`can-traces`.

* The following steps describe how to convert a .asc file to a .log file and simulate CAN with the .log file in your Raspberry-Pi. So that you can verify whether or not your setups function correctly from In-vehicle to Cloud.



asc2log Conversion
##################

1. Make sure that all KUKSA components from In-vehicle to Cloud have been set up from the previous steps.

2. `canplayer` can be run in the same in-vehicle machine (e.g., your Raspberry-Pi). Therefore you should be in your Raspberry-Pi to proceed further.

3. Navigate to `dias_kuksa/utils/canplayer/` where `asc2log_channel_separator.py` with two .asc files are located.

4. `otosan_can0-30092020.asc` is logged with CAN channel 0 while `otosan_can2-30092020.asc` with channel 2 from the Ford Otosan truck.

5. Since `canplayer` can not play a .asc file, you have to convert them to the .log format. You can do this conversion with `asc2log_channel_separator.py`.

6. As the discription of `asc2log_channel_separator.py` states, the script not only performs the `asc2log` conversion but also separates the result by CAN channel in case the target .asc file has traces from more than one CAN channel. If the target .asc file has traces from only one CAN channel, the script would only produces one result .log file.

7. Prior to running `asc2log_channel_separator.py`, the `can-utils` library should be installed first. If you have followed the steps from the beginning, you have already installed this library from :ref:`can-utils`.

8. To convert `otosan_can0-30092020.asc`, navigate to `dias_kuksa/utils/canplayer/` and command the following::

	$ python3 asc2log_channel_separator.py --asc otosan_can0-30092020.asc --can vcan0

* `vcan0` should be already configured before running this command. If you haven't, please follow :ref:`virtual-can` and run the command above.

9. As a result from 8, `can0_otosan_can0-30092020.log` would be created.



Simulation with canplayer
#########################

1. Now that we have the .log file to play, make sure your in-vehicle components are already up and running.

* Configuring `vcan0` with `kuksa-val-server.exe` and `dbcfeeder.py` is mandatory, `cloudfeeder.py` and other cloud-side components are optional here.

2. To run `canplayer` with the target .log file, `can0_otosan_can0-30092020.log`, navigate to `dias_kuksa/utils/canplayer/`, where the .log file is located, and command the following::

	$ canplayer -I can0_otosan_can0-30092020.log

* You should be able to see signals being updated on both terminals, `kuksa-val-server.exe` and `dbcfeeder.py`, as shown in the screenshots below.

.. figure:: /_images/canplayer/canplayer_terminal.png
    :width: 700
    :align: center

* Although the screenshots are taken in an Ubuntu virtual machine for convenience, the environment for this simulation is meant to be Raspberry-Pi.
