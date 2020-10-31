*******************
Step 3: Cloud Setup
*******************

.. figure:: /_images/cloud/cloud_schema.png
    :width: 1200
    :align: center


.._manual-deployment:

Deployment Option 1 - Manual
############################

.. _cloud-hono:

kuksa.cloud - Eclipse Hono (Cloud Entry)
****************************************

.. figure:: /_images/cloud/cloud_hono.png
    :width: 1200
    :align: center

.. figure:: /_images/cloud/eclipse-hono.png
    :width: 270
    :align: center

Eclipse Hono provides remote service interfaces for connecting large numbers of IoT devices to a back end and interacting with them in a uniform way regardless of the device communication protocol.

Bosch IoT Hub as Hono
=====================

.. figure:: /_images/cloud/bosch-iot-hub.PNG
    :width: 300
    :align: center

The Bosch IoT Hub comprises open source components developed in the Eclipse IoT ecosystem and other communities, and uses Eclipse Hono as its foundation. Utilizing Hono is essential to deal with a large amount of connected vehicles due to its scalability, security and reliability. The Bosch IoT Hub is available as a free plan for evaluation purposes. The following steps describe how to create a free Bosch IoT Hub instance.

1. If you don't have a Bosch ID, register one `here <https://identity-myprofile.bosch.com/ui/web/registration>`_ and activate your ID through the registered E-Mail.

2. Go to the `main page <https://www.bosch-iot-suite.com/>`_ and click "Sign-in" and finish signing-up for a Bosch IoT Suite account. Then you would be directed to the "Service Subscriptions" page.

3. In the "Service Subscriptions" page, you can add a new subscription by clicking "+ New Subscription". Then it would direct you to `Product Selection Page <https://accounts.bosch-iot-suite.com/subscriptions/product-selection>`_ that shows you what services can be offered. Choose "Bosch IoT Hub".

4. Then select "Free Plan" and name your Bosch IoT Hub instance. The name should be unique (e.g., `kuksa-tut-jun`) and click "Subscribe".

5. After that, you would see your subscription details. Click "Subscribe" again to finish the subscription process.

6. Now you would be in `Service Subscriptions Page <https://accounts.bosch-iot-suite.com/subscriptions>`_. It would take a minute or two for your instance to change its status from "Provisioning" to "Active". Make sure the status is "Active" by refreshing the page.

7. When the status is "Active", click "Show Credentials" of the target instance. Then it would show the instance's credentials information. This information is used to go to the device registry and register your device in the further steps. (You don't need to save this information since you can always come back to see.) Let's copy and save the values of "username" and "password" keys under "device_registry" somewhere. 

8. Now go to `Bosch IoT Hub - Management API <https://apidocs.bosch-iot-suite.com/index.html?urls.primaryName=Bosch%20IoT%20Hub%20-%20Management%20API>`_. The Management API is used to interact with the Bosch IoT Hub for management operations. This is where you can register a device on the Bosch IoT Hub instance you've just created and get the tenant configuration that you would ultimately use as input arguments when running `cloudfeeder.py` (:ref:`cloud-feeder`) for a specific device (e.g., Raspberry-Pi of a connected vehicle).

8-1. Click "Authorize" and paste the "username" and "password" that you copied in 7, then click "Authorize". If successfully authorized, click "Close" to close the authorization window.

8-2. Under the "devices" tab, you can find the "POST" bar. This is to register a new device. Click the tab and then "Try it out" to edit. Copy and paste the tenant-id of the Bosch IoT Hub instance to where it is intended to be placed.

8-3. Under "Request body", there would be a JSON dictionary like the following::

    {
        "device-id": "4711",
        "enabled": true
    }

You can rename the string value of "device-id" according to your taste::

    {
        "device-id": "kuksa-tut-jun:pc01",
        "enabled": true
    }

8-4. Then click "Execute". If the server responses with a code 201, it means the device is successfully registered. If you click "Execute" with the same JSON dictionary again, it would return a code 409. Which means you have tried to register the same device again so it wouldn't register it due to the conflict with the existing one. However, if you change "device-id" to something new and click "Execute", then it would return a code 201 because you have just registered a new device name. 

* Just like this, you can register up to 25 devices with a free plan Bosch IoT Hub instance. This means that 25 vehicles or any other IoT devices can be connected to this one Bosch IoT Hub instance and each and every one of them interacts with the instance through a unique "device-id".

* To list all the registered devices' ids, you can click the "GET /registration/{tenant-id}" bar, type the instance's tenant-id and click "Execute". If successful, the server would return a code 200 with the device data that lists all the devices that are registered to the instance.

