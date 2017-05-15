# iscsi-target

iSCSI target role that installs and uses targetcli.

## Requirements

An apt- or yum-based package manager and systemd. 
Also, this role is not creating any disks/partitions/LVs. Therefore it is expected that they are already present on machine or created by some other role.

## Role Variables

| Name                  | mandatory  | Description                                                                                  |
|-----------------------|------------|---------------------------------------------------------------------------------------------|
| `iscsi_base_wwn`              | yes  | wwn that identifies the target host |
| `iscsi_disk_path_prefix`       | no | optional prefix to exported disk paths                                    |
| `iscsi_targets`     | yes | list of targets to be configured, see [targets](#targets)                                              |


### targets

each target has the following mandatory vars:

| Name                  | mandatory | Description                                                                            |
|-----------------------|--------|-----------------------------------------------------------------------------------------|
| `name`                | yes |`name` identifier of this target. This is appended to `iscsi_base_wwn` and used as `wwn`    |
| `disks`               | yes | disk configuration, see [disks](#disks)                                                    |
| `initiators`          | yes | list of `initiator`-`wwn`s that are allowed to access this target                          |
| `authentication`      | yes | authentication configuration, see [authentication](#authentication)                        |
| `portal_ip`           | yes | local ip on which access to this target is allowed                                         |
| `portal_port`         | no | iSCSI port, omit to use default port 3260                                                   |


### disks

a list of dicts with the following mandatory entries:

| Name                  | Description                                                                                             |
|-----------------------|---------------------------------------------------------------------------------------------------------|
| `name`                | name that is used for the backstore                                                                     |
| `path`                | existing path to the disk that should be used as backstore. `iscsi_disk_path_prefix` will be prepended if defined. |
| `type`                | one out of {`fileio`, `iblock`, `pscsi`, `rd_mcp`}                                                      |

### authentication

`authentication` is a dict containing the following entries:

| Name                  | mandatory  | Description                                                                                  |
|-----------------------|------------|---------------------------------------------------------------------------------------------|
| `userid`              | yes  | userid to authenticate the initator |
| `password`            | yes | password to authenticate the initiator                                                |
| `userid_mutual`       | no | mutual userid to authenticate the target                                      |
| `password_mutual`     | no | mutual password to authenticate the target                                                 |



## Dependencies

This role depends on the ansible [targetcli modules](https://github.com/stuvusIT/targetcli-modules).

## Example Playbook
```yml
- hosts: storage
  roles:
    - role: iscsi-target
      iscsi_base_wwn: iqn.1994-05.com.redhat
      iscsi_disk_path_prefix: /dev/zvol/tank/vms/
      iscsi_targets:
        - name: target
          disks:
            - name: vm1
              path: testing/vm1
              type: iblock
            - name: vm2
              path: testing/vm2
              type: iblock
          initiators:
            - iqn.1994-05.com.redhat:client1
            - iqn.1994-05.com.redhat:client2
          authentication:
            userid: myuser
            password: mypassword
            userid_mutual: sharedkey
            password_mutual: sharedsecret
          portal_ip: 192.168.1.45
```

## License

GPLv3

## Author Information
* original author: [Ondrej Faměra (OndrejHome)](https://github.com/OndrejHome/) _ondrej-xa2iel8u@famera.cz_
* modified by [Michel Weitbrecht (SlothOfAnarchy)](https://github.com/SlothOfAnarchy) _michel.weitbrecht@stuvus.uni-stuttgart.de_
