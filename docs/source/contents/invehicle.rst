************************
Step 1: In-vehicle Setup
************************

.. figure:: /_images/invehicle/invehicle_schema.png
    :width: 1200
    :align: center

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

.. figure:: /_images/invehicle/invehicle_schema_server.png
    :width: 1200
    :align: center

1. The kuksa.val server is built based on a Genivi VSS (Vehicle Signal Specification) data structure model. The VSS data structure is created according to the JSON file that is put into the `kuksa-val-server` executable file as an arugment under `--vss` (e.g., `vss_rel_2.0.json`). Before we bring up and run the kuksa.val server, we can create our own VSS data structure in the following steps.

1-1. Recursively clone the GENIVI/vehicle_signal_specification repository::

	$ git clone --recurse-submodules https://github.com/GENIVI/vehicle_signal_specification.git

1-2. The name of the cloned repository folder is `vehicle_signal_specification`. Inside there is a `Makefile` that creates the VSS data structure according to `vehicle_signal_specification/spec`. Since we only need a JSON file as output, we can modify the Makefile as follow::

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

1-3. Now we can replace the `vehicle_signal_specification/spec` folder with the modified folder. To get the modified `spec` folder, clone the `junh-ki/dias_kuksa` repository:: 

    $ git clone https://github.com/junh-ki/dias_kuksa.git

1-4. In the directory, `dias_kuksa/utils/in-vehicle/vss_structure_example/`, the `spec` folder can be found. Replace the existing `spec` folder from `vehicle_signal_specification/` with the one from `dias_kuksa/utils/in-vehicle/vss_structure_example/`. Designing the `spec` folder's file structure can be easily self-explained.

1-5. Before commanding `make`, install python dependencies (anytree, deprecation, stringcase) first::

    $ pip3 install anytree deprecation stringcase

1-6. Navigate to the directory, `vehicle_signal_specification/`, and command `make` to create a new JSON file::

    $ make

1-7. As a result, you can get a JSON file named as `vss_rel_2.0.0-alpha+006.json`. Let's rename this file as `modified.json` for convenience and move it to `kuksa.val/build/src/`, where the `kuksa-val-server` executable file is located.

2. Now we can bring up and run the kuksa.val server. Navigate to the directory, `kuksa.val/build/src/`, and command the following::

    $ ./kuksa-val-server --vss modified.json --insecure --log-level ALL

* The kuksa.val server is entirely passive. Which means that you would need supplementary applications to feed and fetch the data. In the following `dbcfeeder.py` and `cloudfeeder.py` are introduced. They are meant to deal with setting and getting the data from the kuksa.val server.


##### WORK IN PROGRESS ... #####

3. :blue:`(Optional / You can proceed without these steps if you just want to use the VSS structure as is.)` You can extend or modify the existing VSS data structure during runtime by using `kuksa.val/vss-testclient/testclient.py`. The followings describe from installing python dependencies, running `testclient.py` to extending or modifying the VSS structure.

3-1. Install requirements (Python 3.8)::

	$ sudo add-apt-repository ppa:deadsnakes/ppa
	$ sudo apt update
	$ sudo apt install python3.8

3-2. Install requirements (websockets, cmd2, pygments)::

	$ pip3 install websockets cmd2 pygments

3-3. Make sure that the kuksa.val server is already up and running, then navigate to the directory, `kuksa.val/vss-testclient/`, and run `testclient.py` with Python 3.8::

	$ python3.8 testclient.py

3-4. If connected to the server successfully, you would be in the VSS Client shell. To get an admin access to the server, you need to assign a JSON token. Command the following::

	VSS Clinet> authorize ../certificates/jwt/super-admin.json.token

3-5. 
authorize ../
getMetaData Vehicle.Speed
setValue Vehicle.Speed 200
setValue Vehicle.Private.ThurstersActive true

shall cat modified.json
updateVSSTree ../build/src/modified.json

getMetaData Vehicle.Speed
getValue Vehicle.Speed
setValue Vehicle.Private.ThurstersActive true
getValue Vehicle.Private.ThurstersActive



kuksa.val - dbcfeeder.py Setup
******************************

.. figure:: /_images/invehicle/invehicle_schema_dbcfeeder.png
    :width: 1200
    :align: center

* `kuksa.val/examples/dbc2val/dbcfeeder.py` is to interpret and write the CAN data that is being received by the CAN interface (e.g., `can0` or `vcan0`) to the kuksa.val server.

