# hmpps-delius-ansible
Ansible repo for instance control

# Bastion

## Adding users

First add the user to the file for the correct environment

Either

    group_vars/dev

or

    group_vars/prod

NOTE: Users must not have the same key for both environments.

To create a 16k rsa key:
    ssh-keygen -t rsa -b 16384  -C "you@youremail.com" -f hmpps-bastion-prod.key


Once the new user is created login to the bastion and run the
the following to set an inital password and for the user to
change their password on next login.

    sudo passwd username
    sudo passwd -e username


## Ansible playbook

First you will need all the external roles.

    make deps

Then you will need to create a control connection to the environment bastion.
Either

    ssh -M -S ~/.ssh/control/bastion ssh.bastion-dev.probation.hmpps.dsd.io

or

    ssh -M -S ~/.ssh/control/bastion ssh.bastion-prod.probation.hmpps.dsd.io

You can now use to the control connection to run the environment playbook

Either

    make dev

or

    make prod

------

# NOTES to assist users.

These simple snippets can make it easier to onboard a new user.

## SSH Key Pair Generation

    ## MOJ DEV - NON-PROD
    ssh-keygen -t rsa -b 16384 -f ~/.ssh/moj_dev_rsa

    ## MOJ PROD - PRODUCTION DATA ENVS
    ssh-keygen -t rsa -b 16384 -f ~/.ssh/moj_prod_rsa

## SSH Config Example

```
Host *.delius-core-dev.internal *.delius.probation.hmpps.dsd.io *.delius-core.probation.hmpps.dsd.io 10.161.* 10.162.* !*.pre-prod.delius.probation.hmpps.dsd.io !*.stage.delius.probation.hmpps.dsd.io
  User YOUR_USER_NAME_HERE
  IdentityFile ~/.ssh/moj_dev_rsa
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  ProxyCommand ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no -W %h:%p moj_dev_bastion

Host ssh.bastion-dev.probation.hmpps.dsd.io moj_dev_bastion
  HostName ssh.bastion-dev.probation.hmpps.dsd.io
  ControlMaster auto
  ControlPath /tmp/ctrl_dev_bastion
  ServerAliveInterval 20
  ControlPersist 1h
  ForwardAgent yes
  User YOUR_USER_NAME_HERE
  IdentityFile ~/.ssh/moj_dev_rsa
  ProxyCommand none

## MOJ PROD - PRODUCTION DATA ENVS

Host *.probation.service.justice.gov.uk *.pre-prod.delius.probation.hmpps.dsd.io *.stage.delius.probation.hmpps.dsd.io 10.160.*
  User YOUR_USER_NAME_HERE
  IdentityFile ~/.ssh/moj_prod_rsa
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  ProxyCommand ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no -W %h:%p moj_prod_bastion

Host ssh.bastion-prod.probation.hmpps.dsd.io moj_prod_bastion
  HostName ssh.bastion-prod.probation.hmpps.dsd.io
  ControlMaster auto
  ControlPath /tmp/ctrl_prod_bastion
  ServerAliveInterval 20
  ControlPersist 1h
  ForwardAgent yes
  User YOUR_USER_NAME_HERE
  IdentityFile ~/.ssh/moj_prod_rsa
  ProxyCommand none
```

## User Notes

After creating and providing their intial password using the above SSH config they should be able to access the bastion to login and immediateley change it.
Then they will need to
1. exit
2. delete the control file
3. test access

```
rm /tmp/ctrl_dev_bastion
or
/tmp/ctrl_prod_bastion
```

Password expires after 60 days. So will need changing then exiting and deleting the control file before logging back in.

### Local tunnel to DB

```
ssh -L localhost:1521:localhost:1521 delius-db-1.test.delius.probation.hmpps.dsd.io
```
### todo// prehook
