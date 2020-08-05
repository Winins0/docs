Networking
==========

86Box supports two connection modes for the :doc:`emulated network cards <../settings/network>`. The specific details on these connection modes and network emulation as a whole are outlined on this page.

SLiRP
-----

SLiRP creates a private network with a virtual router, allowing the emulated machine to reach the host, its network and the Internet; on the other hand, the host and other devices on its network cannot reach the emulated machine, unless :ref:`port forwarding <hardware/network:SLiRP port forwarding>` is configured. This is similar to the **NAT** mode on other virtualizers.

The virtual router provides automatic IP configuration to the emulated machine through DHCP. If that is not an option, use the following static IP settings:

* **IP address:** 10.0.2.15
* **Subnet mask:** 255.255.255.0
* **Default gateway:** 10.0.2.2
* **DNS server:** 10.0.2.3

The host can be reached through IP address 10.0.2.2, while other devices on the host's network can be reached through their normal IP addresses.

.. note:: SLiRP is only capable of routing TCP and UDP traffic. Other protocols such as IPX and NetBEUI can only be used with :ref:`hardware/network:PCap` networking.

PCap
----

PCap connects directly to one of the host's network adapters. The emulated machine must be configured as if it were a real machine on your network. This is similar to the **Bridge** mode on other virtualizers.

This mode requires `Npcap <https://nmap.org/npcap/>`_ (or another WinPcap-compatible driver) to be installed on the host. Only **wired Ethernet network connections** are compatible; Wi-Fi and other connections will not work at all, as they do not allow PCap to listen for packets bound to the emulated machine.

Private PCap network
^^^^^^^^^^^^^^^^^^^^

If you have an incompatible network connection (such as Wi-Fi), or if you wish to connect the emulated machine to the host without also connecting it to your network, a private network can be created with PCap in one of two ways:

* Install and configure the *Microsoft KM-TEST Loopback Adapter* included with Windows.

   * Guides on how to install this adapter are available online.
   * The adapter alone only provides a direct connection to the host, with no DHCP server, therefore requiring manual IP configuration on both the host and the emulated machine.
   * Windows' *Internet Connection Sharing* feature can be used to connect the emulated machine to the host's network and the Internet, with DHCP for automatic IP configuration, similarly to SLiRP but with the added benefit that the host can reach the emulated machine without port forwarding.

* If VMware is installed, use one of the VMnet adapters included with it.

   * *VMnet1* (Host-only) connects to the host only.
   * *VMnet8* (NAT) connects to the host, its network and the Internet.

Advanced features
-----------------

The following advanced features can be accessed by directly editing the virtual machine configuration file, which is ``86box.cfg`` by default.

MAC address
^^^^^^^^^^^

All emulated network cards store their MAC address in the ``mac`` directive of the card's configuration file section. Only the suffix (last three octets) of the MAC address is stored; the prefix (first three octets) will always be the card manufacturer's `OUI <https://en.wikipedia.org/wiki/Organizationally_unique_identifier>`_, such as 00:E0:4C for Realtek.

.. rubric:: Example: MAC address 00:E0:4C:35:F4:C2 for the Realtek RTL8029AS

.. code-block:: none

   [Realtek RTL8029AS]
   mac = 35:f4:c2

SLiRP port forwarding
^^^^^^^^^^^^^^^^^^^^^

Port forwarding allows the host and other devices on its network to access TCP and UDP servers running on the emulated machine. This feature is configured through the ``[SLiRP Port Forwarding]`` section of the configuration file.

Each port forward must be assigned a number, starting at 0 and counting up (skipping a number will result in all subsequent port forwards being ignored), which replaces ``X`` on the following directives:

* ``X_udp``: If this directive is missing or set to 0, forward a TCP port. If set to 1, forward an UDP port.
* ``X_from``: Port number on the host.
* ``X_to``: Port number on the emulated machine. If this directive is missing, use the same port number as the host.

The host can access forwarded ports through 127.0.0.1 or its own IP address, while other devices on the network can access them through the host's IP address.

.. rubric:: Example: forward host TCP port 8080 to guest port 80, and host UDP port 5555 to guest port 5555

.. code-block:: none
   
   [SLiRP Port Forwarding]
   0_from = 8080
   0_to = 80
   1_udp = 1
   1_from = 5555