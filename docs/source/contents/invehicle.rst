************************
Step 2: In-vehicle Setup
************************

.. figure:: /_images/invehicle/invehicle_schema.png
    :width: 1200
    :align: center

* The in-vehicle environment here is Raspberry-Pi 4.
* To ease the complexity, a virtual CAN interface, `vcan0`, is used here. Therefore you should follow :ref:`virtual-can` prior to this part.



.. _can-utils:

can-utils
#########

* `can-utils` is a Linux specific set of utilities that enables Linux to communicate with the CAN network on the vehicle. The basic tutorial can be found `here <https://sgframework.readthedocs.io/en/latest/cantutorial.html>`_.

1. Open a terminal and install `can-utils`::

    $ sudo apt install can-utils

2. To test `can-utils`, command the following in the same terminal::

    $ candump vcan0

* `candump` allows you to print all data that is being received by a CAN interface, `vcan0`, on the terminal.

3. Open another terminal and command the following::

    $ cansend vcan0 7DF#DEADBEEF

* `cansend` sends a CAN message, `7DF#DEADBEEF` to the corresponding CAN interface, `vcan0`.

4. Confirm whether `candump` has received the CAN message. You should be able to see output like the following on the previous terminal::

    vcan0   7DF    [4]   DE AD BE EF



kuksa.val Infrastructure
########################

1. Install Git::

    $ sudo apt install git

2. Recursively clone the `kuksa.val` repository::

    $ git clone --recursive https://github.com/eclipse/kuksa.val.git

3. Make a folder named `build` inside the `kuksa.val` repository folder and navigate to `kuksa.val/build/`::

    $ cd kuksa.val
    $ mkdir build
    $ cd build

4. The following commands should be run before `cmake` to avoid possible errors.

4-1. Install `cmake` (version 3.12 or higher) if it hasn't been installed::

    $ sudo apt-get update && sudo apt-get upgrade

A. Raspberry-Pi::

    $ sudo apt install cmake

B. Ubuntu::

    $ sudo snap install cmake --classic

4-2. Install dependencies (Boost libraries, OpenSSL, Mosquitto and more)::

    $ sudo apt-get install libblkid-dev e2fslibs-dev libboost-all-dev libaudit-dev libssl-dev mosquitto libmosquitto-dev libglib2.0-dev

5. You can `cmake` now. Navigate to `kuksa.val/build/` and command the following::

    $ cmake ..

6. Then command `make` in the same directory::

    $ make

If succeeded, you have successfully built the `kuksa.val` infrastructure.



kuksa.val - kuksa.val VSS Server Setup
**************************************

.. figure:: /_images/invehicle/invehicle_schema_server.png
    :width: 1200
    :align: center

1. The `kuksa.val` server is built based on the Genivi VSS (Vehicle Signal Specification) data structure model. The VSS data structure is created according to the JSON file that is put into the `kuksa-val-server` executable as an arugment under `--vss` (e.g., `vss_rel_2.0.json`). Before we bring up and run the `kuksa.val` server, we can create our own VSS data structure in the following steps.

1-1. Recursively clone the GENIVI/vehicle_signal_specification repository::

	$ git clone --recurse-submodules https://github.com/GENIVI/vehicle_signal_specification.git

1-2. The name of the cloned repository folder is `vehicle_signal_specification`. Inside there is a `Makefile` that creates the VSS data structure according to `vehicle_signal_specification/spec`. Since we only need a JSON file as output, we can modify `Makefile` as follow::

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

* Please note that it is recommended to modify the file manually since `Makefile` is `tab`-sensitive.

1-3. Now we can replace the `vehicle_signal_specification/spec` folder with the modified folder. To get the modified `spec` folder, clone the `junh-ki/dias_kuksa` repository:: 

    $ git clone https://github.com/junh-ki/dias_kuksa.git

