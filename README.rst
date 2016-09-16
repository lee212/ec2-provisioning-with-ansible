ec2-provisioning-with-ansible
===============================================================================

``boot-ec2-with-vpc.yml`` starts 3 ec2 instances to deploy BigDataStacks with
7 inventory groups. This Ansible plays provide:

- create ec2 instances with groups
- run following ansible roles, plays
- terminate instances when its use is completed (TBD)

Resources
-------------------------------------------------------------------------------

- ec2.py is from https://raw.github.com/ansible/ansible/devel/contrib/inventory/ec2.py
- ec2.ini is from https://raw.githubusercontent.com/ansible/ansible/devel/contrib/inventory/ec2.ini
