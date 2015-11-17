shorewall_simple
================

_WARNING: This role can be dangerous to use. If you lose network connectivity
to your target host by incorrectly configuring your firewall, you may be
unable to recover without physical access to the machine._

This role installs and configures Shorewall for a simple, single network interface (can be a bond, of course) server.

Requirements
------------

This role requires Ansible 1.4 or higher and platform requirements are listed
in the metadata file.

Role Variables
--------------

The variables that can be passed to this role and a brief description about
them are as follows. These are all based on the configuration variables of the
Shorewall configuration.

    shorewall_enabled: "Yes"
    shorewall_startup: 1
    shorewall_wait_interface: "eth0"
    shorewall_options: ""
    shorewall_startoptions: ""
    shorewall_restartoptions: ""
    shorewall_initlog: "/var/log/shorewall_init.log"
    shorewall_safestop: 0
    shorewall_blacklistnewonly: "Yes"
    
    shorewall_zones:
    - zone: "fw"
      type: "firewall"
    - zone: "net"
      type: "ipv4"
      options="-"
      options_in="strict"
      options_out="-"
    
    shorewall_interfaces: 
    - interface: "eth0"
      zone: "net"
      broadcast: "detect"
      options: "dhcp,tcpflags,nosmurfs,logmartians"
    
    shorewall_policies:
    - source: "$FW"
      destination: "net"
      policy: "ACCEPT"
    - source: "net"
      destination: "$FW"
      policy: "ACCEPT"
    - source: "all"
      destination: "all"
      policy: "DROP"
      log_level: "info"
      burst_limit: "10/second:100"
    
    shorewall_rules:
    - section: "NEW"
      rules:
      - action: "ACCEPT"
        source: "net"
        destination: "$FW"
        protocol: "tcp"
        destination_port: 22
        source_port: "-"
        original_destination: "-"
        rate_limit: "-"
        user_group: "-"
        mark: "-"
        connection_limit: "-"
        time: "-"
        headers: "-"
        switch: "-"

Examples
========

1) Example allowing all traffic in and out

    - hosts: all
      - roles:
        - role: shorewall_simple
          shorewall_enabled: "Yes"    
          shorewall_zones:
          - zone: "fw"
            type: "firewall"
          - zone: "net"
            type: "ipv4"
          shorewall_interfaces: 
          - interface: "eth0"
            zone: "net"
            broadcast: "detect"
            options: "dhcp,tcpflags,nosmurfs,logmartians"
          shorewall_policies:
          - source: "all"
            destination: "all"
            policy: "ACCEPT"

2) Example allowing all outgoing traffic but block incomming traffic and log
it, but allow incomming SSH traffic and accept Ping

    - hosts: all
      - roles:
        - role: shorewall_simple
          shorewall_enabled: "Yes"    
          shorewall_zones:
          - zone: "fw"
            type: "firewall"
          - zone: "net"
            type: "ipv4"
          shorewall_interfaces: 
          - interface: "eth0"
            zone: "net"
            broadcast: "detect"
            options: "dhcp,tcpflags,nosmurfs,logmartians"
          shorewall_policies:
          - source: "$FW"
            destination: "net"
            policy: "ACCEPT"
          - source: "net"
            destination: "$FW"
            policy: "DROP"
            log_level: "info"
          - source: "all"
            destination: "all"
            policy: "DROP"
          shorewall_rules:
          - section: "NEW"
            rules:
            - action: "Ping/ACCEPT"
              source: "net"
              destination: "$FW"
            - action: "ACCEPT"
              source: "net"
              destination: "$FW"
              protocol: "tcp"
              destination_port: 22

Dependencies
------------

All systems:
- iptables

Red Hat based distributions:
- EPEL

License
-------

BSD

Author Information
------------------

Philippe Dellaert