1-4. In the directory, `dias_kuksa/utils/in-vehicle/vss_structure_example/`, the `spec` folder can be found. Replace the existing `spec` folder in `vehicle_signal_specification/` with the one from `dias_kuksa/utils/in-vehicle/vss_structure_example/`. Designing the `spec` folder's file structure can be easily self-explained. The following figure illustrates what the GENIVI data structure would look like when created with the `spec` folder.

.. figure:: /_images/invehicle/dias_GENIVI_structure_.png
    :width: 700
    :align: center

* By modifying the structure of `spec` folder, a user-specific GENIVI data structure can be created that can be fed onto `kuksa-val-server`.

1-5. Before commanding `make`, install python dependencies (anytree, deprecation, stringcase) first::

    $ sudo apt install python3-pip
    $ pip3 install anytree deprecation stringcase pyyaml

1-6. Navigate to the directory, `vehicle_signal_specification/`, and command `make` to create a new JSON file::

    $ make

1-7. As a result, you can get a JSON file named as `vss_rel_2.0.0-alpha+006.json`. Let's rename this file as `modified.json` for convenience and move it to `kuksa.val/build/src/`, where the `kuksa-val-server` executable file is located.

2. Now we can bring up and run the `kuksa.val` server with `modified.json`. Navigate to the directory, `kuksa.val/build/src/`, and command the following::

    $ ./kuksa-val-server --vss modified.json --insecure --log-level ALL

* The `kuksa.val` server is entirely passive. Which means that you would need supplementary applications to feed and fetch the data. `dbcfeeder.py` and `cloudfeeder.py` are introduced in the following contents. They are meant to deal with setting and getting the data from the `kuksa.val` server.



.. _dbc-feeder:

kuksa.val - dbcfeeder.py Setup
******************************

.. figure:: /_images/invehicle/invehicle_schema_dbcfeeder.png
    :width: 1200
    :align: center

`kuksa.val/examples/dbc2val/dbcfeeder.py` is to interpret and write the CAN data that is being received by the CAN interface (e.g., `can0` or `vcan0`) to the `kuksa.val` server.

* `dbcfeeder.py` takes four compulsory arguments to be run:
	* CAN interface (e.g., `can0` or `vcan0`) / `-d` or `--device` / To connect to the CAN device interface.
	* JSON token (e.g., `super-admin.json.token`) / `-j` or `--jwt` / To have write-access to the server.
	* DBC file (e.g., `dbcfile.dbc`) / `--dbc` / To translate the raw CAN data.
	* Mapping YML file (e.g., `mapping.yml`) / `--mapping` / To map each of the specific signals to the corresponding path in the `kuksa.val` server.

