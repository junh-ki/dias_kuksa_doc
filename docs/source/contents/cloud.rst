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

7. When the status is "Active", click "Show Credentials" of the target instance. Then it would show the instance's credentials information. This information is important to go to the device registry and register your device in the further steps. (You don't need to save this information since you can always come back to see.) Let's copy the username under "device_registry" (e.g., manager@rd23aec9...). 

8. Now go to `Bosch IoT - Hub Management API <https://apidocs.bosch-iot-suite.com/index.html?urls.primaryName=Bosch%20IoT%20Hub%20-%20Management%20API>`_ and click "Authorize".



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




