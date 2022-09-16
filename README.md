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

Role Tasks and Variables
--------------

When running this role from a Tower environment or running from cmdline, the following vars are required and/or overwriteable. This is a breakdown per task:

**Task:** `mq-add-pub-cert.yaml`<br>
Description: This task allows for adding a trusted cert to a keystore.<br>
Variables:
  - `cert_label` - The label the cert will be known by in the keystore.
  - `cert_body` - The actual cert itself.
  - `email_address` - The email address to send any updates for this task.
  - `github_issue` - The github issue number we want to update with our status on this job.

Tags: `addpubcert`

**Task:** `mq-create-keystore.yaml`<br>
Description: This task will create a new keystore for a queue manager and configure the qm to use it.<br>
Variables:
  - `email_address` - The email address to send any updates for this task.
  - `github_issue` - The github issue number we want to update with our status on this job.

Tags: `createkeystore`

**Task:** `mq-getcerts.yaml`<br>
Description: Get a list of all certs that exist in a keystore and send a formatted email with them broken down between Trusted, Personal, and any generated CSRs.<br>
Variables:
  - `email_address` - The email address to send any updates for this task.

Tags: `getallcerts`

**Task:** `mq-get-cert-expirations.yaml`<br>
Description: Get a list of all certs that have imminent or expired dates.<br>
Variables:
  - `email_address` - The email address to send any updates for this task.

Tags: `listexpires`

**Task:** `mq-create-cert.yaml`<br>
Description: Create a new personal cert for the keystore and then extract and email the public cert so we can add it to any SSL peer and update any associated Github issue number with what we've done.<br>
Variables:
  - `cert_label` - The label the cert will be known by in the keystore.
  - `email_address` - The email address to send any updates for this task.
  - `github_issue` - The github issue number we want to update with our status on this job.

Tags: `createcert`

**Task:** `mq-get-pub-cert.yaml`<br>
Description: Extract and email the trusted cert for the named cert label we give.<br>
Variables:
  - `cert_label` - The label the cert will be known by in the keystore.
  - `email_address` - The email address to send any updates for this task.

Tags: `getpubcert`

**Task:** `mq-remove-pub-cert.yaml`<br>
Description: Delete a Trusted cert from the keystores<br>
Variables: 
  - `cert_label` - The label the cert will be known by in the keystore.
  - `email_address` - The email address to send any updates for this task.
  - `github_issue` - The github issue number we want to update with our status on this job.

Tags: `certremove`

**Task:** `mq-get-info.yaml`<br>
Description: This is a required task that is called via tags by any of the other tasks. It retrieves information from the keystore that is used by the other tasks. This task isn't typically called directly. The type of info it returns are as such:
  - Pulls the current keyfile path that is set in the queue manager
  - Pulls the current cert label that is set in the queue manager
  - Pulls the list of trusted certs
  - Pulls thelist of personal certs
  - Pulls the list of certs with expirations
  - Pulls a list of any unsigned CSRs that were created in the keystore

  Tags: `getinfo, getpubcert, certremove, addpubcert, listexpires, createcert, getallcerts`



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
