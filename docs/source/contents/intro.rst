************
Introduction
************



DIAS (DIagnostic Anti-tampering Systems)
########################################

Modern vehicles with internal combustion engines are equipped with exhaust aftertreatment systems that drastically reduce the emission of harmful exhaust gases. However, there are companies that offer facilities and services to disable these exhaust aftertreatment systems. In a joint European research and development project, `DIAS <https://dias-project.com/>`_, we will help prevent or uncover these manipulations.



Eclipse KUKSA
#############

.. figure:: /_images/intro/appstacle-kuksa.PNG 
    :width: 800
    :align: center

- KUKSA is a code-based result of the internationally funded project, APPSTACLE (2017 - 2019).

.. figure:: /_images/intro/kuksa_ecosystem.png 
    :width: 1000
    :align: center

- An open-source eco-system for the connected vehicle domain.

- It is introduced to establish a standard for car-to-cloud scenarios.

- It improves comprehensive domain-related development activities.

- It opens the market to external applications and service provider.

- It facilitates the use of open-source software wherever possible without compromising security.

- The initial release (0.1.0): 30.09.2019 / The second release (0.2.0): 01.2020

- Implementing the DIAS use-case with KUKSA benefits both parties by enabling the solution to be compliant with any vehicles regardless of the OEM-specific standards.



DIAS-KUKSA
##########

One objective of DIAS is to create a cloud-based diagnostic system. Therefore it is essential to establish connectivity from the vehicle to cloud components. The KUKSA infrastructure offers the starting point for instituting such connectivity. The goal of this documentation is to make clear how to set up each infrastructure component according to the case of DIAS in a sequential manner so that readers can have a thorough understanding of how to apply their own implementation on the established connectivity with KUKSA.



DIAS-KUKSA Overall Schema
*************************

.. figure:: /_images/intro/overall_schema.png 
    :width: 1200
    :align: center

The figure illustrates the entire connectivity cycle from the vehicle to the end-consumer. In the following chapters, how to establish such connectivity cycle is described in detail.