* Since the `kuksa.val` work package has the admin JSON token already, you only need a DBC file and a YML file. The `junh-ki/dias_kuksa` repository provides the example DBC file and YML file. (DBC file is target-vehicle-specific and can be offered by the target vehicle's manufacturer.)

1. Assuming you have already cloned the `junh-ki/dias_kuksa` repository, / If you haven't, please clone it now::

	$ git clone https://github.com/junh-ki/dias_kuksa.git

2. Navigate to the directory, `dias_kuksa/utils/in-vehicle/dbcfeeder_example_arguments/`, and copy `dias_mapping.yml` and `dias_simple.dbc` *(omitted due to the copyright issue and thus shared on request)* to `kuksa.val/clients/feeder/dbc2val/` where `dbcfeeder.py` is located.

3. Before running `dbcfeeder.py`, install python dependencies (python-can cantools serial) first::

	$ pip3 install python-can cantools serial websockets

4. If you haven't brought up a virtual CAN interface, `vcan0`, please do it now by following :ref:`virtual-can`.

5. Navigate to `kuksa.val/clients/feeder/dbc2val/`, and command the following::

	$ python3 dbcfeeder.py -d vcan0 -j ../../../certificates/jwt/super-admin.json.token --dbc dias_simple.dbc --mapping dias_mapping.yml

6. (Optional) If your DBC file follows J1939 standard, please follow :ref:`feeder-j1939` to run `dbcfeeder.py` with J1939.



.. _cloud-feeder:

kuksa.val - cloudfeeder.py Setup
********************************

.. figure:: /_images/invehicle/invehicle_schema_cloudfeeder.png
    :width: 1200
    :align: center

* `dias_kuksa/utils/in-vehicle/cloudfeeder_telemetry/cloudfeeder.py` fetches the data from the `kuksa.val` in-vehicle server and preprocesses it with a user-specific preprocessor, `dias_kuksa/utils/in-vehicle/cloudfeeder_telemetry/preprocessor_bosch.py`, and transmits the result to Hono (:ref:`cloud-hono`) in a form of JSON Dictionary.

* `cloudfeeder.py` takes six compulsory arguments to be run:
    * JSON token (e.g., `super-admin.json.token`) / `-j` or `--jwt` / To have write-access to the server.
	* Host URL (e.g., "mqtt.bosch-iot-hub.com") / `--host`
	* Protocol Port Number (e.g., "8883") / `-p` or `--port`
	* Credential Authorization Username (Configured when creating) (e.g., "{username}@{tenant-id}") / `-u` or `--username`
	* Credential Authorization Password (Configured when creating) (e,g., "your_pw")/ `-P` or `--password`
	* Server Certificate File (MQTT TLS Encryption) (e.g., "iothub.crt") / `-c` or `--cafile`
	* Data Type (e.g., "telemetry" or "event") / "-t" or "--type"

1. (Optional) `preprocessor_bosch.py` is designed to follow Bosch's diagnostic methodologies. Therefore you can create your own `preprocessor_xxx.py` or modify `preprocessor_example.py` that replaces `preprocessor_bosch.py` to follow your own purpose. Of course, the corresponding lines in `cloudfeeder.py` should be modified as well in this case.

2. Navigate to `dias_kuksa/utils/in-vehicle/cloudfeeder_telemetry/`, copy `cloudfeeder.py` and `preprocessor_example.py` to `kuksa.val/clients/vss-testclient/`, where the `testclient.py` file is located.

3. Then the `do_getValue(self, args)` function from `kuksa.val/clients/vss-testclient/testclient.py` should be modified as below.

.. code-block:: python

	...

	def do_getValue(self, args)::
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

4. Due to its dependency on the cloud instance information, you should create either a Eclipse Hono or a Bosch-IoT-Hub instance first by following :ref:`cloud-hono`, so that you can get the required information for running `cloudfeeder.py` ready.

5. Download the server certificate `here <https://docs.bosch-iot-suite.com/hub/general-concepts/certificates.html>`_ and place it to `kuksa.val/clients/vss-testclient/`, where the `cloudfeeder.py` file is located.

6. Before running `cloudfeeder.py`, install dependencies (`mosquitto`, `mosquitto-clients`, from `apt` and `pygments`, `cmd2` from `pip3`) first::
    
    $ sudo apt-get update
    $ sudo apt-get install mosquitto mosquitto-clients
    $ pip3 install pygments cmd2

7. When all the required information is ready, navigate to `kuksa.val/clients/vss-testclient/`, and run `cloudfeeder.py` by commanding::

	$ python3 cloudfeeder.py -j {admin_json_token} --host {host_url} -p {port_number} -u {auth-id}@{tenant-id} -P {password} -c {server_certificate_file} -t {transmission_type}

* Just a reminder, the information between `{}` should be different depending on the target Hono instance. You can follow :ref:`cloud-hono` to create a Hono instance.

* `admin_json_token` can be found under the directory (`kuksa.val/certificates/jwt/super-admin.json.token`). Therefore, `../../certificates/jwt/super-admin.json.token` should be entered for `-j` when considering the current directory is `kuksa.val/clients/vss-testclient/`.

* If you have successuly made it here, you would be able to see `cloudfeeder.py` fetching and transmitting the data every 1~2 seconds by now. 
