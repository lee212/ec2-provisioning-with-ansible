Basic Issues
-------------------------------------------------------------------------------

- secgroup is only available for vpc
- vpc or subnetid is necessary to create ec2 instance
- '{{secgroup.name}}' doesn't work even though it is defined from http://docs.ansible.com/ansible/ec2_group_module.html
  Did I something wrong? 
- group_id works!::
  31             group_id: "{{secgroup_first.group_id}}"


 VPC & Subnet
-------------------------------------------------------------------------------

- manually added from portal website for those

Additional Files e.g. ec2.ini or ec2.py for dynamic inventory
-------------------------------------------------------------------------------

``aws_access_key_id`` and ``aws_secret_key`` need to be defined in the ``ec2.ini`` file.
additionally, ``rds`` and ``elasticache`` need to be set False::

         rds = False
         elasticache = False

Performance Measurement
-------------------------------------------------------------------------------

Time lapse can be measured by callback functions defined in ``ansible.cfg`` 

::
  [defaults]
  callback_whitelist = profile_tasks

Sample results::

  $ ansible-playbook play.yml -i ec2.py  -vvv
  Using /home/lee212/git/aws-cloudformation-by-ansible/ansible.cfg as config file

  PLAYBOOK: play.yml *************************************************************
  1 plays in play.yml

  PLAY [create a test instance] **************************************************

  TASK [Create security group] ***************************************************
  task path: /home/lee212/git/aws-cloudformation-by-ansible/play.yml:5
  Wednesday 14 September 2016  00:22:33 -0400 (0:00:00.019)       0:00:00.019 ***
  <127.0.0.1> ESTABLISH LOCAL CONNECTION FOR USER: lee212
  <127.0.0.1> EXEC /bin/sh -c '( umask 77 && mkdir -p "` echo $HOME/.ansible/tmp/ansible-tmp-1473826953.12-47283165924615 `" && echo ansible-tmp-1473826953.12-47283165924615="` echo $HOME/.ansible/tmp/ansible-tmp-1473826953.12-47283165924615 `" ) && sleep 0'
  <127.0.0.1> PUT /tmp/tmpvNc7Ff TO /home/lee212/.ansible/tmp/ansible-tmp-1473826953.12-47283165924615/ec2_group
  <127.0.0.1> EXEC /bin/sh -c 'LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8 LC_MESSAGES=en_US.UTF-8 /home/lee212/venv/ansible/bin/python /home/lee212/.ansible/tmp/ansible-tmp-1473826953.12-47283165924615/ec2_group; rm -rf "/home/lee212/.ansible/tmp/ansible-tmp-1473826953.12-47283165924615/" > /dev/null 2>&1 && sleep 0'
  ok: [localhost] => {"changed": false, "group_id": "sg-692c4d13", "invocation": {"module_args": {"aws_access_key": null, "aws_secret_key": null, "description": "A Security group", "ec2_url": null, "name": "security-group-test", "profile": null, "purge_rules": true, "purge_rules_egress": true, "region": "us-east-1", "rules": [{"cidr_ip": "0.0.0.0/0", "from_port": 22, "proto": "tcp", "to_port": 22}], "rules_egress": [{"cidr_ip": "0.0.0.0/0", "from_port": null, "group_desc": "example of ec2 secgroup", "proto": -1, "to_port": null}], "security_token": null, "state": "present", "validate_certs": true, "vpc_id": "vpc-e6c17c83"}, "module_name": "ec2_group"}}

  TASK [ec2 launch test] *********************************************************
  task path: /home/lee212/git/aws-cloudformation-by-ansible/play.yml:22
  Wednesday 14 September 2016  00:22:33 -0400 (0:00:00.552)       0:00:00.571 ***
  <127.0.0.1> ESTABLISH LOCAL CONNECTION FOR USER: lee212
  <127.0.0.1> EXEC /bin/sh -c '( umask 77 && mkdir -p "` echo $HOME/.ansible/tmp/ansible-tmp-1473826953.7-216570325713589 `" && echo ansible-tmp-1473826953.7-216570325713589="` echo $HOME/.ansible/tmp/ansible-tmp-1473826953.7-216570325713589 `" ) && sleep 0'
  <127.0.0.1> PUT /tmp/tmpnwiErZ TO /home/lee212/.ansible/tmp/ansible-tmp-1473826953.7-216570325713589/ec2
  <127.0.0.1> EXEC /bin/sh -c 'LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8 LC_MESSAGES=en_US.UTF-8 /home/lee212/venv/ansible/bin/python /home/lee212/.ansible/tmp/ansible-tmp-1473826953.7-216570325713589/ec2; rm -rf "/home/lee212/.ansible/tmp/ansible-tmp-1473826953.7-216570325713589/" > /dev/null 2>&1 && sleep 0'
  changed: [localhost] => {"changed": true, "instance_ids": ["i-f7364c0f"], "instances": [{"ami_launch_index": "0", "architecture": "x86_64", "block_device_mapping": {"/dev/sda1": {"delete_on_termination": true, "status": "attached", "volume_id": "vol-f678e477"}}, "dns_name": "", "ebs_optimized": false, "groups": {"sg-692c4d13": "security-group-test"}, "hypervisor": "xen", "id": "i-f7364c0f", "image_id": "ami-2d39803a", "instance_type": "t2.micro", "kernel": null, "key_name": "hrlee", "launch_time": "2016-09-14T04:22:34.000Z", "placement": "us-east-1a", "private_dns_name": "ip-172-30-3-157.ec2.internal", "private_ip": "172.30.3.157", "public_dns_name": "", "public_ip": "54.197.111.173", "ramdisk": null, "region": "us-east-1", "root_device_name": "/dev/sda1", "root_device_type": "ebs", "state": "running", "state_code": 16, "tags": {}, "tenancy": "default", "virtualization_type": "hvm"}], "invocation": {"module_args": {"assign_public_ip": true, "aws_access_key": null, "aws_secret_key": null, "count": 1, "count_tag": null, "ebs_optimized": false, "ec2_url": null, "exact_count": null, "group": null, "group_id": ["sg-692c4d13"], "id": null, "image": "ami-2d39803a", "instance_ids": null, "instance_profile_name": null, "instance_tags": null, "instance_type": "t2.micro", "kernel": null, "key_name": "hrlee", "monitoring": false, "network_interfaces": null, "placement_group": null, "private_ip": null, "profile": null, "ramdisk": null, "region": "us-east-1", "security_token": null, "source_dest_check": true, "spot_launch_group": null, "spot_price": null, "spot_type": "one-time", "spot_wait_timeout": "600", "state": "present", "tenancy": "default", "termination_protection": false, "user_data": null, "validate_certs": true, "volumes": null, "vpc_subnet_id": "subnet-719a774d", "wait": true, "wait_timeout": "300", "zone": null}, "module_name": "ec2"}, "tagged_instances": []}

  TASK [Add new instance to host group] ******************************************
  task path: /home/lee212/git/aws-cloudformation-by-ansible/play.yml:35
  Wednesday 14 September 2016  00:22:55 -0400 (0:00:22.170)       0:00:22.741 ***
  creating host via 'add_host': hostname=54.197.111.173
  changed: [localhost] => (item={u'kernel': None, u'root_device_type': u'ebs', u'private_dns_name': u'ip-172-30-3-157.ec2.internal', u'public_ip': u'54.197.111.173', u'private_ip': u'172.30.3.157', u'id': u'i-f7364c0f', u'ebs_optimized': False, u'state': u'running', u'virtualization_type': u'hvm', u'architecture': u'x86_64', u'ramdisk': None, u'block_device_mapping': {u'/dev/sda1': {u'status': u'attached', u'delete_on_termination': True, u'volume_id': u'vol-f678e477'}}, u'key_name': u'hrlee', u'image_id': u'ami-2d39803a', u'tenancy': u'default', u'groups': {u'sg-692c4d13': u'security-group-test'}, u'public_dns_name': u'', u'state_code': 16, u'tags': {}, u'placement': u'us-east-1a', u'ami_launch_index': u'0', u'dns_name': u'', u'region': u'us-east-1', u'launch_time': u'2016-09-14T04:22:34.000Z', u'instance_type': u't2.micro', u'root_device_name': u'/dev/sda1', u'hypervisor': u'xen'}) => {"add_host": {"groups": ["launched"], "host_name": "54.197.111.173", "host_vars": {}}, "changed": true, "invocation": {"module_args": {"groupname": "launched", "hostname": "54.197.111.173"}, "module_name": "add_host"}, "item": {"ami_launch_index": "0", "architecture": "x86_64", "block_device_mapping": {"/dev/sda1": {"delete_on_termination": true, "status": "attached", "volume_id": "vol-f678e477"}}, "dns_name": "", "ebs_optimized": false, "groups": {"sg-692c4d13": "security-group-test"}, "hypervisor": "xen", "id": "i-f7364c0f", "image_id": "ami-2d39803a", "instance_type": "t2.micro", "kernel": null, "key_name": "hrlee", "launch_time": "2016-09-14T04:22:34.000Z", "placement": "us-east-1a", "private_dns_name": "ip-172-30-3-157.ec2.internal", "private_ip": "172.30.3.157", "public_dns_name": "", "public_ip": "54.197.111.173", "ramdisk": null, "region": "us-east-1", "root_device_name": "/dev/sda1", "root_device_type": "ebs", "state": "running", "state_code": 16, "tags": {}, "tenancy": "default", "virtualization_type": "hvm"}}

  TASK [Wait for SSH to come up] *************************************************
  task path: /home/lee212/git/aws-cloudformation-by-ansible/play.yml:38
  Wednesday 14 September 2016  00:22:55 -0400 (0:00:00.035)       0:00:22.776 ***
  <127.0.0.1> ESTABLISH LOCAL CONNECTION FOR USER: lee212
  <127.0.0.1> EXEC /bin/sh -c '( umask 77 && mkdir -p "` echo $HOME/.ansible/tmp/ansible-tmp-1473826975.89-214245185864373 `" && echo ansible-tmp-1473826975.89-214245185864373="` echo $HOME/.ansible/tmp/ansible-tmp-1473826975.89-214245185864373 `" ) && sleep 0'
  <127.0.0.1> PUT /tmp/tmp2vGL53 TO /home/lee212/.ansible/tmp/ansible-tmp-1473826975.89-214245185864373/wait_for
  <127.0.0.1> EXEC /bin/sh -c 'LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8 LC_MESSAGES=en_US.UTF-8 /home/lee212/venv/ansible/bin/python /home/lee212/.ansible/tmp/ansible-tmp-1473826975.89-214245185864373/wait_for; rm -rf "/home/lee212/.ansible/tmp/ansible-tmp-1473826975.89-214245185864373/" > /dev/null 2>&1 && sleep 0'
  ok: [localhost] => (item={u'kernel': None, u'root_device_type': u'ebs', u'private_dns_name': u'ip-172-30-3-157.ec2.internal', u'public_ip': u'54.197.111.173', u'private_ip': u'172.30.3.157', u'id': u'i-f7364c0f', u'ebs_optimized': False, u'state': u'running', u'virtualization_type': u'hvm', u'architecture': u'x86_64', u'ramdisk': None, u'block_device_mapping': {u'/dev/sda1': {u'status': u'attached', u'delete_on_termination': True, u'volume_id': u'vol-f678e477'}}, u'key_name': u'hrlee', u'image_id': u'ami-2d39803a', u'tenancy': u'default', u'groups': {u'sg-692c4d13': u'security-group-test'}, u'public_dns_name': u'', u'state_code': 16, u'tags': {}, u'placement': u'us-east-1a', u'ami_launch_index': u'0', u'dns_name': u'', u'region': u'us-east-1', u'launch_time': u'2016-09-14T04:22:34.000Z', u'instance_type': u't2.micro', u'root_device_name': u'/dev/sda1', u'hypervisor': u'xen'}) => {"changed": false, "elapsed": 60, "invocation": {"module_args": {"connect_timeout": 5, "delay": 60, "exclude_hosts": null, "host": "54.197.111.173", "path": null, "port": 22, "search_regex": null, "state": "started", "timeout": 320}, "module_name": "wait_for"}, "item": {"ami_launch_index": "0", "architecture": "x86_64", "block_device_mapping": {"/dev/sda1": {"delete_on_termination": true, "status": "attached", "volume_id": "vol-f678e477"}}, "dns_name": "", "ebs_optimized": false, "groups": {"sg-692c4d13": "security-group-test"}, "hypervisor": "xen", "id": "i-f7364c0f", "image_id": "ami-2d39803a", "instance_type": "t2.micro", "kernel": null, "key_name": "hrlee", "launch_time": "2016-09-14T04:22:34.000Z", "placement": "us-east-1a", "private_dns_name": "ip-172-30-3-157.ec2.internal", "private_ip": "172.30.3.157", "public_dns_name": "", "public_ip": "54.197.111.173", "ramdisk": null, "region": "us-east-1", "root_device_name": "/dev/sda1", "root_device_type": "ebs", "state": "running", "state_code": 16, "tags": {}, "tenancy": "default", "virtualization_type": "hvm"}, "path": null, "port": 22, "search_regex": null, "state": "started"}

  PLAY RECAP *********************************************************************
  localhost                  : ok=4    changed=2    unreachable=0    failed=0

  Wednesday 14 September 2016  00:23:56 -0400 (0:01:00.194)       0:01:22.971 ***
  ===============================================================================
  Wait for SSH to come up ------------------------------------------------ 60.19s
  /home/lee212/git/aws-cloudformation-by-ansible/play.yml:38 --------------------
  ec2 launch test -------------------------------------------------------- 22.17s
  /home/lee212/git/aws-cloudformation-by-ansible/play.yml:22 --------------------
  Create security group --------------------------------------------------- 0.55s
  /home/lee212/git/aws-cloudformation-by-ansible/play.yml:5 ---------------------
  Add new instance to host group ------------------------------------------ 0.04s
  /home/lee212/git/aws-cloudformation-by-ansible/play.yml:35 --------------------



