.. _openvpn_roadwarrior-section:

====================
OpenVPN Road Warrior
====================


Road Warrior refers to a specific configuration of the OpenVPN VPN tailored for remote users, allowing them secure access to a
private network from anywhere on the internet.
This setup is particularly useful for businesses and organizations with employees or collaborators spread across different locations,
ensuring encrypted communication and data privacy.

OpenVPN is a protocol supported by the most widely used platforms, with :ref:`free clients <client_software-section>` available for Windows, MacOS, Linux, Android, and iOS systems.

.. note::  Before configuring the OpenVPN Road Warrior, make sure you have read the chapter related to the :ref:`user database <users_database-section>`.

Server configuration
--------------------

An OpenVPN server is running on NethSecurity waiting for remote clients to contact it and establish a connection. It must be reachable from the internet on its specific port (default: ``1194/UDP``).
Multiple clients can connect to the server, authenticate themselves and gain access to the private network; however, the clients do not need to be reachable on the internet. Each connecting client, after authentication, receives an IP address with which it will present itself to the remote network.

An OpenVPN server on NethSecurity is closely tied to a user database, which can be local or remote.
The association with the database is defined during server creation and cannot be modified later.

The server configuration is straightforward because NethSecurity automatically sets most fields to sane defaults, which usually only need verification.

To configure a new OpenVPN server, click :guilabel:`Create Server` button and configure the proposed fields:

* ``Server name``: give a name to this OpenVPN server

* ``User database``: choose the user database to use for authentication, it can be a local database or a remote one (e.g. LDAP or Active Directory)

* ``Create an account for each user``: this is a special field and won't be shown again in the future, it allows you to automatically create a VPN account for each user present in the database. All accounts created will have a certificate valid for 3650 days.

* ``Mode``: bridged or routed; routed mode is the default one and the most common, it allows to create a virtual network where clients 
  are connected to the server and can communicate with each other.
  Bridged mode is less common and allows to connect the clients to the server as if they were connected to the same LAN
  this mode is useful when the clients need to access resources that are not directly accessible from the server.
  If unsure, select routed mode.

* ``Authentication mode``: several authentication modes are supported:

  * ``Username and password``: the connecting client must provide a valid username and password; only users with a password set can use this mode

  * ``Certificate``: the connecting client must have its own certificate to authenticate; this is the recommended mode for most cases

  * ``Username password and certificate``: the connecting client must provide a valid username, password and certificate 

  * ``Username, certificate and OTP``: the connecting client must provide a valid username, certificate and also an OTP code used as a password. This mode requires additional configuration in the client to receive to OTP code

* ``VPN Network``: the virtual network used by clients; every client will receive an IP address taken from this network. NethSecurity already suggests an uncommon network to avoid overlaps with other networks used by the firewall

* ``Dynamic range IP start``: the first IP address that will be assigned to clients connecting to the server; the address must be part of the VPN network.
  When adding an IP reservation to client, make sure that the IP address is outside the dynamic range.

* ``Dynamic range IP end``: the last IP address that will be assigned to clients connecting to the server

* ``Public IP/hostname of this unit``: NethSecurity automatically fills out this field with the public IP address of each configured WAN interface.
  These IPs/hostnames will go into the client configuration.
  The order of the elements is crucial because the connecting client will start contacting the IPs/hostnames beginning with the first in the list and then progressing down the list in case of unavailability.

Click on the :guilabel:`Create` button to create the server. After that, main server details will be shown in the Web UI.

Advanced settings
^^^^^^^^^^^^^^^^^

If needed, you can also customize some advanced options:

* ``Protocol``: UDP (default), TCP 

* ``Port``: 1194 (default)

* ``Route all client traffic through VPN``: if enabled, all traffic from the client will be routed into the VPN tunnel, even standard internet traffic. It can be used for monitoring and control purposes, but is typically disabled because it introduces increased latency and consumes bandwidth.

* ``Push network routes``: a list of networks that the client should route in the VPN tunnel; the LAN networks are automatically added, but they can also be removed and other networks can be added in the same way

* ``Allow client-to-client network traffic``: allows all connected clients to exchange traffic between them; it is recommended to leave it disabled.

* ``Compression``: compress the OpenVPN tunnel traffic to save bandwidth. However, it is now a less useful option and, in some cases, can be detrimental. It is highly recommended to leave it disabled. When this option is modified, it is necessary to download the client configuration again.

* ``Digest``: the digest authenticates data channel packets (default SHA 256)

* ``Cipher``: encryption cipher used (default AES-256-GCM) 

* ``Enforce a minimum TLS version``: allows connection only for clients using a TLS version equal to or greater than the one specified.

* ``Custom DHCP options``: pass specific DHCP options to the client (e.g. DOMAIN, DNS, WINS and so on)


DHCP options
~~~~~~~~~~~~

The DHCP options are used to pass specific configuration parameters to the client.
Available DHCP options are:

1. ``DNS [addr]``: set primary and secondary DNS server addresses (IPv4 or IPv6). Repeat the option to set multiple addresses.

2. ``WINS [addr]``: set primary and secondary Windows Internet Name Service Server (NetBIOS over TCP/IP Name Server) addresses.
   Repeat the option to set multiple addresses.

