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


## Enable/Disable Civica User access

Please see [the CivicaOnOffboarding README](/CivicaOnOffboarding.md)
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
* Mac/Linux
```bash
# MOJ DEV - NON-PROD
ssh-keygen -t rsa -b 16384 -f ~/.ssh/moj_dev_rsa

# MOJ PROD - PRODUCTION DATA ENVS
ssh-keygen -t rsa -b 16384 -f ~/.ssh/moj_prod_rsa
```

* Windows
```cmd
# Create the .ssh directory (required on windows)
mkdir %USERPROFILE%/.ssh

# Generate dev and prod keypairs
ssh-keygen -t rsa -b 16384 -f %USERPROFILE%/.ssh/moj_dev_rsa
ssh-keygen -t rsa -b 16384 -f %USERPROFILE%/.ssh/moj_prod_rsa
```

## SSH Config Example

Replace `USERNAME` with the SSH username (e.g. `jbloggs`).

* Mac/Linux
<pre>
Host *
 User <b>USERNAME</b>
 ControlPersist yes
 ControlMaster auto
 ControlPath /tmp/ssh_control_%r@%h:%p
 ServerAliveInterval 20
 UserKnownHostsFile /dev/null
 StrictHostKeyChecking no

Host awsdevgw moj_dev_jump_host moj_dev_bastion ssh.bastion-dev.probation.hmpps.dsd.io
 Hostname ssh.bastion-dev.probation.hmpps.dsd.io
 IdentityFile ~/.ssh/moj_dev_rsa

Host awsprodgw moj_prod_jump_host moj_prod_bastion ssh.bastion-prod.probation.hmpps.dsd.io
 Hostname ssh.bastion-prod.probation.hmpps.dsd.io
 IdentityFile ~/.ssh/moj_prod_rsa

Host *.probation.hmpps.dsd.io !*.stage.delius.probation.hmpps.dsd.io !*.pre-prod.delius.probation.hmpps.dsd.io !*.perf.delius.probation.hmpps.dsd.io 10.16* !10.160.?.* !10.160.1?.* !10.160.2?.* !10.160.3?.* !10.160.4?.* !10.160.5?.*
 ForwardAgent yes
 ProxyCommand ssh -W %h:%p moj_dev_bastion
 IdentityFile ~/.ssh/moj_dev_rsa

Host *.stage.delius.probation.hmpps.dsd.io *.pre-prod.delius.probation.hmpps.dsd.io *.perf.delius.probation.hmpps.dsd.io *.probation.service.justice.gov.uk 10.160.?.* 10.160.1?.* 10.160.2?.* 10.160.3?.* 10.160.4?.* 10.160.5?.* 
 ForwardAgent yes
 ProxyCommand ssh -W %h:%p moj_prod_bastion
 IdentityFile ~/.ssh/moj_prod_rsa
</pre>

* Windows

Replace `HOMEDIR` with the path to the home directory in Windows (e.g. `C:\Users\Joe.Bloggs`)

<pre>
Host *
 User <b>USERNAME</b>
 ServerAliveInterval 20
 StrictHostKeyChecking no
 UserKnownHostsFile /dev/null

Host awsdevgw moj_dev_jump_host moj_dev_bastion ssh.bastion-dev.probation.hmpps.dsd.io
 Hostname ssh.bastion-dev.probation.hmpps.dsd.io
 IdentityFile <b>HOMEDIR</b>\.ssh\moj_dev_rsa

Host awsprodgw moj_prod_jump_host moj_prod_bastion ssh.bastion-prod.probation.hmpps.dsd.io
 Hostname ssh.bastion-prod.probation.hmpps.dsd.io
 IdentityFile <b>HOMEDIR</b>\.ssh\moj_prod_rsa

Host *.probation.hmpps.dsd.io !*.stage.delius.probation.hmpps.dsd.io !*.pre-prod.delius.probation.hmpps.dsd.io !*.perf.delius.probation.hmpps.dsd.io 10.16* !10.160.?.* !10.160.1?.* !10.160.2?.* !10.160.3?.* !10.160.4?.* !10.160.5?.*
 ForwardAgent yes
 ProxyCommand ssh -W %h:%p moj_dev_bastion
 IdentityFile <b>HOMEDIR</b>\.ssh\moj_dev_rsa

Host *.stage.delius.probation.hmpps.dsd.io *.pre-prod.delius.probation.hmpps.dsd.io *.perf.delius.probation.hmpps.dsd.io *.probation.service.justice.gov.uk 10.160.?.* 10.160.1?.* 10.160.2?.* 10.160.3?.* 10.160.4?.* 10.160.5?.* 
 ForwardAgent yes
 ProxyCommand ssh -W %h:%p moj_prod_bastion
 IdentityFile <b>HOMEDIR</b>\.ssh\moj_prod_rsa
</pre>


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

### Notes on Windows SSH access

* OpenSSH client is installed by default in Windows 10 as of April 2018, and no longer needs to be enabled manually.
  This allows the same SSH commands to be used as MacOS/Linux with minimal changes.
* If the built-in OpenSSH client does not work, try using ssh from [Git Bash](https://git-scm.com/downloads) instead.
* Using `~` for the output paths in the `ssh-keygen` commands may cause issues.
  If you get *No such file or directory* errors, try referencing the full path instead (eg. `C:\Users\username\.ssh\moj_dev_rsa`).
* In the ProxyCommand lines in `.ssh/config`, `ssh` must be replaced with `ssh.exe`.
* When using control sockets, you may get the error *getsockname failed: Not a socket*.
  If so, the easiest thing to do is remove the `ControlMaster`, `ControlPath` and `ControlPersist` lines from `.ssh/config`.
  Note: the users' SSH password will then need to be entered on each new connection.

## PRE-COMMIT Hook

This project supports protecting yaml files from bad formatting with the "pre-commit" tool.
Install on local machine using instructions: ( https://pre-commit.com )


Before you can run hooks, you need to have the pre-commit package manager installed.

Using pip:
```
pip install pre-commit
```
Non-administrative installation:

to upgrade: run again, to uninstall: pass uninstall to python
does not work on platforms without symlink support (windows)
```
curl https://pre-commit.com/install-local.py | python -
```
In a python project, add the following to your requirements.txt (or requirements-dev.txt):

pre-commit
Using homebrew:
```
brew install pre-commit
```
Using conda (via conda-forge):
```
conda install -c conda-forge pre-commit
```

Then from root of project run
```
pre-commit install
```

Now you should have the pre-commit hook running whwn you commit.

To check prior to committing you can use this command
```
pre-commit run --all-files
```