ec2.py 
-------------------------------------------------------------------------------

``ec2.py --list`` option displays cached data. Try with ``--refresh`` for update.

::

        $ ./ec2.py --list
        {
          "_meta": {
            "hostvars": {
              "54.197.206.131": {
                "ansible_ssh_host": "54.197.206.131", 
                "ec2__in_monitoring_element": false, 
                "ec2_ami_launch_index": "0", 
                "ec2_architecture": "x86_64", 
                "ec2_client_token": "", 
                "ec2_dns_name": "", 
                "ec2_ebs_optimized": false, 
                "ec2_eventsSet": "", 
                "ec2_group_name": "", 
                "ec2_hypervisor": "xen", 
                "ec2_id": "i-b4a8d24c", 
                "ec2_image_id": "ami-2d39803a", 
                "ec2_instance_profile": "", 
                "ec2_instance_type": "t2.micro", 
                "ec2_ip_address": "54.197.206.131", 
                "ec2_item": "", 
                "ec2_kernel": "", 
                "ec2_key_name": "hrlee", 
                "ec2_launch_time": "2016-09-14T14:52:55.000Z", 
                "ec2_monitored": false, 
                "ec2_monitoring": "", 
                "ec2_monitoring_state": "disabled", 
                "ec2_persistent": false, 
                "ec2_placement": "us-east-1a", 
                "ec2_platform": "", 
                "ec2_previous_state": "", 
                "ec2_previous_state_code": 0, 
                "ec2_private_dns_name": "ip-172-30-3-230.ec2.internal", 
                "ec2_private_ip_address": "172.30.3.230", 
                "ec2_public_dns_name": "", 
                "ec2_ramdisk": "", 
                "ec2_reason": "", 
                "ec2_region": "us-east-1", 
                "ec2_requester_id": "", 
                "ec2_root_device_name": "/dev/sda1", 
                "ec2_root_device_type": "ebs", 
                "ec2_security_group_ids": "sg-692c4d13", 
                "ec2_security_group_names": "security-group-test", 
                "ec2_sourceDestCheck": "true", 
                "ec2_spot_instance_request_id": "", 
                "ec2_state": "running", 
                "ec2_state_code": 16, 
                "ec2_state_reason": "", 
                "ec2_subnet_id": "subnet-719a774d", 
                "ec2_virtualization_type": "hvm", 
                "ec2_vpc_id": "vpc-e6c17c83"
              }
            }
          }, 
          "ami_2d39803a": [
            "54.197.206.131"
          ], 
          "ec2": [
            "54.197.206.131"
          ], 
          "i-b4a8d24c": [
            "54.197.206.131"
          ], 
          "key_hrlee": [
            "54.197.206.131"
          ], 
          "security_group_security_group_test": [
            "54.197.206.131"
          ], 
          "tag_none": [
            "54.197.206.131"
          ], 
          "type_t2_micro": [
            "54.197.206.131"
          ], 
          "us-east-1": [
            "54.197.206.131"
          ], 
          "us-east-1a": [
            "54.197.206.131"
          ], 
          "vpc_id_vpc_e6c17c83": [
            "54.197.206.131"
          ]
        }


