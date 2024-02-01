# Ansible
Building an inventory
Inventories organize managed nodes in centralized files that provide Ansible with system information and network locations. Using an inventory file, Ansible can manage a large number of hosts with a single command.

To complete the following steps, you will need the IP address or fully qualified domain name (FQDN) of at least one host system. For demonstration purposes, the host could be running locally in a container or a virtual machine. You must also ensure that your public SSH key is added to the authorized_keys file on each host.

Continue getting started with Ansible and build an inventory as follows:

Create a file named inventory.ini in the ansible_quickstart directory that you created in the preceding step.

Add a new [myhosts] group to the inventory.ini file and specify the IP address or fully qualified domain name (FQDN) of each host system.

[myhosts]
192.0.2.50
192.0.2.51
192.0.2.52
Verify your inventory.

ansible-inventory -i inventory.ini --list
Ping the myhosts group in your inventory. 
-----------------------------------------------------
Pinging hosts in a configuration management context, particularly with tools like Ansible, is often used as a basic connectivity test to verify that the hosts are reachable and responsive. It's a simple way to check the network connectivity and ensure that the hosts can be communicated with before more complex tasks are executed. The ping operation typically involves sending a small packet of data to the target host and waiting for a response.


ansible myhosts -m ping -i inventory.ini
Note

Pass the -u option with the ansible command if the username is different on the control node and the managed node(s).


--------------------------------------------------------------------------------------------------------------------------------------

Tips for building inventories
Ensure that group names are meaningful and unique. Group names are also case sensitive.

Avoid spaces, hyphens, and preceding numbers (use floor_19, not 19th_floor) in group names.

Group hosts in your inventory logically according to their What, Where, and When.

What
Group hosts according to the topology, for example: db, web, leaf, spine.

Where
Group hosts by geographic location, for example: datacenter, region, floor, building.

When
Group hosts by stage, for example: development, test, staging, production.

