ec2 facts
===============================================================================

Dynamic inventory provides metadata from ec2 module and ``hostvars`` contains
additional information like:

.. include:: ec2-instance-metadata-example.txt

Documentation about ec2 facts:
http://docs.ansible.com/ansible/ec2_facts_module.html

In Practice
-------------------------------------------------------------------------------

``boot.yml`` launches ec2 instances and ``ec2.py --list`` returns running ec2
instances with public ip addresses.  When the actual playbook ``site.yml`` runs
for Big Data Stacks, inventory groups and IPs are required to connect.

Traditional Inventory file
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``inventory.txt`` contains, for example::

  [namenodes]
  mycluster0
  mycluster1

  [resourcemanagernodes]
  mycluster0
  mycluster1

  [datanodes]
  mycluster0
  mycluster1
  mycluster2

  [zookeepernodes]
  mycluster0
  mycluster1
  mycluster2

  [hadoopnodes]
  mycluster0
  mycluster1
  mycluster2

  [historyservernodes]
  mycluster2

  [journalnodes]
  mycluster2
  mycluster1
  mycluster0

  [frontendnodes]
  mycluster2

and the following directories for each hostname, e.g. mycluster[0-2] look
like::

  $ cat host_vars/mycluster0
  ansible_ssh_host: 192.168.1.100
  zookeeper_id: 0

Dynamic Inventory (ec2)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Let's assume the official ``ec2.py`` is used for dynamic inventory for Amazon
EC2.  (It's from
https://raw.github.com/ansible/ansible/devel/contrib/inventory/ec2.py)

It returns JSON based on running instances on Amazon, for example, if no vm is running::

  $ ./ec2.py --list
  {
    "_meta": {
      "hostvars": {}
     }
  }

.. note:: chmod a+x ec2.py for executable

If there is running VMs, the return JSON data looks like:

.. include:: ec2-py-sample.txt

EC2 Tags for Inventory Groups
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you noticed that there are tags for host groups like
``tag_hadoopnodes_True``, ec2 instances have key:value tags and these are used
for identifying inventory groups.

Ansible ``ec2`` module provides ``instance_tags`` option to define them.

Example::

  ec2:
    ...
    instance_tags:
       namenodes: True
       ...
       frontendnodes: False

``boot.yml`` defines tags information when it starts EC2 instances.

add_host
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This task converts ``tag_*_True`` list to Inventory groups therefore existing
roles run without changes between dynamic and static file inventories.
    
``convert.yml`` contains add_host tasks to convert the inventory groups.

For example, to convert hadoopnodes::

  - name: convert EC2 tags_*_True inventory groups
    hosts: "tag_hadoopnodes_True"
    tasks:

      - name: Add new instance to host group (hadoopnodes)
        add_host: hostname="{{ hostvars[item]['ec2_ip_address'] }}" groupname=hadoopnodes
        with_items: "{{ groups['tag_hadoopnodes_True'] }}"

The hosts in the ``tag_hadoopnodes_True`` list run the ``add_host`` task to add
its ``ec2_ip_address`` (in this case, AWS VPC, virtual private cloud,
associates with a floating ip address and it is visible via ``ec2_ip_address``)
to the new group name ``hadoopnodes`` which the following ansible roles for
BigData Stack requires.

site.yml with Include
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Three Ansible playbooks are provided to deploy BDS on ec2.

- boot.yml: to start EC2 instances
- convert.yml: to provide inventory groups from dynamic inventory of ec2.
- example-project-nist-fingerprint-matching.yml: to run BDS playbooks