9. What we have done so far is, create a Bosch IoT Hub instance and register devices in it. However, we haven't yet configured credentials for each device.
Credential information helps you access to a specific device that is registered in the instance. The following steps illustrate how to add new credentials for a device.

9-1. Under the "credentials" tab, find and click the "POST" bar.

9-2. Click "Try it out" and paste the tenant-id of the Bosch IoT Hub instance to where it is intended to be placed.

9-3. In the JSON dictionary, change the value of "device-id" to the target device-id's value.

9-4. Set values of "auth-id" and "password" according to your preference::

    {
        "device-id": "kuksa-tut-jun:pc01",
        "type": "hashed-password",
        "auth-id": "pc01",
        "enabled": true,
        "secrets": [
            {
                "password": "kuksatutisfun01"
            }
        ]
    }

If the server responses with a code 201, it means that new credentials have been added successfully.

* Here the values of "auth-id" and "password" are used to run `cloudfeeder.py`. Therefore it is recommended to save them somewhere.

9-5. Now we have all information to run `cloudfeeder.py`:

    * Host URL: "mqtt.bosch-iot-hub.com"
    * Protocol Port Number: "8883"
    * Credential Authorization Username (e.g., "{username}@{tenant-id}"): "pc01@td23aec9b9335415594a30c7113f3a266"
    * Credential Authorization Password: "kuksatutisfun01"
    * Server Certificate File: "`iothub.crt <https://docs.bosch-iot-suite.com/hub/general-concepts/certificates.html>`_"
    * Data Type: "telemetry"

10. With the information in 9-5 (should be different in your case), we can run `cloudfeeder.py` (:ref:`cloud-feeder`). Navigate to `kuksa.val/vss-testclient/` and command::

    $ python3 cloudfeeder.py --host mqtt.bosch-iot-hub.com -p 8883 -u pc01@td23aec9b9335415594a30c7113f3a266 -P kuksatutisfun01 -c iothub.crt -t telemetry



kuksa.cloud - InfluxDB (Time Series Database)
*********************************************

.. figure:: /_images/cloud/cloud_influxdb.png
    :width: 1200
    :align: center

Now that we have set up a Hono instance, `cloudfeeder.py` can send the telemetry data to Hono every one to two seconds. Hono may be able to collect all the data from its connected vehicles. However, Hono is not a database, meaning that it doesn't store all the collected data in itself. This also means that we have to hire a time series database manager that can collect and store the data received by Hono in chronological order.

InfluxDB is another kuksa.cloud's component, that is an open-source time series database. In KUKSA, InfluxDB is meant to be used as the back-end that stores the data incoming to Hono. With InfluxDB, we can make use of the collected data not only for visualization but also for a variety of external services such as a mailing service or an external diagnostic service. InfluxDB should be located in the northbound of Hono along with Hono-InfluxDB-Connector that should be placed in-between Hono and InfluxDB. 

* To set up InfluxDB and Hono-InfluxDB-Connector, we can use a Linux machine (:ref:`data-consumer`). Based on Hono, the Linux machine here can be considered as a data consumer while the in-vehicle Raspberry-Pi is considered as a data publisher.

* The following steps to setup InfluxDB is written based on `this tutorial <http://www.andremiller.net/content/grafana-and-influxdb-quickstart-on-ubuntu>`_.

1. VirtualBox with Ubuntu 18.04 LTS is used here for setting up InfluxDB and Hono-InfluxDB-Connector. (VM Setup Tutorial can be found `here <https://codebots.com/library/techies/ubuntu-18-04-virtual-machine-setup>`_.) (If your default OS is already Linux, this step can be skipped.)

2. Run your Virtual Machine (VM) and open a terminal.

3. Before InfluxDB installation, command the following::

    $ sudo apt-get update

    $ sudo apt-get upgrade

    $ sudo apt install curl

    $ curl -sL https://repos.influxdata.com/influxdb.key | sudo apt-key add -

    $ source /etc/lsb-release

    $ echo "deb https://repos.influxdata.com/${DISTRIB_ID,,} ${DISTRIB_CODENAME} stable" | sudo tee /etc/apt/sources.list.d/influxdb.list

4. Then install InfluxDB::

    $ sudo apt-get update && sudo apt-get install influxdb

5. Start InfluxDB::

    $ sudo service influxdb start

* If there is no output produced from this command, you have successfully set up InfluxDB on your VM. Please continue with 6 if you want to know how to interact with InfluxDB through a Command Line Interface (CLI). Otherwise, you can directly move onto Hono-InfluxDB-Connector (:ref:`cloud-hono-influxdb-connector`).

6. Connect to InfluxDB by commanding::

    $ influx

* After this command, you would be inside the InfluxDB shell.

7. Create a database, "kuksademo", by commanding inside the InfluxDB shell::

    > CREATE DATABASE kuksademo

