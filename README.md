# check_pfsense
nagios check for pfSense using SNMP

This readme describes how to monitor pfSense over SNMP.  It is assumed that you are using nagios for monitoring.


It is assumed you already have the nagios plugins `check_snmp` and `check_http` and `check_ssh` from the nagios-plugins package that shipped with your distro.

It is assumed you already have the nagios plugin `check_snmp_storage` from http://nagios.manubulon.com

Ensure the following sections exist in /etc/nagios/commands.cfg (or wherever your distro puts commands.cfg)

    # 'check_snmp' command definition
    define command{
        command_name    check_snmp
        command_line    $USER1$/check_snmp -H $HOSTADDRESS$ -C $ARG1$ -o $ARG2$ -w $ARG3$ -c $ARG4$
        }

    # 'check_snmp_storage' command definition
    define command{
        command_name    check_snmp_storage
        command_line    $USER1$/check_snmp_storage -H $HOSTADDRESS$ -C $ARG1$ -m $ARG2$ -w $ARG3$ -c $ARG4$
        }


Add the following sections to the /etc/nagios/services.cfg file (or wherever your distro puts services.cfg)

    # check pfSense CPU
    ## The OID 1.3.6.1.4.1.2021.10.1.5.1 shows 1  minute average for %cpu util
    ## The OID 1.3.6.1.4.1.2021.10.1.5.2 shows 5  minute average for %cpu util
    ## The OID 1.3.6.1.4.1.2021.10.1.5.3 shows 15 minute average for %cpu util
    ## The syntax is: !snmp_community!oid!warn!critical
    define service{
       use                             generic-24x7-service
        hostgroup_name                 all_pfsense_routers
        service_description            cpu load
        notification_period            14x7
        check_command                  check_snmp!public!1.3.6.1.4.1.2021.10.1.5.2!70!90
        }



    # check pfSense swap space
    # This depends on the check_snmp_storage script
    # You can find the name of the swap space by looking in /dev/label/ from an SSH login
    # Or by clicking Diagnostics, Command prompt, ls -l /dev/label
    define service {
        use                             generic-24x7-service
        hostgroup_name                  all_pfsense_routers
        service_description             swap
        check_command                   check_snmp_storage!public!/dev/label/swap0!50!75
        }

    # check pfSense / filesystem space
    # This depends on the check_snmp_storage script
    # pfSense only has the root filesystem and swap space, but no other filesystems
    # You can figure out the name of the local disk clicking Diagnostics, Command prompt, ls -l /dev/ufsid
    define service {
        use                             generic-24x7-service
        hostgroup_name                  all_pfsense_routers
        service_description             disk space
        check_command                   check_snmp_storage!public!/dev/ufsid/587f4636bdb209d5!90!95
        }

    # check pfSense WAN interface up
    # figure out which interface is which with these commands:
    #   snmpwalk -v 1 -c public routername 1.3.6.1.2.1.2.2.1.2    <---- shows the interface names:
    #   snmpwalk -v 1 -c public routername 1.3.6.1.2.1.2.2.1.8    <---- shows the interface operational status
    define service {
        use                             generic-24x7-service
        hostgroup_name                  all_pfsense_routers
        service_description             WAN up
        check_command                   check_snmp!public!1.3.6.1.2.1.2.2.1.8.5!1!1
        }

    # check pfSense DMZ interface up
    # figure out which interface is which with these commands:
    #   snmpwalk -v 1 -c public routername 1.3.6.1.2.1.2.2.1.2    <---- shows the interface names:
    #   snmpwalk -v 1 -c public routername 1.3.6.1.2.1.2.2.1.8    <---- shows the interface operational status
    define service {
        use                             generic-24x7-service
        hostgroup_name                  all_pfsense_routers
        service_description             DMZ up
        check_command                   check_snmp!public!1.3.6.1.2.1.2.2.1.8.6!1!1
        }

    # check pfSense LAN interface up
    # figure out which interface is which with these commands:
    #   snmpwalk -v 1 -c public routername 1.3.6.1.2.1.2.2.1.2    <---- shows the interface names:
    #   snmpwalk -v 1 -c public routername 1.3.6.1.2.1.2.2.1.8    <---- shows the interface operational status
    define service {
        use                             generic-24x7-service
        hostgroup_name                  all_pfsense_routers
        service_description             LAN up
        check_command                   check_snmp!public!1.3.6.1.2.1.2.2.1.8.7!1!1
        }

    # Define a service to check web interface
    define service{
        use                             generic-24x7-service
        hostgroup_name                  all_pfsense_routers
        service_description             http
        check_command                   check_http
        }
        
    # confirm SSH daemon is listening
    define service{
        use                            generic-14x7-service
        hostgroup_name                 all_ssh_servers,all_linux,all_supermicro_ipmi,all_vcenter
        service_description            SSH
        check_command                  check_ssh
        notification_period            14x7
        }



The Nagios web interface will look similar to the following:
<img src=images/pfsense.png>

