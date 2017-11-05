# Xen vm management (xen_vman)

This role managed xen vms with systemd services.

For each configured vm a xen vm configuration will be auto generated. You can start and stop the vms with systemd.
Currently iscsi and nfs are completely supported as filesystem backend. For all other backends you need to install the system afterwards manually.

## systemd services

### vm@&lt;organisation&gt;-&lt;host&gt;.service
Start and stop the given VM. For ex. `systemd start vm@stuvus-web01` will start the _web01_ server from the organisation _stuvus_.

### vm_iscsi@&lt;organisation&gt;-&lt;host&gt;.service
Does a login or a logout for the iSCSI root disk for the given VM, this is only done, when as backend iSCSI is selected, otherwise systemd will ignore any request silently.

### vm_mount@&lt;organisation&gt;-&lt;host&gt;.service
Mount or umount the root filesystem from the given VM. For ex. `systemd start vm_mount@stuvus-web02` mounts the root fs from _web01 @stuvus_ into `/mnt/vms/stuvus-web01`. The mount point is automated created and removed.

## Requirements

A xen host machine running a debian host system. And filesystem utils for all used filesystems installed.

## Role Variables

### Primary
| Option                                 | Type                    | Default           | Description                                                                                                                           | Required |
|----------------------------------------|-------------------------|-------------------|---------------------------------------------------------------------------------------------------------------------------------------|:--------:|
| xen_vman_xl_path                       | string                  | _auto determined_ | Fully qualified path for the xl command                                                                                               |     N    |
| xen_vman_iscsiadm_path                 | string                  | _auto determined_ | Fully qualified path for the iscsiadm                                                                                                 |     N    |
| xen_vman_vms                           | list of dicts           | `[]`              | A list of all vm specification see [xen_vman_vms](#xen_vman_vms)                                                                      |     N    |
| xen_vman_vm_config_path                | string                  | `/etc/xen/vms`    | Path to store the generate xen vm configuration                                                                                       |     N    |
| xen_vman_start_timeout                 | integer                 | `15`              | Systemd timeout in seconds to start a VM                                                                                              |     N    |
| xen_vman_stop_timeout                  | integer                 | `300`             | Systemd timeout in seconds to shutdown a VM, after timeout is reached, systemd kills the VM                                           |     N    |
| xen_vman_iscsi_login_timeout           | integer                 | `40`              | Maximum time in seconds to wait for an iSCSI login request to complete                                                                |     N    |
| xen_vman_iscsi_logout_timeout          | integer                 | `40`              | Maximum time in seconds to wait for an iSCSI logout request to complete                                                               |     N    |
| xen_vman_mount_timeout                 | integer                 | `20`              | Maximum time in seconds to wait for mount before systemd unit enters failed state                                                     |     N    |
| xen_vman_umount_timeout                | integer                 | `20`              | Maximum time in seconds to wait for umount before systemd unit enters failed state                                                    |     N    |
| xen_vman_default_memory                | integer                 | `1024`            | Memory to assign to a xen domU in MiB [¹](#xen_doc)                                                                                   |     N    |
| xen_vman_default_vcpus                 | integer                 | `2`               | Number of virtual CPU cores to assign to a VM [¹](#xen_doc)                                                                           |     N    |
| xen_vman_default_storage_type          | string                  | `nfs`             | Default filesystem backend to use                                                                                                     |     N    |
| xen_vman_default_cpu_str               | string                  | `all,^1`          | List of which cpus the guest is allowed to use [¹](#xen_doc)                                                                          |     N    |
| xen_vman_default_spice                 | `0` or `1`              | `0`               | Allow access to the display via the SPICE protocol [¹](#xen_doc)                                                                      |     N    |
| xen_vman_default_spicehost             | string(IP-Address)      | `0.0.0.0`         | Specify the interface address to listen on if given, otherwise any interface [¹](#xen_doc)                                            |     N    |
| xen_vman_default_spiceport             | integer(Port number)    | `3100`            | Specify the port to listen on by the SPICE server if the SPICE is enabled [¹](#xen_doc)                                               |     N    |
| xen_vman_default_mouse                 | `0` or `1`              | `1`               | Whether SPICE agent is used for client mouse mode [¹](#xen_doc)                                                                       |     N    |
| xen_vman_default_spicevdagent          | `0` or `1`              | `1`               | Enables spice vdagent, is automatically enabled when clipboard sharing is enabled [¹](#xen_doc)                                       |     N    |
| xen_vman_default_share_clipboard       | `0` or `1`              | `1`               | Enables Spice clipboard sharing                                                                                                       |     N    |
| xen_vman_default_max_usb_redirections  | integer(0-4)            | `4`               | Enables spice usbredirection                                                                                                          |     N    |
| xen_vman_default_disks                 | list of strings         | `[]`              | Xen disc specification (see also [xl-disk-configuration.txt](http://xenbits.xen.org/docs/4.8-testing/misc/xl-disk-configuration.txt)) |     N    |
| xen_vman_default_builder               | string                  | `generic`         | Select the domU guest type, valid values are `generic`(para virtualisation) and `hvm`(all hardware is emulated)                       |     N    |
| xen_vman_default_additonal_xen_options | list of key value dicts | `[]`              | Add additional xen options                                                                                                            |     N    |
| xen_vman_default_filesystem            | string                  | `ext4`            | Filesystem to use for VMs with iSCSI root disk                                                                                        |     N    |
| xen_vman_default_auto_boot             | boolean                 | `True`            | Start VM when system boots                                                                                                            |     N    |
| xen_vman_default_boot_after_creation   | boolean                 | `True`            | Starts the vm after installation                                                                                                      |     N    |

<a id="xen_doc">¹</a> For future reference see [man xl.cfg](http://xenbits.xen.org/docs/4.8-testing/man/xl.cfg.5.html)


## Example Playbook
### Vars:
```yml
```
### Result:

## License

<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-sa/4.0/80x15.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">Creative Commons Attribution-ShareAlike 4.0 International License</a>.

## Author Information
- [Markus Mroch (Mr. Pi)](https://github.com/Mr-Pi) _markus.mroch@stuvus.uni-stuttgart.de_
