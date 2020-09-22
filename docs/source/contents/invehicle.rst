************************
Step 1: In-vehicle Setup
************************

* The in-vehicle environment here is Raspberry-Pi 4.
* To ease the complexity, Virtual CAN is used here as an interface. Therefore you should follow :ref:`virtual-can` prior to this part.



can-utils
#########

* `can-utils` is a Linux specific set of utilities that enables Linux to communicate with the CAN network on the vehicle. The basic tutorial can be found `here <https://sgframework.readthedocs.io/en/latest/cantutorial.html>`_

1. Open a terminal and install `can-utils`::

    $ sudo apt install can-utils

2. To test `can-utils`, command the following in the same terminal::

    $ candump vcan0

`candump` allows you to print all data that is being received by a CAN interface, `vcan0`.

3. Open another terminal and command the following::

    $ cansend vcan0 7DF#DEADBEEF

`cansend` sends a CAN message, `7DF#DEADBEEF` to the corresponding CAN interface, `vcan0`.

4. Confirm whether `candump` has received the CAN message. You should be able to see output like the following in the previous terminal::

    vcan0   7DF    [4]   DE AD BE EF



kuksa.val Infrastructure
########################

1. Install Git::

    $ sudo apt install git

2. Clone the kuksa.val repository::

    $ git clone https://github.com/eclipse/kuksa.val.git

3. Make a `build` folder inside the repository and navigate to the folder::

    $ cd kuksa.val
    $ mkdir build
    $ cd build

4. The following commands should be run before `cmake` to avoid possible errors.

4-1. Install `cmake` if it does not exist::

    $ sudo apt-get update && sudo apt-get upgrade
    $ sudo apt install cmake

* If you are using Ubuntu 18.04.4 and not Raspberry-Pi 4, follow this `description <https://www.claudiokuenzler.com/blog/796/install-upgrade-cmake-3.12.1-ubuntu-14.04-trusty-alternatives>`_.

4-2. Install Boost libraries::

    $ sudo apt-get install cmake libblkid-dev e2fslibs-dev libboost-all-dev libaudit-dev

4-3. Install OpenSSL::

    $ sudo apt-get install libssl-dev

4-4. Install Mosquitto library::

    $ sudo apt-get update
    $ sudo apt-get install libmosquitto-dev

5. You can `cmake` now from `kuksa.val/build/`::

    $ cmake ..

6. The following commands should be run before `make` to avoid possible errors.

6-1. `jsoncons` library should be included. Download the `latest release <https://github.com/danielaparker/jsoncons/releases/tag/v0.156.1>`_.

6-2. Unpack the zip file and copy the directory, `jsoncons-x.xxx.x/include/*`, to your '/usr/include/' directory::

    $ sudo cp -R jsoncons-x.xxx.x/include/* /usr/include/

6-3. `jwt-cpp` should also be included. Download the `latest release <https://github.com/Thalhammer/jwt-cpp/releases/tag/v0.4.0>`_.

6-4. Unpack the zip file and copy the directory, `jwt-cpp-x.x.x/include/*`, to your `/usr/include/` directory::

    $ sudo cp -R jwt-cpp-x.x.x/include/* /usr/include/

6-5. The `jsonpath` directory from `kuksa.val/src/SubscriptionHandler.cpp`, `kuksa.val/src/VssDatabase.cpp` and `kuksa.val/src/main.cpp` should be modified::

    #include <jsonpath/json_query.hpp>   =>   #include <jsoncons_ext/jsonpath/json_query.hpp>

7. Then command `make` from `kuksa.val/build/`::

    $ make
