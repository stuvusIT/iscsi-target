# iscsi-target

iSCSI target role that installs and uses targetcli.

## Requirements

An apt- or yum-based package manager and systemd. 
Also, this role is not creating any disks/partitions/LVs. Therefore it is expected that they are already present on machine or created by some other role.

## Role Variables

| Name                  | mandatory  | Description                                                                                  |
|-----------------------|------------|----------------------------------------------------------------------------------------------|
| `iscsi_base_wwn`              | yes  | WWN that identifies the target host |
| `iscsi_disk_path_prefix`       | no | Optional prefix to exported disk paths                                    |
| `iscsi_targets`     | yes | List of targets to be configured, see [targets](#targets)                                              |


### Targets

Each target has the following vars:

| Name                  | mandatory | Description                                                                          |
|-----------------------|--------|-----------------------------------------------------------------------------------------|
| `name`                | yes |`name` identifier of this target. This is appended to `iscsi_base_wwn` and used as `wwn`    |
| `disks`               | yes | Disk configuration, see [disks](#disks)                                                    |
| `initiators`          | yes | List of `initiators` that are allowed to connect to this target, see [initiators](#initiators)   |
| `portals`             | yes | List of dicts that contains the local `ip` and optionally the `port` on which access to this target is allowed (default port is `3260`)                                        |
| `state`               | no  | `present` or `absent`. Default: `present`                                                  |


### Disks

A list of dicts with the following mandatory entries:

| Name                  | Description                                                                                             |
|-----------------------|---------------------------------------------------------------------------------------------------------|
| `name`                | Name that is used for the backstore                                                                     |
| `path`                | Existing path to the disk that should be used as backstore. `iscsi_disk_path_prefix` will be prepended if defined. |
| `type`                | One out of {`fileio`, `iblock`, `pscsi`, `rd_mcp`}                                                      |

### Initiators

`initiators` is a lists of dicts that must have a `wwn` attribute and can optionally have a `authentication` attribute, which is also a dict that can contain the following entries:

| Name                  | mandatory  | Description                                                                                  |
|-----------------------|------------|----------------------------------------------------------------------------------------------|
| `userid`              | yes  | Userid to authenticate the initator                                                                |
| `password`            | yes | Password to authenticate the initiator                                                              |
| `userid_mutual`       | no | Mutual userid to authenticate the target                                                             |
| `password_mutual`     | no | Mutual password to authenticate the target                                                           |



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
            - name: iqn.1994-05.com.redhat:client1
              authentication:
                userid: myuser
                password: mypassword
                userid_mutual: sharedkey
                password_mutual: sharedsecret
            - name: iqn.1994-05.com.redhat:client2
              authentication:
                userid: otheruser
                password: otherpw
                userid_mutual: somekey
                password_mutual: somesecret
          portals:
            - ip: 192.168.1.45
              port: 5555
            - ip: 192.168.2.10
```

This example provides two devices as disks and allows two initiators to connect to them using different authentication credentials.
The targets provides this on `192.168.1.45:5555` and `192.168.2.10:3260`


## License

GPLv3

## Author Information
* original author: [Ondrej Faměra (OndrejHome)](https://github.com/OndrejHome/) _ondrej-xa2iel8u@famera.cz_
* modified by [Michel Weitbrecht (SlothOfAnarchy)](https://github.com/SlothOfAnarchy) _michel.weitbrecht@stuvus.uni-stuttgart.de_
