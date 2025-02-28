Migrate Ethernet Drivers to ESP-IDF 5.0
=======================================

esp_eth_ioctl() API
-------------------
:cpp:func:`esp_eth_ioctl` third argument could take `int` (`bool`) number as an input in some cases. However, it was not properly documented and, in addition, the number had to be "unnaturally" type casted to `void *` datatype to prevent compiler warnings as shown in below example:

.. highlight:: c

::

    esp_eth_ioctl(eth_handle, ETH_CMD_S_FLOW_CTRL, (void *)true);


This could lead to misuse of the :cpp:func:`esp_eth_ioctl`. Therefore, ESP-IDF 5.0 unified usage of :cpp:func:`esp_eth_ioctl`. Its third argument now always acts as pointer to a memory location of specific type from/to where the configuration option is read/stored.

Usage example to set Ethernet configuration:

.. highlight:: c

::

    eth_duplex_t new_duplex_mode = ETH_DUPLEX_HALF;
    esp_eth_ioctl(eth_handle, ETH_CMD_S_DUPLEX_MODE, &new_duplex_mode);

Usage example to get Ethernet configuration:

.. highlight:: c

::

    eth_duplex_t duplex_mode;
    esp_eth_ioctl(eth_handle, ETH_CMD_G_DUPLEX_MODE, &duplex_mode);


KSZ8041/81 and LAN8720 Driver Update
------------------------------------
KSZ8041/81 and LAN8720 Drivers were updated to support more devices (generations) from associated product family. The drivers are able to recognize particular chip number and its potential support by the driver.

As a result, the specific "chip number" functions calls were replaced by generic ones as follows:

* `esp_eth_phy_new_ksz8041` and `esp_eth_phy_new_ksz8081` were removed, use :cpp:func:`esp_eth_phy_new_ksz80xx` instead
* `esp_eth_phy_new_lan8720` was removed, use :cpp:func:`esp_eth_phy_new_lan87xx` instead


ESP NETIF Glue Event Handlers
-----------------------------
``esp_eth_set_default_handlers()`` and ``esp_eth_clear_default_handlers()`` functions were removed. Registration of the default IP layer handlers for Ethernet is now handled automatically. If users have already followed the recommendation to fully initialize the Ethernet driver and network interface prior to registering their Ethernet/IP event handlers, then no action is required (except for deleting the affected functions). Otherwise, users should ensure that they register the user event handlers as the last thing prior to starting the Ethernet driver.


PHY Address Auto-detect
-----------------------
Ethernet PHY address auto-detect function ``esp_eth_detect_phy_addr`` was renamed to :cpp:func:`esp_eth_phy_802_3_detect_phy_addr` and its header declaration was moved to :component_file:`esp_eth/include/esp_eth_phy_802_3.h`.


SPI-Ethernet Modules Initialization
-----------------------------------
SPI-Ethernet Modules Initialization was simplified. It is not needed to create an SPI device handle by calling :cpp:func:`spi_bus_add_device` prior SPI-Ethernet MAC instance creation since it is done internally now. The configuration structures :cpp:class:`eth_dm9051_config_t`, :cpp:class:`eth_w5500_config_t` and :cpp:class:`eth_ksz8851snl_config_t` members were updated to include SPI device configuration (to be able to fine tune SPI timig which may be dependend on PCB design, for instance). See :doc:`Ethernet <../api-reference/network/esp_eth>` to find SPI-Ethernet Module initialization example.
