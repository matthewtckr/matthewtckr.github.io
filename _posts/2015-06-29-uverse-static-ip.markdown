---
layout: post
title:  "Configuring AT&T U-Verse NVG589 for Static IPs with Pass-Through to Almond+ OpenWRT Router"
date:   2015-06-23
categories: uverse
excerpt_separator: <!--more-->

---

## Background

For this tutorial, we will be configuring the network so that all client systems and IP addresses will be managed via the Almond+ router.  Aside from the router, no other devices will be connected to the AT&T modem's built-in switch or WiFi.

When configuring AT&T U-Verse with Static IPs, you'll be assigned a block of 8 IP addresses, 5 of which are "usable".  This tutorial will walk you through the steps necessary to get an Almond+ Router (firmware R076; OpenWRT-based) working, while also keeping the "Dynamic IP" that will still be associated with the U-Verse modem, allowing 6 total IPs to be used.
<!--more-->
For the sake of this tutorial, we'll assume the following configuration information is provided by AT&T:

* Assigned Subnet: 1.2.3.136/29
* Subnet Mask: 255.255.255.248
* Network Address: 1.2.3.136
* Broadcast Address: 1.2.3.143
* Router Address: 1.2.3.142
* Usable Addresses: 1.2.3.137 - 1.2.3.141

## AT&T NVG589 Configuration

1. Ensure that the modem is working normally before following these steps
1. Connect to the management website by navigating to `http://192.168.1.254`
1. Under the `Home Network` section, click `Wireless`
  * Set "Wireless Operation" to "Off"
1. Under the `Home Network` section, click `Subnets & DHCP`
  * Leave the Private LAN Subnet section as-is
  * Set the "Public Subnet Enable" option to "On"
  * In the "Public IPv4 Address", enter the Router Address (e.g. 1.2.3.143)
  * In the "Public Subnet Mask", enter the Subnet Mask (e.g. 255.255.255.248)
  * In the DHCPv4 Start Address, enter the lowest "Usable Address" (e.g. 1.2.3.137)
  * In the DHCPv4 End Address, enter the highest "Usable Address" (e.g. 1.2.3.141)
  * Set the "Allow Inbound Traffic" option to "On"
  * Set the "Primary DHCP Pool" to "Private" (We'll let the router assign these later)
  * Ensure that "Cascaded Router Enable" is set to "Off"
  * Save the settings
1. Under the `Firewall` section, ensure the following settings:
  * Packet Filter: Off
  * IP Passthrough: On
    * Allocation Mode: Passthrough
    * Passthrough Mode: DHCPS-Dynamic
  * NAT Default Server: Off
  * Firewall Advanced: Off

## Almond+ (OpenWRT) Configuration

1. Ensure that the router is working normally (Dynamic WAN IP), and the modem has been configured, before following these steps
1. Connect to the management website by navigating to `http://10.10.10.254`
1. In the `Advanced` tab, click `OpenWRT`
1. Under the `Network` section, select `Interfaces`, for the "WAN" network, click the "Edit" button
  * In "General Setup" tab, ensure the "Protocol" setting is "DHCP Client"
  * Under the "IP-Aliases" section, enter "STATIC1" and click "Add"
    * After reloading, find the "STATIC1" section.
    * For "IPv4-Address", enter the first "Usable Address" (e.g. 1.2.3.137)
    * For the "IPv4-Netmask", enter the Subnet Mask (e.g. 255.255.255.248)
    * For the "IPv4-Gateway", enter the Router Address (e.g. 1.2.3.142)
    * In the "Advanced Settings" tab, in the "IPv4-Broadcast", enter the "Broadcast Address" (e.g. 1.2.3.143)
    * Click Save.
  * Continue adding additional "IP-Aliases" entries for the remaining Usable Addresses
1. Under the `Network` section, select `LAN Settings`
  * Create a static lease for each system that will use a static IP.
    * For ease-of-administration, set the last octet of the local address to the same as what the static IP will be
    * For example, a machine allocated a Local IP of `10.10.10.137` will map to `1.2.3.137`
1. Under the `Network` section, select `Firewall`, and then `General Settings`
  * For the "wan" zone, click Edit
  * Switch to the "Advanced Settings" tab
  * Enter the local network and mask (e.g. `10.10.10.0/24`) in the "Restrict Masquerading to given source subnets" option
1. Under the `Network` section, select `Firewall`, and then `Port Forwards`
  * Create a new Firewall Rule named "STATIC1", and click "Add"
  * Locate the "STATIC1" rule in the table, and click "Edit"
  * For the "Source zone", select "wan"
  * For the "External IP address", enter the public "STATIC1" address (e.g. `1.2.3.137`)
  * For the "Internal IP address", select "Custom", and enter the corresponding local address (e.g. `10.10.10.137`)
  * Click Save
  * Continue adding additional Port Forward rules for each Static IP
1. Under the `Network` section, select `Firewall`, and then `Traffic Rules`
  * Create a new "Source NAT" rule named "STATIC1", enter the "Usable Address" in the "To source IP" textbox, and click Add
  * Locate the "STATIC1" Source NAT rule, and click Edit
  * In the "Source zone", select "lan"
  * In the "Source IP Address" textbox, enter the Local IP (e.g. `10.10.10.137`)
  * In the "Destination zone", select "wan"
  * In the "SNAT IP address" , ensure that the "Usable Address (e.g. `1.2.3.137`) is selected
1. Reboot the router
1. Checking the results
  * In the `Network` section, on the `Interfaces` page, the "WAN" interface should now list a dynamic IP, and all static IPs
  * For devices not associated to an IP, use a "What is my IP" Google search, and check that the IP shown is the dynamic IP
  * For devices associated to a static IP, use a "What is my IP" Google search, and check that the IP shown is the desired static IP

Enjoy!