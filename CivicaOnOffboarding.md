#### Civica Users - Enabling and Disabling Access

We enable/disable user ssh permissions via add/remove of their group memberships.

The user will still be able to ssh onto the bastion and target servers but won't be able to perform any actions.

##### Example - Access Enabled:
```yaml
- username: auser
    name: "Anthony User"
    groups:
      - vcms
    ssh_key:
      - "..."
    shell: /bin/bash
    update_password: on_create
```

##### Example - Access Disabled:
(we just comment out the groups item in the username entry)
```yaml
- username: auser
    name: "Anthony User"
    # groups:
    #   - vcms
    ssh_key:
      - "..."
    shell: /bin/bash
    update_password: on_create
```
