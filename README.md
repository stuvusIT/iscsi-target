# iscsi-target

iSCSI target role that installs and uses targetcli.

## Requirements

An apt- or yum-based package manager and systemd. 
Also, this role is not creating any disks/partitions/LVs. Therefore it is expected that they are already present on machine or created by some other role.

## Role Variables

A `iscsi_base_wwn` variable that identifies the host and `iscsi_targets` which contains a list of entries containing the following mandatory entries:

| Name                  | Description                                                                                 |
|-----------------------|---------------------------------------------------------------------------------------------|
| `name`                | `name` identifier of this target. This is appended to `iscsi_base_wwn` and used as `wwn`    |
| `disks`               | disk configuration, see [below](#disks)                                                     |
| `initiators`          | list of `initiator`-`wwn`s                                                                  |
| `authentication`      | authentication configuration, see [below](#authentication)                                  |

Optionally, a `iscsi_disk_path_prefix` var can be added at the top level and will be prepended before each disk `path`.

### disks

a list of dicts with the following mandatory entries:

| Name                  | Description                                                                                             |
|-----------------------|---------------------------------------------------------------------------------------------------------|
| `path`                | existing path to the disk that should be used as backstore. `iscsi_disk_path_prefix` will be prepended if defined. |
| `name`                | name that is used for the backstore                                                                     |
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
```

## License

GPLv3

## Author Information
* original author: [Ondrej Faměra (OndrejHome)](https://github.com/OndrejHome/) _ondrej-xa2iel8u@famera.cz_
* modified by [Michel Weitbrecht (SlothOfAnarchy)](https://github.com/SlothOfAnarchy) _michel.weitbrecht@stuvus.uni-stuttgart.de_
