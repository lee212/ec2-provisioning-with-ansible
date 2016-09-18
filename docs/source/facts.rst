Facts for Create and Terminate Instances
===============================================================================

Ansible provides additional EC2 information of ``ec2`` module using ``ec2_facts``
which contains instance ids, image ids, regions, etc. Boot up plays creates ec2
instances for projects and applications are installed and datasets are loaded
to complete jobs on Cloud resources. If all jobs are finished successfully, the
last step would be terminating VM instances.  ``boot.yml`` playbook stores ec2
instance ids using ``ec2_facts`` in YAML format and ``terminate.yml`` playbook
uses the information to delete instances.

Sample ec2 facts are:
.. include:: ec2_facts_for_vars.txt

to_nice_yaml
-------------------------------------------------------------------------------

``boot.yml`` playbook stores instance information in a YAML file therefore
``terminate.yml`` can find instance ids to terminate. ``to_nice_yaml`` and
``copy`` module (with content option) is used to store ec2_fact information in
a YAML file.

include_vars
-------------------------------------------------------------------------------

The nice thing about Ansible is that data variables in a YAML file can be
easily imported using ``include_vars``. ``terminate.yml`` playbook uses it to
read the instance information that started by ``boot.yml``



