*******************
Step 3: Cloud Setup
*******************

.. figure:: /_images/cloud/cloud_schema.png
    :width: 1200
    :align: center



.. _cloud-hono:

kuksa.cloud - Eclipse Hono (Cloud Entry)
########################################

.. figure:: /_images/cloud/cloud_hono.png
    :width: 1200
    :align: center



Hono Option 1 - Bosch IoT Hub as Hono
*************************************

.. figure:: /_images/cloud/bosch-iot-hub.PNG
    :width: 300
    :align: center

* The Bosch IoT Hub comprises open source components developed in the Eclipse IoT ecosystem and other communities, and uses :blue:`Eclipse Hono` as its foundation.

* Utilizing Hono is essential to take care of a large amount of connected vehicles due to its scalability, security and reliability.

* The Bosch IoT Hub is available as a free plan for evaluation purposes. The following steps describe how to create a free Bosch IoT Hub instance.

1. If you don't have a Bosch ID, register one `here <https://identity-myprofile.bosch.com/ui/web/registration>`_ and activate your ID through the registered E-Mail.

2. Go to the `main page <https://www.bosch-iot-suite.com/>`_ and click "Sign-in" and finish signing-up for a Bosch IoT Suite account. Then you would be directed to the "Service Subscriptions" page.

3. In the "Service Subscriptions" page, you can add a new subscription by clicking "+ New Subscription". Then it direct you to `Product Selection Page <https://accounts.bosch-iot-suite.com/subscriptions/product-selection>`_ that shows you what services can be offered. Click "Bosch IoT Hub".

4. Then select "Free Plan" and name your Bosch IoT Hub instance. The name should be unique (e.g., kuksa-tut-jun) and click "Subscribe".

5. After you will see your subscription details. Click "Subscribe" again to finish the subscription process.

6. Now you would be in `Service Subscriptions Page <https://accounts.bosch-iot-suite.com/subscriptions>`_. It would take a minute or two for your instance to change its status from "Provisioning" to "Active". Make sure the status is "Active" by refreshing the page.

7. When the status is "Active", click "Show Credentials" of the target instance. Then it would show the instance's credentials information. This information is important to go to the device registry and register your device in the further steps. (You don't need to save this information since you can always come back to see.) Let's copy and save the values of "username" and "password" keys under "device_registry" somewhere. 

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

10. With the information in 9-5, we can run `cloudfeeder.py`. Navigate to `kuksa.val/vss-testclient/` and command::

    $ python3 cloudfeeder.py --host mqtt.bosch-iot-hub.com -p 8883 -u pc01@td23aec9b9335415594a30c7113f3a266 -P kuksatutisfun01 -c iothub.crt -t telemetry



Hono Option 2 - Hono from The KUKSA Cluster
*******************************************

.. figure:: /_images/cloud/eclipse-hono.png
    :width: 270
    :align: center



kuksa.cloud - InfluxDB (Time Series Database)
#############################################

.. figure:: /_images/cloud/cloud_influxdb.png
    :width: 1200
    :align: center





dias_kuksa - Hono-InfluxDB Connector
####################################

.. figure:: /_images/cloud/cloud_hono-influxdb-connector.png
    :width: 1200
    :align: center





kuksa.cloud - Grafana (Visualization Web App)
#############################################

.. figure:: /_images/cloud/cloud_grafana.png
    :width: 1200
    :align: center




