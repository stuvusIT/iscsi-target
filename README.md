# iscsi-target

iSCSI target role that installs and uses targetcli.

## Requirements

An apt- or yum-based package manager and systemd. 
Also, this role is not creating any disks/partitions/LVs. Therefore it is expected that they are already present on machine or created by some other role.

## Role Variables

| Name                     | Default / Mandatory | Description                                               |
|:-------------------------|:-------------------:|:----------------------------------------------------------|
| `iscsi_base_wwn`         | :heavy_check_mark:  | WWN that identifies the target host                       |
| `iscsi_disk_path_prefix` |        `''`         | Optional prefix to exported disk paths                    |
| `iscsi_targets`          | :heavy_check_mark:  | List of targets to be configured, see [targets](#targets) |


### Targets

Each target has the following vars:

| Name         | Default / Mandatory | Description                                                                                                                             |
|:-------------|:-------------------:|:----------------------------------------------------------------------------------------------------------------------------------------|
| `name`       | :heavy_check_mark:  | `name` identifier of this target. This is appended to `iscsi_base_wwn` and used as `wwn`                                                |
| `disks`      | :heavy_check_mark:  | Disk configuration, see [disks](#disks)                                                                                                 |
| `initiators` | :heavy_check_mark:  | List of `initiators` that are allowed to connect to this target, see [initiators](#initiators)                                          |
| `portals`    | :heavy_check_mark:  | List of dicts that contains the local `ip` and optionally the `port` on which access to this target is allowed (default port is `3260`) |
| `state`      |      `present`      | `present` or `absent`. Default: `present`                                                                                               |


### Disks

A list of dicts with the following entries:

| Name   | Default / Mandatory | Description                                                                                                        |
|:-------|:-------------------:|:-------------------------------------------------------------------------------------------------------------------|
| `name` | :heavy_check_mark:  | Name that is used for the backstore                                                                                |
| `path` | :heavy_check_mark:  | Existing path to the disk that should be used as backstore. `iscsi_disk_path_prefix` will be prepended if defined. |
| `type` | :heavy_check_mark:  | One out of {`fileio`, `iblock`, `pscsi`, `rd_mcp`}                                                                 |

### Initiators

`initiators` is a lists of dicts that must have a `wwn` attribute and can optionally have a `authentication` attribute, which is also a dict that can contain the following entries:

| Name              |     mandatory      | Description                                |
|:------------------|:------------------:|:-------------------------------------------|
| `userid`          | :heavy_check_mark: | Userid to authenticate the initator        |
| `password`        | :heavy_check_mark: | Password to authenticate the initiator     |
| `userid_mutual`   |     (omitted)      | Mutual userid to authenticate the target   |
| `password_mutual` |     (omitted)      | Mutual password to authenticate the target |



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
