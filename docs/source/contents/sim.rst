******************
Step 4: Simulation
******************

* When everything from step 1 to 3 is set up, you can finally test to see whether or not they communicate each other and work in a correct way.

* If your CAN interface is physical one (e.g., `can0`), you can use either a simulation tool such as Vector CANalyzer, CANoe and etc, or connect to the actual CAN in a vehicle.
	
	* When connected to the actual CAN in a vehicle, you can use `ssh` from your laptop to access your Raspberry-Pi. If your laptop's OS is Windows, you can simply use `Putty <https://www.chiark.greenend.org.uk/~sgtatham/putty/>`_. A video tutorial to how to remotely access your Raspberry-Pi with Putty is available `here <https://youtu.be/IDqQIDL3LKg>`_.

* If your CAN interface is virtual one (e.g., `vcan0`), you can use `canplayer` from the `can-utils` library. Before running `canplayer`, you need to prepare a proper .log file that should be used as an argument to run `canplayer`. The .log file that is used here was originally recorded with CANalyzer in the .asc format, and converted to the .log format with a Python script.

* To record CAN traces with CANalyzer and get the .asc file (that should be converted to the .log format later), you can follow :ref:`can-canalyzer`.

* The following steps describe how to convert a .asc file to a .log file and simulate CAN with the .log file in your Raspberry-Pi. So that you can verify whether or not your setups function correctly from In-vehicle to Cloud.

1. Assuming that all KUKSA components from In-vehicle to Cloud have been set up from the previous steps, 


`canplayer` can be run in the same in-vehicle machine (e.g., your Raspberry-Pi). The following steps describe how to 
