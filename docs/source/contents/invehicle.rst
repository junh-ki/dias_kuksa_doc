************************
Step 1: In-vehicle Setup
************************

* The in-vehicle environment here is Raspberry-Pi 4.
* To ease the complexity, Virtual CAN is used here as an interface. Therefore you should follow :ref:`virtual-can` prior to this part.



can-utils
#########

* `can-utils` is a Linux specific set of utilities that enables Linux to communicate with the CAN network on the vehicle. The basic tutorial can be found `here <https://sgframework.readthedocs.io/en/latest/cantutorial.html>`_.

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

2. Recursively clone the kuksa.val repository::

    $ git clone --recursive https://github.com/eclipse/kuksa.val.git

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

5. You can `cmake` now in `kuksa.val/build/`::

    $ cmake ..

6. Then command `make` in `kuksa.val/build/`::

    $ make



kuksa.val - kuksa.val VSS Server Setup
**************************************

1. The kuksa.val server is built based on a Genivi VSS (Vehicle Signal Specification) data structure model. The VSS data structure is created according to the JSON file that is put into the `kuksa-val-server` executable file as an arugment under `--vss` (e.g., `vss_rel_2.0.json`). Before we bring up and run the kuksa.val server, we can create our own VSS data structure in the following steps.

1-1. Recursively clone the GENIVI/vehicle_signal_specification repository::

	$ git clone --recurse-submodules https://github.com/GENIVI/vehicle_signal_specification.git

1-2. The name of the cloned repository folder is `vehicle_signal_specification`. Inside there is a `Makefile` that creates the VSS data structure according to `vehicle_signal_specification/spec`. Since we only need a json file as output, we can modify the Makefile as follow::

    #
    # Makefile to generate specifications
    #

    .PHONY: clean all json

    all: clean json

    DESTDIR?=/usr/local
    TOOLSDIR?=./vss-tools
    DEPLOYDIR?=./docs-gen/static/releases/nightly


    json:
        ${TOOLSDIR}/vspec2json.py -i:spec/VehicleSignalSpecification.id -I ./spec ./spec/VehicleSignalSpecification.vspec vss_rel_$$(cat VERSION).json

    clean:
        rm -f vss_rel_$$(cat VERSION).json
        (cd ${TOOLSDIR}/vspec2c/; make clean)

    install:
        git submodule init
        git submodule update
        (cd ${TOOLSDIR}/; python3 setup.py install --install-scripts=${DESTDIR}/bin)
        $(MAKE) DESTDIR=${DESTDIR} -C ${TOOLSDIR}/vspec2c install
        install -d ${DESTDIR}/share/vss
        (cd spec; cp -r * ${DESTDIR}/share/vss)

    deploy:
        if [ -d $(DEPLOYDIR) ]; then \
            rm -f ${DEPLOYDIR}/vss_rel_*;\
        else \
            mkdir -p ${DEPLOYDIR}; \
        fi;
            cp  vss_rel_* ${DEPLOYDIR}/

1-3. Now we can replace the `vehicle_signal_specification/spec` folder with the modified folder. To get the modified `spec` folder, clone the junh-ki/dias_kuksa repository:: 

    $ git clone https://github.com/junh-ki/dias_kuksa.git

1-4. In the directory, `dias_kuksa/vss_spec_database/`, the `spec` folder can be found. Replace the existing `spec` folder from `vehicle_signal_specification/` with the one from `dias_kuksa/vss_spec_database/`. Designing the `spec` folder's file structure can be easily self-explained.

1-5. Before commanding `make`, install dependencies (anytree, deprecation, stringcase) first::

    $ pip3 install anytree deprecation stringcase

1-6. Navigate to the directory, `vehicle_signal_specification/`, and command `make` to create a new JSON file::

    $ make

1-7. As a result, you can get a JSON file named as `vss_rel_2.0.0-alpha+006.json`. Let's rename this file as `modified.json` for convenience and move it to `kuksa.val/build/src/` where the `kuksa-val-server` executable file is located.

2. Now we can bring up and run the kuksa.val server. Navigate to the directory, `kuksa.val/build/src/`, and command the following::

    $ ./kuksa-val-server --vss modified.json --insecure --log-level ALL




### Work in progress.......... ###




3. The kuksa.val server is built based on a Genivi VSS (Vehicle Signal Specification) data structure model. The VSS data structure is created according to the JSON file, the argument under `--vss`, (`vss_rel_2.0.json`). You can extend or modify this structure in your application during runtime by using `testclient.py`. The followings describe from installing dependencies to running `testclient.py`.

3-1. Install requirements (Python 3.8)::

	$ sudo add-apt-repository ppa:deadsnakes/ppa
	$ sudo apt update
	$ sudo apt install python3.8

3-2. Install requirements (websockets, cmd2, pygments)::

	$ pip3 install websockets cmd2 pygments

3-3. Make sure that the kuksa.val server is up and running, then navigate to the directory, `kuksa.val/vss-testclient/`, and run `testclient.py` with Python 3.8::

	$ python3.8 testclient.py

4. You would be in the VSS Client shell if connected successfully. To get an admin access to the server, you need to assign a json token. Command the following::

	VSS Clinet> authorize ../certificates/jwt/super-admin.json.token
