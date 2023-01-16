# Bastion access user instructions
- [Bastion access user instructions](#bastion-access-user-instructions)
    - [A note on ssh vs ssh-over-ssm](#a-note-on-ssh-vs-ssh-over-ssm)
    - [Deadline for dev bastion access cutover from ssh to ssh-over-ssm: 18 November 2022]
    (#deadline-for-dev-bastion-access-cutover-from-ssh-to-ssh-over-ssm-18-november-2022)
    - [Deadline for prod bastion access cutover from ssh to ssh-over-ssm: 14 February 2022]
    (#deadline-for-prod-bastion-access-cutover-from-ssh-to-ssh-over-ssm-14-February-2022)
  - [Pre-requisites (required for ssh-over-ssm)](#pre-requisites---required-for-ssh-over-ssm)
  - [General process](#general-process)
  - [Helpful other information](#helpful-other-information)
  - [SSH Key Pair Generation](#ssh-key-pair-generation)
  - [SSH Config for DEV bastion](#ssh-config-for-dev-bastion)
  - [SSH Config for PROD bastion](#ssh-config-for-prod-bastion)
  - [Complete bastion setup](#complete-bastion-setup)
    - [Delete the control file](#delete-the-control-file)
    - [Password expiry](#password-expiry)
    - [Port-forwarding example](#port-forwarding-example)
    - [Notes on Windows SSH access](#notes-on-windows-ssh-access)


The content on this page is designed to help users obtain ssh access to the dev and/or prod Delius bastion.

### A note on ssh vs ssh-over-ssm
We are currently in a transition period where access to the bastions is taking a more secure route, namedly ssh over an AWS SSM connection. The content on this page reflects the current situation at this time in the transition period. This current transition period takes the form of the prod bastion access using ssh-over-ssm. Note that roll out of ssh-over-ssm to the dev bastion is already fully completed.

Because of this transition period with dev already rolled out of ssh-over-ssm and prod currently rolling out of ssh-over-ssm, please note the following.

Note I. **For users who already have access to dev bastion**, they will need to ensure they have the pre-requisites listed below, then they will need to update their SSH config setup and also test their bastion connection as described later on in this document. 

Note II. **For users who already have access to prod bastion**, they will need to ensure they have the pre-requisites listed below, then they will need to update their SSH config setup and also test their bastion connection as described later on in this document. After the deadline, stated below, port 22 over the public internet for the prod bastion server will cease to work.

Note III. **It is the users' responsibility to go through this document** and ensure they update their ssh config setup and test their bastion connection with the new configuration before the set deadline.

### Deadline for dev bastion access cutover from ssh to ssh-over-ssm: 18 November 2022 -- step completed

### Deadline for prod bastion access cutover from ssh to ssh-over-ssm: 14 February 2023

The rest of this page describes how to set up access for the dev and/or prod bastion.
<br /><br /><br />

## Pre-requisites (required for ssh-over-ssm)
- Activities performed by a webops engineer
  - Ensure user has an AWS user account, with MFA configured and programmatic access enabled 
  - AWS user is a member of the relevant IAM group enabling MFA and access to connect to the bastion
- End user responsibilities
  - the AWS CLI installed (see https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
  - the Session Manager plugin for the AWS CLI (see https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html)
  - AWS CLI profile(s) configured for access (with details of AWS account ids and AWS role names supplied by a webops engineer). This will mean correct configuration of the AWS CLI configuration files - see https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html for general information about CLI configuration, with a [specific example here](/BastionUserOnboarding-AWS-CLI-Setup.md)
  - Tested AWS login using the CLI with MFA. Run the following AWS CLI command to test your ability to run CLI commands. This command also retrieves the dev/prod bastion instance id (referred to as `DEV_BASTION_INSTANCE_ID` or `PROD_BASTION_INSTANCE_ID`), which is used in later steps
  ```
  aws ssm describe-instance-information --profile <your-chosen-aws-profile-name> | jq -c '.InstanceInformationList[] | select(.ComputerName | contains("bastion-dev")) | .InstanceId '
  ``` 
  For users with PROD bastion access only, try
  ```
  aws ssm describe-instance-information --profile <your-chosen-aws-profile-name> | jq -c '.InstanceInformationList[] | select(.ComputerName | contains("bastion-prod")) | .InstanceId '
  ``` 

## General process 
The general setup process is:
1. [SSH Key Pair Generation](#ssh-key-pair-generation) - user creates an SSH key pair for each bastion (dev and/or prod) they would like to access. (The public key is requested from the user by the engineer setting up the access. The private key always remains with the user).

2. [SSH Config for DEV bastion](#ssh-config-for-dev-bastion) and/or [SSH Config for PROD bastion](#ssh-config-for-prod-bastion) - user updates their .ssh/config file
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

## SSH Config for DEV bastion

* Mac/Linux

Replace `YOUR_USER_NAME_HERE` with your ssh username. This is usually `<first initial><surname>`, e.g. jbloggs.<br />
Replace `DEV_BASTION_INSTANCE_ID` with the instance id. You can retrieve the dev bastion instance id by running
```
aws ssm describe-instance-information --profile <your-chosen-aws-profile-name> | jq -c '.InstanceInformationList[] | select(.ComputerName | contains("bastion-dev")) | .InstanceId '
```
Replace `ENG_DEV_PROFILE_NAME` with the AWS CLI profile name you're using to represent the Engineering Dev Account<br /><br />

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
```

* Windows

Replace `USER_NAME` with your ssh username. This is usually `<first initial><surname>`, e.g. jbloggs.<br />
Replace `HOMEDIR` with the path to the home directory in Windows (e.g. `C:\Users\Joe.Bloggs`).<br />
Replace `DEV_BASTION_INSTANCE_ID` with the instance id. You can retrieve the dev bastion instance id by running
```
aws ssm describe-instance-information --profile <your-chosen-aws-profile-name> | jq -c '.InstanceInformationList[] | select(.ComputerName | contains("bastion-dev")) | .InstanceId '
```
Replace `ENG_DEV_PROFILE_NAME` with the AWS CLI profile name you're using to represent the Engineering Dev Account<br /><br />

```
Host *
 User <b>USERNAME</b>
 ServerAliveInterval 20
 StrictHostKeyChecking no
 UserKnownHostsFile /dev/null

Host awsdevgw moj_dev_jump_host moj_dev_bastion ssh.bastion-dev.probation.hmpps.dsd.io
 Hostname ssh.bastion-dev.probation.hmpps.dsd.io
 IdentityFile <b>HOMEDIR</b>\.ssh\moj_dev_rsa
 ProxyCommand C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe "aws ssm start-session --target DEV_BASTION_INSTANCE_ID --profile ENG_DEV_PROFILE_NAME --document-name AWS-StartSSHSession --parameters portNumber=%p"

Host *.probation.hmpps.dsd.io !*.stage.delius.probation.hmpps.dsd.io !*.pre-prod.delius.probation.hmpps.dsd.io !*.perf.delius.probation.hmpps.dsd.io 10.16* !10.160.?.* !10.160.1?.* !10.160.2?.* !10.160.3?.* !10.160.4?.* !10.160.5?.*
 ForwardAgent yes
 ProxyCommand ssh -W %h:%p moj_dev_bastion
 IdentityFile <b>HOMEDIR</b>\.ssh\moj_dev_rsa
```

## SSH Config for PROD bastion

* Mac/Linux

Replace `YOUR_USER_NAME_HERE` with your ssh username. This is usually `<first initial><surname>`, e.g. jbloggs<br />
Replace `PROD_BASTION_INSTANCE_ID` with the instance id. You can retrieve the prod bastion instance id by running
```
aws ssm describe-instance-information --profile <your-chosen-aws-profile-name> | jq -c '.InstanceInformationList[] | select(.ComputerName | contains("bastion-prod")) | .InstanceId '
```
Replace `ENG_PROD_PROFILE_NAME` with the AWS CLI profile name you're using to represent the Engineering Prod Account<br /><br />

```
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
  ProxyCommand sh -c "aws ssm start-session --target PROD_BASTION_INSTANCE_ID --profile ENG_PROD_PROFILE_NAME --document-name AWS-StartSSHSession --parameters 'portNumber=%p'"
```

* Windows

Replace `USER_NAME` with your ssh username. This is usually `<first initial><surname>`, e.g. jbloggs. <br />
Replace `HOMEDIR` with the path to the home directory in Windows (e.g. `C:\Users\Joe.Bloggs`).<br />
Replace `PROD_BASTION_INSTANCE_ID` with the instance id. You can retrieve the prod bastion instance id by running
```
aws ssm describe-instance-information --profile <your-chosen-aws-profile-name> | jq -c '.InstanceInformationList[] | select(.ComputerName | contains("bastion-prod")) | .InstanceId '
```
Replace `ENG_PROD_PROFILE_NAME` with the AWS CLI profile name you're using to represent the Engineering Dev Account<br /><br />

```
Host *
 User <b>USER_NAME</b>
 ServerAliveInterval 20
 StrictHostKeyChecking no
 UserKnownHostsFile /dev/null

Host awsprodgw moj_prod_jump_host moj_prod_bastion ssh.bastion-prod.probation.hmpps.dsd.io
 Hostname ssh.bastion-prod.probation.hmpps.dsd.io
 IdentityFile <b>HOMEDIR</b>\.ssh\moj_prod_rsa
ProxyCommand C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe "aws ssm start-session --target PROD_BASTION_INSTANCE_ID --profile ENG_PROD_PROFILE_NAME --document-name AWS-StartSSHSession --parameters portNumber=%p"

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
* Putty can be used but no support is provided to users for its configuration. However, in order to set Putty up to use ssh-over-ssm, the following is required
  * In Connection -> Proxy, select "Local" radio button
  * Within "Telnet command, or local proxy command", enter `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe “aws ssm start-session --target %host --document-name AWS-StartSSHSession --parameters portNumber=%port`
  * Tunnelling/port forwarding can be set up in Connection -> SSH -> Auth -> Tunnels