You can compare with aws ec2 results::

        $ aws ec2 describe-instances
        {
            "Reservations": [
                {
                    "OwnerId": "461335111454", 
                    "ReservationId": "r-233f22da", 
                    "Groups": [], 
                    "Instances": [
                        {
                            "Monitoring": {
                                "State": "disabled"
                            }, 
                            "PublicDnsName": "", 
                            "State": {
                                "Code": 16, 
                                "Name": "running"
                            }, 
                            "EbsOptimized": false, 
                            "LaunchTime": "2016-09-14T14:52:55.000Z", 
                            "PublicIpAddress": "54.197.206.131", 
                            "PrivateIpAddress": "172.30.3.230", 
                            "ProductCodes": [], 
                            "VpcId": "vpc-e6c17c83", 
                            "StateTransitionReason": "", 
                            "InstanceId": "i-b4a8d24c", 
                            "ImageId": "ami-2d39803a", 
                            "PrivateDnsName": "ip-172-30-3-230.ec2.internal", 
                            "KeyName": "hrlee", 
                            "SecurityGroups": [
                                {
                                    "GroupName": "security-group-test", 
                                    "GroupId": "sg-692c4d13"
                                }
                            ], 
                            "ClientToken": "", 
                            "SubnetId": "subnet-719a774d", 
                            "InstanceType": "t2.micro", 
                            "NetworkInterfaces": [
                                {
                                    "Status": "in-use", 
                                    "MacAddress": "06:a5:49:cf:87:a1", 
                                    "SourceDestCheck": true, 
                                    "VpcId": "vpc-e6c17c83", 
                                    "Description": "", 
                                    "Association": {
                                        "PublicIp": "54.197.206.131", 
                                        "PublicDnsName": "", 
                                        "IpOwnerId": "amazon"
                                    }, 
                                    "NetworkInterfaceId": "eni-8ae97085", 
                                    "PrivateIpAddresses": [
                                        {
                                            "Association": {
                                                "PublicIp": "54.197.206.131", 
                                                "PublicDnsName": "", 
                                                "IpOwnerId": "amazon"
                                            }, 
                                            "Primary": true, 
                                            "PrivateIpAddress": "172.30.3.230"
                                        }
                                    ], 
                                    "Attachment": {
                                        "Status": "attached", 
                                        "DeviceIndex": 0, 
                                        "DeleteOnTermination": true, 
                                        "AttachmentId": "eni-attach-4243efba", 
                                        "AttachTime": "2016-09-14T14:52:55.000Z"
                                    }, 
                                    "Groups": [
                                        {
                                            "GroupName": "security-group-test", 
                                            "GroupId": "sg-692c4d13"
                                        }
                                    ], 
                                    "SubnetId": "subnet-719a774d", 
                                    "OwnerId": "461335111454", 
                                    "PrivateIpAddress": "172.30.3.230"
                                }
                            ], 
                            "SourceDestCheck": true, 
                            "Placement": {
                                "Tenancy": "default", 
                                "GroupName": "", 
                                "AvailabilityZone": "us-east-1a"
                            }, 
                            "Hypervisor": "xen", 
                            "BlockDeviceMappings": [
                                {
                                    "DeviceName": "/dev/sda1", 
                                    "Ebs": {
                                        "Status": "attached", 
                                        "DeleteOnTermination": true, 
                                        "VolumeId": "vol-21e37ca0", 
                                        "AttachTime": "2016-09-14T14:52:56.000Z"
                                    }
                                }
                            ], 
                            "Architecture": "x86_64", 
                            "RootDeviceType": "ebs", 
                            "RootDeviceName": "/dev/sda1", 
                            "VirtualizationType": "hvm", 
                            "AmiLaunchIndex": 0
                        }
                    ]
                }
            ]
        }

