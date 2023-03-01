# hmpps-delius-ansible
Ansible repo for instance control

## Bastion access - notes for users

Please see [the BastionUserOnboarding-SSH-over-SSM README](/BastionUserOnboarding-SSH-over-SSM.md) for specific information about how to set up access to the dev and/or prod bastion.

## Bastion access - engineering notes

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