* This command produces no output, but when you list the database, you should see that it was created.

8. List the database by commadning inside the InfluxDB shell::

    > SHOW DATABASES

9. Select the newly created database, "kuksademo", by commanding inside the InfluxDB shell::

    > USE kuksademo

* It should produce the following output on the terminal: "Using database kuksademo" 

10. Insert some test data using the following command::

    > INSERT cpu,host=serverA value=0.64

* More information about inserting data can be found `here <https://docs.influxdata.com/influxdb/v0.12/guides/writing_data/>`_

11. The insert command does not produce any output, but you should see your data when you perform a query::

    > SELECT * from cpu

12. Type “exit” to leave the InfluxDB shell and return to the Linux shell::

    > exit

13. (Optional) If you want to write test data from the Linux shell, you can run the following one line script::

    $ while true; do curl -i -XPOST 'http://localhost:8086/write?db=kuksademo' --data-binary "cpu,host=serverA value=`cat /proc/loadavg | cut -f1 -d ' '`"; sleep 1; done

* This command will write data to the `kuksademo` database every 1 second.

14. You can verify if data is being sent to InfluxDB by using the influx shell and running a query::

    > influx
    > USE kuksademo
    > SELECT * FROM cpu



.. _cloud-hono-influxdb-connector:

dias_kuksa - Hono-InfluxDB-Connector
************************************

.. figure:: /_images/cloud/cloud_hono-influxdb-connector.png
    :width: 1200
    :align: center

Now that Hono and InfluxDB are set up, we need a connector application to transmit the incoming data from Hono to InfluxDB. `cloudfeeder.py` produces and sends Hono the result telemetry messages in a form of JSON dictionary. Therefore the connector application should be able to read the JSON dictionary from Hono, map the dictionary to several individual metrics and send them to InfluxDB by using the `curl` command.

* Since the messaging endpoint of Hono (Bosch IoT Hub) follows the AMQP 1.0 protocol, the connector application should also be AMQP based.

* An AMQP Based connector application can be found in `dias_kuksa/utils/cloud/maven.consumer.hono` from the `junh-ki/dias_kuksa` repository. The application is written based on `iot-hub-examples/example-consumer` from the `bosch-io/iot-hub-example` `respoitory <https://github.com/bosch-io/iot-hub-examples/tree/master/example-consumer>`_.

1. To set up the connector, you have to clone the `junh-ki/dias_kuksa` repository on your machine first::

    $ git clone https://github.com/junh-ki/dias_kuksa.git

2. Navigate to `dias_kuksa/utils/cloud/maven.consumer.hono` and check `README.md`. As stated in `README.md`, there are three prerequisites to be installed before running this application.

2-1. Update the system::

    $ sudo apt update
    $ sudo apt upgrade

2-1. Install Java (OpenJDK 11.0.8)::

    $ sudo apt install openjdk-11-jre-headless openjdk-11-jdk-headless
    $ export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64/
    $ echo $JAVA_HOME

2-2. Install Maven (Apache Maven 3.6.0)::

    $ sudo apt install maven
    $ mvn --version

2-3. Install mosquitto-clients::

    $ sudo apt install mosquitto-clients

2-4. Install curl::

    $ sudo apt install curl

3. Navigate to `dias_kuksa/utils/cloud/maven.consumer.hono/` and command the following::

    $ mvn clean package -DskipTests

* This command compiles the `src` folder with Maven and produces the `target` folder that contains a .jar formatted binary file, `maven.consumer.hono-0.0.1-SNAPSHOT.jar`.

4. Now that you have the binary file, you can execute the connector application. In the same directory, `dias_kuksa/utils/cloud/maven.consumer.hono/`, command the following::

    $ java -jar target/maven.consumer.hono-0.0.1-SNAPSHOT.jar --hono.client.tlsEnabled=true --hono.client.username={messaging-username} --hono.client.password={messaging-password} --tenant.id={tenant-id} --device.id={device-id} --export.ip={export-ip}

* (Bosch IoT Hub) The corresponding info (messaging-username, messaging-password, tenant-id, device-id) can be found in `Service Subscriptions Page <https://accounts.bosch-iot-suite.com/subscriptions>`_.

* The startup can take up to 10 seconds. If you are still running `cloudfeeder.py`, the connector application should print out telemetry messages on the console.

5. (Optional) If you want to change the way the connector application post-processes telemetry messages, you can modify `ExampleConsumer.java` that can be found in the directory: `dias_kuksa/utils/cloud/maven.consumer.hono/src/main/java/maven/consumer/hono/`.

* The method, `handleMessage`, is where you can post-process.

* The `content` variable is where the received JSON dictionary string is stored.