3. ``NBDD [addr]``: set primary and secondary NetBIOS Datagram Distribution Server (NetBIOS over TCP/IP Datagram Distribution Server) addresses.
   Repeat the option to set multiple addresses.

4. ``NTP [addr]``: set primary and secondary Network Time Protocol server addresses. Repeat the option to set multiple addresses.

5. ``NBT [type]``: set NetBIOS over TCP/IP ode type:

   - ``1``: Broadcast
   - ``2``: Point-to-point (uses WINS)
   - ``4``: Mixed (broadcast, then query name server)
   - ``8``: Hybrid (query name server, then broadcast)

6. ``NBS [scope-id]``: set NetBIOS Scope ID to isolate NetBIOS traffic and allow unique computer names across different scopes.

7. ``DISABLE-NBT [1]``: Turn off NetBIOS over TCP/IP. The parameter is simply ``1`` to enable the option.

VPN accounts
------------

Now that the server has been configured, it is necessary to create the accounts for the connecting clients. To do this, click on :guilabel:`Add VPN Account` and fill out the form:

* ``User``: each account is associated with only one user from the chosen database, select the user for this account

* ``Reserved IP``: specify an IP address that is part of the defined VPN network but outside the dynamic range.
  The entered IP address will always be assigned to this specific account, this can be very useful for creating firewall rules.
  Leave it blank to assign a random IP address on every connection.

* ``Certificate expiration (days)``: specify a certificate duration (default 3650 days)

Once the account is created, it is necessary to export the configuration and load it into the client that needs to connect. To do this, simply click on the menu of the specific account and choose ``Download configuration``.
This action downloads the ready-to-use file, simply to be loaded into the client. This file is dynamically generated based on the current configuration of the OpenVPN server and already contains all the necessary information, including configuration details (server addresses, port, etc.) and required certificates. In case the server's operating mode is changed (e.g., if the authentication mode is altered), it is necessary to download the file again.

Other available actions are:

* ``Disable``: disable the account; the account can be re-enabled at any time.

.. note:: If a client is already connected to the roadwarrior server, the ``Disable`` action on the respective account causes an immediate disconnection from the server, interrupting the communication.

* ``Regenerate certificate``: recreate the personal certificate for the account; if the current certificate has not expired, it will be revoked, and it will be necessary to use the new one. After recreating the certificate, it is necessary to update it on the client by either re-downloading the entire configuration or just the certificate.

* ``Delete``: delete the account and its certificate, this operation is irreversible and the certificate is not recoverable.

Client behavior
^^^^^^^^^^^^^^^

Some information about the behavior of the clients:

* Clients connected to the Road Warrior VPN are assigned to the ``rwopenvpn`` zone, which is inherently trusted.
  By default, this zone has privileged access to both LAN and WAN zones within the network infrastructure.

* Connection backup: in case of multiple WANs, clients will connect using the first IP/hostname of the server configuration, if it's unavailable they will use the second IP/hostname and so on.

* For security reasons, it is not possible to connect multiple clients with the same account. Each account can be used by only one client at a time. 
  If a new client attempts to connect with an account that is already connected to the system, the first account will be disconnected.


.. _client_software-section:

Client software
^^^^^^^^^^^^^^^

All major platforms are supported. Here are some references to download the necessary software:

* Windows Systems: `OpenVPN WebSite <https://openvpn.net/community-downloads/>`_ 

* MacOS Systems: `TunnelBlick <https://tunnelblick.net/>`_ or the `Official Client <https://openvpn.net/client-connect-vpn-for-mac-os/>`_

* Linux Systems: usually already available in most distribution software section, sources are available at `OpenVPN WebSite <https://openvpn.net/community-downloads/>`_ 

* Android Systems: `OpenVPN Connect on Play Store <https://play.google.com/store/apps/details?id=net.openvpn.openvpn>`_

* iOS Systems: `OpenVPN Connect on App Store <https://apps.apple.com/it/app/openvpn-connect-openvpn-app/id590379981>`_

MTU Issue and Packet Fragmentation
----------------------------------

VPN users may experience connectivity issues due to packet fragmentation. The LAN interface has an MTU of 1500, but when packets are encrypted for VPN transmission, the size increases, leading to packet drops. To resolve this, lower the MTU on the VPN server's TUN interface. No changes are required on the client side.

Add these options to the Roadwarrior server configuration ::

  uci set openvpn.ns_roadwarrior1.tun_mtu='1300'
  uci set openvpn.ns_roadwarrior1.mssfix='1250'
  uci commit openvpn.ns_roadwarrior1
  /etc/init.d/openvpn restart ns_roadwarrior1

The MTU values may need to be adjusted to fit your specific network environment. A lower MTU ensures packets fit within the limits of the VPN tunnel without fragmentation, but depending on network latency or overhead, slightly different values might be necessary.

Connection history
------------------

Every time a client connects or disconnects from the server, the event is saved inside a SQLite database stored in RAM.
Such event history can be viewed by clicking on tab ``Connection History`` available on the top of the page.

By default the page will display all connections from current day, but it is possible to filter the results by date and time and account name.

To download all history in CSV format, click on the button :guilabel:`Download server history`.
The header of the CSV file explains the meaning of each column, including the units of measure.

Once the server is rebooted, the local history is lost.
If the server is connected to a :ref:`controller-section`, the history is sent to the controller and can be viewed inside the :ref:`historical_monitoring-section`.
