# Task

**Debian only**

Configure snmpd on a host.

 * `/etc/snmp/snmpd.conf` - Set up the right community string and information.
 * Install the LibreNMS distribution detection script

# Variables

 * `snmp_community`: The SNMP used by all devices to take part in the monitoring
   network.
   * Default: librenms
 * `monitor_admin_email`: Email address of the administrator of the monitoring
   server.
   * Default: root@localhost
 * `monitor_admin_name`: Name of the monitoring server administrator.
   * Default: Local root user.
 * `monitor_location`: Location of the monitoring server.
   * Default: Rick & Morty's home.