* `dbcfeeder.py` takes four compulsory arguments to be run:
	* CAN interface (e.g., `can0` or `vcan0`) / `-d` or `--device` / To connect to the CAN device interface.
	* JSON token (e.g., `super-admin.json.token`) / `-j` or `--jwt` / To have write-access to the server.
	* DBC file (e.g., `dbcfile.dbc`) / `--dbc` / To translate the raw CAN data.
	* Mapping YML file (e.g., `mapping.yml`) / `--mapping` / To map each of the specific signals to the corresponding path in the kuksa.val server.

* Since the kuksa.val work package already has the admin JSON token, you only need a DBC file and a YML file. The `junh-ki/dias_kuksa` repository provides the example DBC file and YML file. :blue:`(DBC file is target-vehicle-specific and can be offered by the target vehicle's manufacturer.)`

1. Assuming you have already cloned the `junh-ki/dias_kuksa` repository, / If you haven't, please clone it now::

	$ git clone https://github.com/junh-ki/dias_kuksa.git

2. Navigate to the directory, `dias_kuksa/utils/in-vehicle/dbcfeeder_example_arguments/`, and copy `dias_mapping.yml` and `dias_simple.dbc` to the directory, `kuksa.val/examples/dbc2val/`, where `dbcfeeder.py` is located.

3. Before running `dbcfeeder.py`, install python dependencies (python-can cantools serial) first::

	$ pip3 install python-can cantools serial

4. If you haven't brought up a virtual CAN interface, `vcan0`, please do it now by following :ref:`virtual-can`.

5. Navigate to the directory, `kuksa.val/examples/dbc2val/`, and command the following::

	$ python3 dbcfeeder.py -d vcan0 -j ../../certificates/jwt/super-admin.json.token --dbc dias_simple.dbc --mapping dias_mapping.yml



kuksa.val - cloudfeeder.py Setup
********************************

.. figure:: /_images/invehicle/invehicle_schema_cloudfeeder.png
    :width: 1200
    :align: center

* `dias_kuksa/utils/in-vehicle/cloudfeeder_telemetry/cloudfeeder.py` fetches the data from the kuksa.val in-vehicle server and preprocess it with a case-specific preprocessor, `dias_kuksa/utils/in-vehicle/cloudfeeder_telemetry/preprocessor_bosch.py`, and transmit the result to the cloud instance (MQTT broker in DIAS use-case).

* `cloudfeeder.py` takes six compulsory arguments to be run:
	* Host URL (e.g., "mqtt.bosch-iot-hub.com") / `--host`
	* Protocol Port Number (e.g., "8883") / `-p` or `--port`
	* Credential Authorization Username (Configured when creating) (e.g., "{username}@{tenant-id}") / `-u` or `--username`
	* Credential Authorization Password (Configured when creating) (e,g., "your_pw")/ `-P` or `--password`
	* Server Certificate File (MQTT TLS Encryption) (e.g., "iothub.crt") / `-c` or `--cafile`
	* Data Type (e.g., "telemetry" or "event") / "-t" or "--type"

1. (Optional) `preprocessor_bosch.py` is designed to follow Bosch's diagnostic methodologies. Therefore you can create your own `preprocessor_xxx.py` that follows your purpose and replace `preprocessor_bosch.py` with it. Of course, the corresponding lines in `cloudfeeder.py` should be modified as well in this case.

2. Navigate to `dias_kuksa/utils/in-vehicle/cloudfeeder_telemetry/`, copy `cloudfeeder.py` and `preprocessor_bosch.py` (or your own `preprocessor_xxx.py`) to `kuksa.val/vss-testclient/`, where the `testclient.py` file is located.

3. Then the `do_getValue(self, args)` function from `kuksa.val/vss-testclient/testclient.py` should be modified as below.

.. code-block:: python

	...

	def do_getValue(self, args):
        """Get the value of a parameter"""
        req = {}
        req["requestId"] = 1234
        req["action"]= "get"
        req["path"] = args.Parameter
        jsonDump = json.dumps(req)
        self.sendMsgQueue.put(jsonDump)
        resp = self.recvMsgQueue.get()
        # print(highlight(resp, lexers.JsonLexer(), formatters.TerminalFormatter()))
        self.pathCompletionItems = []
        datastore = json.loads(resp)
        return datastore

    ...

2. Due to its dependency on the cloud instance information, you should create either a Eclipse Hono or a Bosch-IoT-Hub instance first by following :ref:`cloud-hono`, so that you can get the required information ready to run `cloudfeeder.py`.

3. When all the required information is ready, navigate to the directory, `kuksa.val/vss-testclient/`, and run `cloudfeeder.py` by commanding::

	$ python3 cloudfeeder.py --host {host_url} -p {port_number} -u {username}@{tenant-id} -P {password} -c {server_certificate_file} -t {transmission_type}

* Just a reminder, the information between `{}` should be different depending on the target Hono instance.
