Get Started
===============================================================================

In this document, we expect to start ec2 instances, deploy software stacks with
a dynamic inventory like a text inventory file and terminate instances when
its completed. This can be applied to other cloud platforms such as Microsoft
Azure or OpenStack.

Background
-------------------------------------------------------------------------------

Typical ansible playbook runs with a static text inventory file which consists
of a set of IP addresses or host names accessible via SSH because it assume
that there are machines ready to install software and configure settings
through Ansible roles, plays. When it comes to cloud resources, those IPs or
host names are not permanent because new instances are started with different
network access information.  Ansible Dynamic Inventory allows to retrieve
instance information directly from EC2 instead of reading a text file.
 
In addition, Ansible ``ec2`` cloud module supports creating/terminating ec2
instances as Ansible tasks, therefore booting and terminating ec2 instances are
automated by Ansible plays, in this case, ``boot.yml`` and ``terminate.yml``
files.

AWS Credentials
-------------------------------------------------------------------------------

``AWS_ACCESS_KEY`` and ``AWS_SECRET_ACCESS_KEY`` are required to communicate
with AWS under your account.  Ansible provides several options to import
credentials like ``ec2.ini`` file, boto profile (e.g. ~/.aws/credentials) and
environment variables but we will use environment variables with
``ansible-vault`` to secure the information.

- Be ready with your ``AWS_ACCESS_KEY`` and ``AWS_SECRET_ACCESS_KEY``
- Store the information in a file like::

          $ nano cred
            export AWS_ACCESS_KEY=1234567890
            export AWS_SECRET_ACCESS_KEY=1234567890

(Replace ``1234567890`` with a real value)

- Encrypt the file with ``ansible-vault`` with your password for this file::

          $ ansible-vault encrypt cred
          New Vault password:
          Confirm New Vault password:

The encrypted file looks like::

 $ cat cred
 $ANSIBLE_VAULT;1.1;AES256
 36383835346339373263616238633662613265653766646236616538326130616666373864323537
 6265613637646530396663393236636534313138623536300a343638643134656330646661653230
 32363234646631323034336338376337323836616532396639656234396235303531306133323663
 3838383962386338320a623932313934356563303964303831323732323363343364646235633261
 61396361366131363262623739376535393062663361303966333533396664663832616234306234
 34623461316530643063313338666635396463336361643962633536666264613665343135303163
 613937373831323931613039653438363431

- Enable your EC2 credentials in a shell by::

          $ $(ansible-vault view cred)
          Vault password:
          $

You can confirm whether it's loaded or not by::

  $ env|grep AWS
  AWS_SECRET_ACCESS_KEY=1234567890
  AWS_ACCESS_KEY=1234567890

Dynamic Inventory
-------------------------------------------------------------------------------

It is not required to have a text inventory file in your Ansible home directory
as long as ``ec2.py`` script provides dynamic inventory. 

- Download and run ``ec2.py`` AWS External Inventory Script::

          $ curl -L https://raw.github.com/ansible/ansible/devel/contrib/inventory/ec2.py > ec2.py
          % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
          Dload  Upload   Total   Spent    Left  Speed
          0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
          100 63776  100 63776    0     0   135k      0 --:--:-- --:--:-- --:--:--  317k

- Make it executable by::

  $ chmod a+x ./ec2.py

- Check running EC2 instances, (wait a few seconds)::

          $ ./ec2.py
           {
             "_meta": {
               "hostvars": {}
             }
           }

There is no running instances now but if you have some, the JSON return dataset
looks like:

  .. literalinclude:: ec2-py-sample.txt


- Switch inventory argument option in your ``ansible-playbook`` command like::

  $ ansible-playbook -i ec2.py

For more information, see Ansible Dynamic Inventory `here
<http://docs.ansible.com/ansible/intro_dynamic_inventory.html>`_

Launching EC2 Instances via Ansible ``ec2` module
-------------------------------------------------------------------------------

Ansible provide cloud modules to manage instances and we use ``ec2`` module for
Amazon EC2 cloud to launch or terminate instances. Other clouds i.e. Azure,
Digital Ocean, Docker, Google Compute Engine (GCE) or OpenStack are also
supported in the cloud modules. For more information see documentation `here
<http://docs.ansible.com/ansible/list_of_cloud_modules.html>`_ and source code
from ansible-modules-core `here
<https://github.com/ansible/ansible-modules-core/tree/devel/cloud>`_

The following ``ec2`` task starts a single EC2 instance with Ubuntu 14.04 image::

  - ec2:
      key_name: albert          # SSH registered key name
      instance_type: t2.micro   # EC2 Instance size
      image: ami-2d39803a       # Ubuntu 14.04 image
      group: default            # security group
      region: us-east-1         # EC2 Region

This five lines in ``ec2`` task launches a new EC2 instance on us-east-1 region.
Find out more details `here <http://docs.ansible.com/ansible/ec2_module.html>`_

EC2 Tags for Inventory Groups
-------------------------------------------------------------------------------

``instance_tags`` is used to create inventory groups therefore Ansible plays
run on particular hosts.  For example, deploying hadoop cluster requires
several inventory groups such as namenodes, datanodes or zookeepernodes.
``instance_tags`` is useful to put a label on ec2 instances to distinguish
virtual servers between different inventory groups. The following example shows
that three ec2 instances will be created with the tags for namenodes, datanodes
and zookeepernodes like::

  - ec2:
      key_name: "{{ keypair }}"
      instance_type: "{{ instance_size }}"
      image: "{{ image }}"
      region: "{{ region }}"
      exact_count: 3
      count_tag: three-nodes
      group:  [ 'default', "{{ security_group }}" ]
      instance_tags:
        namenodes: True
        datanodes: True
        zookeepernodes: True
        historyservernodes: False
        frontendnodes: False

The ``key:value`` tags are used here and the AWS EC2 returns JSON data like:

.. literalinclude:: ec2-instance-metadata-example.txt

As you noticed that tags create separate host groups in a ``tag_<key>_<value>``
form, for example, namenodes are listed like::

 u'tag_namenodes_True': [u'52.23.213.103',
                         u'54.196.41.145',
                         u'54.209.137.235'],

It is good because Ansible directly call this group like::

  $ ansible -m ping -i ec2.py -u ubuntu "tag_namenodes_True"

However, if hostnames should be just ``namenodes`` without any prefix or postfix,
we can fix that using ``add_host`` module. Let's see that in ``convert.yml``.

Converting Inventory Groups
-------------------------------------------------------------------------------

``convent.yml`` uses ``add_host`` to create a new inventory groups and it
corrects group names from ``tag_<key>_<value>`` fomat to ``<key>`` like just
``namenodes``. Sample task looks like ::

    - name: convert EC2 tags_*_True inventory groups
      hosts: "tag_namenodes_True"
      tasks:
          - name: Add new instance to host group (namenodes)
            add_host: hostname="{{ hostvars[item]['ec2_ip_address'] }}" groupname=namenodes
            with_items: "{{ groups['tag_namenodes_True'] }}"

This task runs towards ``tag_namenodes_True`` group and create a new group
named ``namenodes`` with EC2 IP addresses therefore following ansible playbooks
or roles can be used without modification in hostnames.

Terminating EC2 Instances
-------------------------------------------------------------------------------

``state: 'absent'`` terminates ec2 instances with instance ids which
``boot.yml`` stores using ``ec2_facts``. Therefore, ``terminate.yml``
reads the YAML fact file and gets instance ids to terminate. ``include_vars``
imports data from YAML and makes them accessible in Ansible Plays.

Sample ec2_fact file looks like:

.. literalinclude:: ec2_facts_for_vars.txt

