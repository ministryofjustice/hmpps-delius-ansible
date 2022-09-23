# Bastion user onboarding

This page explains with examples how a user can be onboarded to use the delius bastion server(s) to gain ssh access to resources within the Delius AWS accounts. (The method of ssh used here is performed over an AWS SSM session).

## Pre-requisites
Accessing one or both bastions depends on the user having the following pre-requisites.
- Activities performed by a webops engineer
  - creation of an AWS user account, with MFA configured and programmatic access enabled
  - AWS user given membership of an IAM group(s) enabling MFA and access to connect to the dev bastion and/or prod bastion
- End user responsibilities
  - the AWS CLI installed (see https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
  - the Session Manager plugin for the AWS CLI (see https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html)
  - AWS CLI profile(s) configured for access (with details of AWS account ids and AWS role names supplied by a webops engineer). This will mean correct configuration of the AWS CLI configuration files - see https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html
  - Tested AWS login using the CLI with MFA, e.g. it should be possible to run `aws ssm describe-instance-information --profile <name of profile(s) set up above>` without error

## General process 
The general setup process is:
1. [SSH Key Pair Generation](#ssh-key-pair-generation) - user creates an SSH key pair for each bastion (dev and/or prod) they would like to access. (The public key is requested from the user by the engineer setting up the access. The private key always remains with the user).

2. [SSH Config](#ssh-config) - user updates their .ssh/config file
3. [User completes bastion setup](#complete-bastion-setup) - logs onto bastion server, changes passwords and deletes control file


## Helpful other information

- See [note on password expiry](#password-expiry)
- See [port-forwarding example](#port-forwarding-example), e.g. to database server
- See [notes for windows users](#notes-on-windows-ssh-access)


## SSH Key Pair Generation
* Mac/Linux
```bash
# Dev bastion - for accessing **non-prod** environments
ssh-keygen -t rsa -b 16384 -f ~/.ssh/moj_dev_rsa

# Prod bastion - for accessing **prod** environments
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

## SSH Config

* Mac/Linux

Replace `YOUR_USER_NAME_HERE` with your ssh username. This is usually `<first initial><surname>`, e.g. jbloggs
Replace `DEV_BASTION_INSTANCE_ID` or `PROD_BASTION_INSTANCE_ID` with the instance ids
Replace `ENG_DEV_PROFILE_NAME` with the AWS CLI profile name you're using to represent the Engineering Dev Account
Replace `ENG_PROD_PROFILE_NAME` with the AWS CLI profile name you're using to represent the Engineering Prod Account
```
Host *.delius-core-dev.internal *.delius.probation.hmpps.dsd.io *.delius-core.probation.hmpps.dsd.io 10.161.* 10.162.* !*.pre-prod.delius.probation.hmpps.dsd.io !*.stage.delius.probation.hmpps.dsd.io !*.perf.delius.probation.hmpps.dsd.io
  User YOUR_USER_NAME_HERE
  IdentityFile ~/.ssh/moj_dev_rsa
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  ProxyCommand ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no -W %h:%p moj_dev_bastion

Host ssh.bastion-dev.probation.hmpps.dsd.io moj_dev_bastion awsdevgw
  HostName ssh.bastion-dev.probation.hmpps.dsd.io
  ControlMaster auto
  ControlPath /tmp/ctrl_dev_bastion
  ServerAliveInterval 20
  ControlPersist 1h
  ForwardAgent yes
  User YOUR_USER_NAME_HERE
  IdentityFile ~/.ssh/moj_dev_rsa
  ProxyCommand sh -c "aws ssm start-session --target DEV_BASTION_INSTANCE_ID --profile ENG_DEV_PROFILE_NAME --document-name AWS-StartSSHSession --parameters 'portNumber=%p'"

## MOJ PROD - PRODUCTION DATA ENVS
Host *.probation.service.justice.gov.uk *.pre-prod.delius.probation.hmpps.dsd.io *.stage.delius.probation.hmpps.dsd.io 10.160.*
  User YOUR_USER_NAME_HERE
  IdentityFile ~/.ssh/moj_prod_rsa
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  ProxyCommand ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no -W %h:%p moj_prod_bastion

Host ssh.bastion-prod.probation.hmpps.dsd.io moj_prod_bastion awsprodgw
  HostName ssh.bastion-prod.probation.hmpps.dsd.io
  ControlMaster auto
  ControlPath /tmp/ctrl_prod_bastion
  ServerAliveInterval 20
  ControlPersist 1h
  ForwardAgent yes
  User YOUR_USER_NAME_HERE
  IdentityFile ~/.ssh/moj_prod_rsa
  ProxyCommand sh -c "aws ssm start-session --target PROD_BASTION_INSTANCE_ID  --profile ENG_PROD_PROFILE_NAME --document-name AWS-StartSSHSession --parameters 'portNumber=%p'"
```

* Windows

Replace `USER_NAME` with your ssh username. This is usually `<first initial><surname>`, e.g. jbloggs. Also replace `HOMEDIR` with the path to the home directory in Windows (e.g. `C:\Users\Joe.Bloggs`). 

Note: this example assumes the user doesn't have any pre-existing SSH config.

```
Host *
 User <b>USERNAME</b>
 ServerAliveInterval 20
 StrictHostKeyChecking no
 UserKnownHostsFile /dev/null

Host awsdevgw moj_dev_jump_host moj_dev_bastion ssh.bastion-dev.probation.hmpps.dsd.io
 Hostname ssh.bastion-dev.probation.hmpps.dsd.io
 IdentityFile <b>HOMEDIR</b>\.ssh\moj_dev_rsa
 ProxyCommand sh -c "aws ssm start-session --target DEV_BASTION_INSTANCE_ID --profile ENG_DEV_PROFILE_NAME --document-name AWS-StartSSHSession --parameters 'portNumber=%p'"

Host awsprodgw moj_prod_jump_host moj_prod_bastion ssh.bastion-prod.probation.hmpps.dsd.io
 Hostname ssh.bastion-prod.probation.hmpps.dsd.io
 IdentityFile <b>HOMEDIR</b>\.ssh\moj_prod_rsa
 ProxyCommand sh -c "aws ssm start-session --target PROD_BASTION_INSTANCE_ID --profile ENG_PROD_PROFILE_NAME --document-name AWS-StartSSHSession --parameters 'portNumber=%p'"

Host *.probation.hmpps.dsd.io !*.stage.delius.probation.hmpps.dsd.io !*.pre-prod.delius.probation.hmpps.dsd.io !*.perf.delius.probation.hmpps.dsd.io 10.16* !10.160.?.* !10.160.1?.* !10.160.2?.* !10.160.3?.* !10.160.4?.* !10.160.5?.*
 ForwardAgent yes
 ProxyCommand ssh -W %h:%p moj_dev_bastion
 IdentityFile <b>HOMEDIR</b>\.ssh\moj_dev_rsa

Host *.stage.delius.probation.hmpps.dsd.io *.pre-prod.delius.probation.hmpps.dsd.io *.perf.delius.probation.hmpps.dsd.io *.probation.service.justice.gov.uk 10.160.?.* 10.160.1?.* 10.160.2?.* 10.160.3?.* 10.160.4?.* 10.160.5?.* 
 ForwardAgent yes
 ProxyCommand ssh -W %h:%p moj_prod_bastion
 IdentityFile <b>HOMEDIR</b>\.ssh\moj_prod_rsa
```


## Complete bastion setup

The user should now
1. Log onto the bastion server, e.g. ```ssh moj_dev_bastion``` or ```ssh moj_prod_bastion``` 
2. Enter initial password and follow prompts to change it immediately
3. Exit the bastion
4. [Delete the control file](#delete-the-control-file) as identified by the ControlPath option in the user's ssh config file
5. Log back onto the bastion to confirm access using the new password

### Delete the control file
```
rm /tmp/ctrl_dev_bastion
# or
/tmp/ctrl_prod_bastion
```

### Password expiry 
Password expires after 60 days. To reset after expiry, the user should choose a new password, exit and delete the control file before logging back in.

### Port-forwarding example
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
* Putty can be used but no support is provided to users for its configuration. However, in order to set Putty up, the following is required
  * In Connection -> Proxy, select "Local" radio button
  * Within "Telnet command, or local proxy command", enter `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe “aws ssm start-session --target %host --document-name AWS-StartSSHSession --parameters portNumber=%port`
  * Tunnelling/port forwarding can be set up in Connection -> SSH -> Auth -> Tunnels
