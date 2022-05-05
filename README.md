simple-dhcp
===========

A simple Ansible role to manage a very simple DHCP server with static IP
mappings only (ie, no pool), and includes a sample notification mechanism.

Requirements
------------

This role should run fine on most Linux distributions.

Role Variables
--------------

* ```dhcp_state``` -- either ```present``` or ```absent``` to indicate whether this role should be present or absent on the destination host(s). Defaults to ```present```.
* ```dhcp_interfaces``` -- a list of network interface name and address pairs for this host to serve DHCP requests on. Defaults to empty. See below for an example.
* ```dhcp_leases``` -- a list of DHCP leases (MAC, IP, optional name) for this host to serve. See below for an example.
* ```dhcp_leasetime``` -- the time to lease addresses for. Defaults to ```1h```.
* ```dhcp_config_file``` -- the path to the dnsmasq configuration file on the remote node(s). Defaults to ```/etc/dnsmasq.conf```.
* ```dhcp_config_directory``` -- the path to included dnsmasq configuration files. Defaults to ```/etc/dnsmasq.d```.
* ```dhcp_notification_script``` -- the (local) path to a dnsmasq-compatible notification script to be used with the DHCP server. A default sample has been included. Note that this script is passed through the ansible.builtin.template module, so will have standard jinja2 template substitution performed, allowing for the inclusion of variables/facts.

Here are some examples of variables:

```
dhcp_leasetime: 2h
dhcp_state: present
dhcp_interfaces:
  - name: eth0
    address: 10.0.1.10
  - name: eth1
    address: 10.0.2.10
dhcp_leases:
  - mac: 00:00:de:ad:be:ef
    ip: 10.0.1.12
    name: beefy
  - mac: 00:0b:ad:c0:ff:ee
    ip: 10.0.1.13
    name: coffee
  - mac: 00:00:0d:0g:f0:0d
    ip: 10.0.2.12
    name: dogfood
```

Role Tags
---------

This role can be thought of as two separate logical steps. The first is to ensure that the services and packages are installed, have a default configuration and are running as expected. The second is to update and maintain the list of DHCP leases. The former is generally going to be a one-off (but is idempotent, so can be applied periodically to ensure no configuration change/mismatch), whereas the latter will change as the list of nodes changes.

This role can be applied as a whole each time it is executed (the default), but each task has been tagged to allow the caller to differentiate (in an agnostic way) between the two processes mentioned above. The tag ```role``` has been used to denote the initial application of the role (package installation, service configuration, etc) to get to a default state. The tag ```policy``` has been used to denote the tasks used to update and maintain the list of static DHCP leases.

For example, a playbook using this role could be executed as a whole like this:

```
ansible-playbook ./playbook.yml
```

Or we could do the initial role preparation in one stage:

```
ansible-playbook ./playbook.yml -t role
```

and then later periodically handle updates to the configuration:

```
ansible-playbook ./playbook.yml -t policy
```

Dependencies
------------

This role has no dependencies on external roles or modules.

Example Playbook
----------------

This role can quickly be imported into any playbook in the same way any Ansible
Galaxy role can be:

```
    - hosts: dhcpservers
      roles:
         - mattgeddes-platina.simple-dhcp
```

License
-------

Apache-2.0

Author Information
------------------

Matthew Geddes <mgeddes@platinasystems.com>

