# ISC_dhcp_server_RaspberryPi
Internet Services Consortium (ISC) dhcp server Version 4.3.1 on Raspbian Jessie, 2016.

1    INTRODUCTION

1.1    The Internet Systems Consortium (ISC) DHCP Server
The ISC dhcp server version 4.3.1 is a freely available, command line, dhcp server that has recently been ported to the Raspberry Pi. Despite being is freely available, it is a defacto standard dhcp server in the ISP (Internet Service Provider) industry, with a rich set of features that support cable modems and other customer premesis equipment. ISC does not ship prebuilt binaries, or support Windows. Therefore to create a package on a target OS such as Linux or Solaris requires the underlying dependancies to be present before the package is created.

1.2    Project Concept
The objective of the project was to install the isc-dhcp-server package from Raspberry Pi and test functionality of key features in a laboratory environment. The project was run in a two phases. Firstly to test basic functionality and afterwards to move to a more advanced configuration that gave failover backup in the case of the primary dhcp server failing. Also the configuration was written to allow the server to issue IP address leases from different address pools based on the hardware OUI (Organization Unique Identifier) given in dhcp discovery packet option 60. Using option 60 vendor-class-indentifier, all Raspberry Pi client boards would be issued addresses from the private IP address pool 10.0.91.0/24 while other machines in the lab namely Dell would receive IP addresses from the private IP address pool 10.0.0.0/24.

2   CONFIGURATION

2.1    Configuration Process
Configuring a Raspberry Pi as a dhcp server is a 4 stage process.
First check for any new updates. This is an important step since the isc-dhcp-server has dependencies that must be at specific revision levels in order to function.
: sudo apt-get update

Once the updates has been performed the isc-dhcp-server application can be downloaded an installed.
: sudo apt-get install isc-dhcp-server

By default all Raspberry Pi’s are configured to receive their eth0 IP address automatically when connected to the internet via a dhcp server. In order to administer addresses to other clients it must have a static ip address on its eth0 interface. As of Raspbian 4.4 the static IP address is configured in /etc/dhcpcd.conf . (n.b. it is no longer configured in /etc/network/interfaces ).
: sudo nano /etc/dhcpcd.conf
Append to the bottom of the file,
Interface eth0
Static ip_address = 10.0.91.2
Static routers = 10.0.91.1
Static domain_name_servers = 10.0.91.1
Save the new config and reboot to take effect.

Third, the dhcp “defaults” file must be configured with the server config file and the run process id. This is found at /etc/default/isc-dhcp-server .
Sudo nano /etc/default/isc-dhcp-server
Remove the hash to uncomment the following lines and add interface name eth0.
DHCPD_CONF=/etc/dhcp/dhcp.conf
DHCPD_PID=/var/run/dhcpd.pid
INTERFACES=”eth0”
Save and restart the server with:
: sudo service isc-dhcp-server restart

Other useful commands include
: sudo service isc-dhcp-server stop
: sudo service isc-dhcp-server start
: sudo service isc-dhcp-server status

Fourth, the dhcp daemon config file at /etc/dhcp/dhcpd.conf , requires the server logical configuration. The initial basic configuration used the simplest dhcpd.conf to test that the isc-dhcp-server functioned. The first process failed due to the log-facility function which used the default local 7 level of logging. Raspberry Pi supports logging levels 0 - 3 , hence the configuration encountered an error and the process quit.

The basic configuration used a single subnet 10.0.91.0/24 which offered addresses in the range 10.0.91.10 through 10.0.91.253. The lower and top end of the range were reserved for the dhcp servers and services such as, default-gateway, tftp, ftp, and DNS. The log-facility parameter was set to local3, and the dhcpd server process ran. A RaspberryPi board running Wheezy Debian and a Dell PC running Ubuntu Linux were offered dhcp leases.

The failover peer config gives failover resilience and server load balancing to the class pool to which it is applied. Secondary dhcp server becomes promary
The class ”RASPI” used a, match if substring, to detected the Raspberry Pi OUI of 002001. The full output of ”Vendor-Vlass option 60 string is 32 bytes: ”PXEClient:Arch:00000:UNDI:002001”. The substring performs a match 26 bytes from the start of the string for 6 bytes on 002001. Presumably the RaspberryPi used a DSP Solutions of Palo Alto, Ethernet chipset, since they are the vendor registered with the OUI of 00:20:01.

class "RASPI"{
match if substring(option vendor-class-identifier, 26, 6) = "002001"
}
The class RASPI assigns addressesin the 10.0.91/24 subnet range.
The class default assigns addresses from the default pool to all clients with an OUI other than 002001, in the address subnet range 10.0.0/24.