* To seperate the dictionary into several metrics and store them in a map, the `mapJSONDictionary` method is used.

* Each metric is stored in a variable individually according to its type and sent to the InfluxDB server through the `curlWriteInfluxDBMetrics` method.

* You can add the post-processing part before `curlWriteInfluxDBMetrics` if necessary.



kuksa.cloud - Grafana (Visualization Web App)
*********************************************

.. figure:: /_images/cloud/cloud_grafana.png
    :width: 1200
    :align: center

So far we have successfully managed to set up Hono and InfluxDB, and transmit data incoming to Hono to InfluxDB by running Hono-InfluxDB-Connector. Now our concern is how to visualize the data inside InfluxDB. One way to do this is to use Grafana.

Grafana is a multi-platform open source analytics and interactive visualization web application. The idea here is to get Grafana to read InfluxDB and visualize the read data.

* The installation steps to setup Grafana is written based on `here <https://grafana.com/docs/grafana/latest/installation/debian/>`_.

1. To install Grafana (stable version 2.6) on your VM, run following commands::

    $ sudo apt-get install -y apt-transport-https
    $ sudo apt-get install -y software-properties-common wget
    $ wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
    $ echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
    $ sudo apt-get update
    $ sudo apt-get install grafana

2. Start Grafana service::

    $ sudo service grafana-server start

* If this command doesn't work, list PIDs on port 3000 (Grafana uses port 3000) to see whether grafana-server is already running on one of them::

    $ sudo apt install net-tools
    $ sudo netstat -anp tcp | grep 3000

* assuming the PID number is: 13886::

    $ sudo kill 13886
    $ sudo service grafana-server start

3. Check whether the Grafana instance is running::

    $ sudo service grafana-server status

* `ctrl` + `c` to get out.

4. Now that the Grafana server is running on your machine, you can access to the server by using a web-browser. Open a browser and access to the following address::

    http://localhost:3000/

5. Log in with the admin account::

    Email or username: admin
    Password: admin

6. After logging in, click "Configuration" on the left, click "Add data source" and select "InfluxDB". 

7. Then you would be in the InfluxDB Settings page. Go to "HTTP" and set URL as follow::

    URL: http://localhost:8086

8. Then go to "IndluxDB Details". Here we are going to select the "kuksademo" database that we have created to test InfluxDB. You can also choose another database that Hono-InfluxDB-Connector has been sending data to. To choose "kuksademo", enter in the following information::

    Database: kuksademo
    User: admin
    Password: admin
    HTTP Method: GET

9. Click "Save & Test". If you see the message, "Data source is working", it means that Grafana has been successfully connected to InfluxDB.

10. Now you can create a new dashboard. Click "Create" on the left and click "Add new panel".

11. Then you would be in the panel editting page. You can choose what metrics you want to analyze. This depends entirely on what metrics you have been sending IndluxDB. Since the metrics we have created in "kuksademo" is `cpu`, you can set the following information:: 

    FROM: `default` `cpu`

12. Click "Apply" on the upper right. Now a new dashboard has been created, you can change the time scope, refresh or save the dashboard on the top.

* In the same way, you can create multiple dashboards for different metrics.



Deployment Option 2 - Docker Compose
####################################

:ref:`manual-deployment` has been introduced to understand what kinds of cloud components are used for `kuksa.cloud` and how to configure them so that they can interact each other. However, deploying each and every cloud component and configuring them manually is not plausible when considering a large number of connected vehicles. This is where container technology like Docker comes into play. A couple of key concepts are described below:

* Docker Container: A standard unit of software that packages up code and all its dependencies so the application runs quickly and reliably from one computing environment to another.
* Docker Compose: A tool for defining and running serveral Docker containers. A YAML file is used to configure the application's services.
* Kubernetes: One difference between Docker Compose and Kubernetes is that Docker Compose runs on a single host, whereas Kubernetes is for running and connecting containers on multiple host.

.. figure:: /_images/cloud/docker_example.png
    :width: 450
    :align: center

.. `here <https://identity-myprofile.bosch.com/ui/web/registration>`_
.. Docker and  comes in.
.. inefficient




Deployment Option 3 - Azure Kubernetes Service (AKS)
####################################################





(Additional) dias_kuksa - InfluxDB-Consumer
###########################################

Since there are possibly more applications that use InfluxDB other than Grafana, it makes sense to create a consumer application that fetches data from InfluxDB and makes them available for any purposes.

* There is an InfluxDB consumer Python script, `influxDB_consumer.py`, in `dias_kuksa/utils/cloud/`.

* The script fetches the last data under certain keys from the local InfluxDB server and store them in the corresponding Python dictionary to each key by using the function, `storeNewMetricVal`. Then you can use the data in the Python dictionary according to your purpose and goals.
