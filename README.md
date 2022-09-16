MQ Cert Managment
=========

This role allows for simple certificate management in the local keystore on each node of MQ. This works for any RDQM cluster or standalone MQ host.

Requirements
------------

At the moment this role requires a lot of specific settings in the inventory:

```
[mqipt_hosts]
mqipt ansible_host=<MQIPT HOST IP> ansible_user=<SYSTEM USER FOR ANSIBLE>

[washington]
ceng-wdc1-mq1 ansible_host=<HOST IP 1> mq_role=primary
ceng-wdc2-mq1 ansible_host=<HOST IP 2> mq_role=standby
ceng-wdc3-mq1 ansible_host=<HOST IP 3> mq_role=standby

[dallas]
ceng-dal1-mq1 ansible_host=<HOST IP 4> mq_role=primary
ceng-dal2-mq1 ansible_host=<HOST IP 5> mq_role=standby
ceng-dal3-mq1 ansible_host=<HOST IP 6> mq_role=standby

[washington:vars]
ansible_port=22
ansible_user=<ANSIBLE USER IF DIFFERENT>
queuemgr='<QUEUE MANAGER ON THIS CLUSTER>'
keypath='/var/mqm/vols/<QM>/qmgr/<QM>/ssl/defaultSSLKeyFile.kdb'
private_key_file=keys/<PATHTOSSHKEY>_pem
load_balancer="<OUR REGIONAL LOADBALANCER FOR THIS CLUSTER>"
## Add this line to inventory if using a bastion host as ssh proxy
ansible_ssh_common_args='-l <USER> -i <path to ssh pem> -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -o ProxyCommand="ssh -l <USER> -i <path to pem> -W %h:%p -q <BASTION HOST IP"'

[dallas:vars]
ansible_port=22
ansible_user=<ANSIBLE USER IF DIFFERENT>
queuemgr='<QUEUE MANAGER ON THIS CLUSTER>'
keypath='/var/mqm/vols/<QM>/qmgr/<QM>/ssl/defaultSSLKeyFile.kdb'
private_key_file=keys/<PATHTOSSHKEY>_pem
load_balancer="<OUR REGIONAL LOADBALANCER FOR THIS CLUSTER>"
## Add this line to inventory if using a bastion host as ssh proxy
ansible_ssh_common_args='-l <USER> -i <path to ssh pem> -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -o ProxyCommand="ssh -l <USER> -i <path to pem> -W %h:%p -q <BASTION HOST IP"'
```

In our example above, we define the hosts and we set a `primary` node for each cluster. This isn't conducive to failovers in this revision of the role.

We also set a `queuemgr` for each group as it's specific to that cluster. The milage may vary on different types of MQ configurations, but this is a variable that can be overwritten at execution time.

Also in our example, ansible will connect as the root user. For future use, always configure a local user with the correct sudo permissions. This also necessitates the need to define a path to the ssh private key for ansible to use.

The `ansible_ssh_common_args` is used for ansible to connect if you are using a bastion host and not running on the same secured network

When importing this role, it's important to also note that it is useful to have the following

Role Variables
--------------

When running this role from a Tower environment or running from cmdline, the following vars are required and/or overwriteable:

- 

Dependencies
------------

A list of other roles hosted on Galaxy should go here, plus any details in regards to parameters that may need to be set for other roles, or variables that are used from other roles.

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
         - { role: username.rolename, x: 42 }

License
-------

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
