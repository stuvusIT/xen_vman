# Xen VM management (xen_vman)

This role manages Xen VMs with systemd services.

For each configured VM a Xen VM configuration is generated. You can start and stop the VMs with systemd.
Currently, iSCSI and NFS are completely supported as filesystem backends.
For all other backends you manually need to install the system afterwards.

For each installed kernel, a dracut initrd will automatically created at `/etc/xen/initrd.img-$VERSION` (similar to /boot initrds for dom0).
When starting a VM. `/etc/xen/initrd.img` will automatically be created as a symlink, pointing to the initrd of the currently running kernel.


## Requirements

- A Xen host machine running a Debian host system (you can use [xen-setup](https://github.com/stuvusIT/xen-setup) to perform this task)
- Filesystem utilities for filesystems used by the guests
- `python-netaddr` on the executing machine


## systemd services

### vm@&lt;organisation&gt;-&lt;host&gt;.service

Start and stop the given VM. For example `systemd start vm@stuvus-web01` will start the _web01_ server from the organisation _stuvus_.

### vm_iscsi@&lt;organisation&gt;-&lt;host&gt;.service

Does a login or a logout for the iSCSI root disk for the given VM, this is only done when iSCSI is selected as root backend, otherwise systemd will ignore any request silently.

### vm_mount@&lt;organisation&gt;-&lt;host&gt;.service

Mount or umount the root filesystem from the given VM. For example `systemd start vm_mount@stuvus-web02` mounts the root filesystem of _web01 @stuvus_ into `/mnt/vms/stuvus-web01`. The mount point is automated created and removed.


## nfs vms

All NFS exports need to be accessible for the hypervisor as well as the respective VM.
The exported path for the root filesystem must be of the following scheme:
`{{ xen_vman_nfsroot_base }}/<org name>/<vm name>-root` .


## Role Variables

| Name                                     | Type                           | Default/Required [¹](#__required)                                                                                                     | Description                                                                                                                           |
|------------------------------------------|--------------------------------|:-------------------------------------------------------------------------------------------------------------------------------------:|---------------------------------------------------------------------------------------------------------------------------------------|
| `xen_vman_xl_path`                       | string                         | **auto determined**                                                                                                                   | Fully qualified path of the xl command                                                                                                |
| `xen_vman_iscsiadm_path`                 | string                         | **auto determined**                                                                                                                   | Fully qualified path of the iscsiadm command                                                                                          |
| `xen_vman_vms`                           | [list of dicts](#vm-variables) | `[]`                                                                                                                                  | A list of all VM specifications, see [below](#vm-variables)                                                                           |
| `xen_vman_vm_config_path`                | string                         | `/etc/xen/vms`                                                                                                                        | Path to store the generated Xen VM configurations                                                                                     |
| `xen_vman_dracut_modules`                | list of strings                | `[nfs,network,base,bash,systemd,terminfo,shutdown,kernel-modules,kernel-network-modules,console-setup,systemd-initrd,dracut-systemd]` | Dracut modules to include in the Xen initrd                                                                                           |
| `xen_vman_dracut_extra_flags`            | string                         |                                                                                                                                       | Extra flags to pass to dracut when building the Xen initrds                                                                           |
| `xen_vman_start_timeout`                 | integer                        | `15`                                                                                                                                  | systemd timeout in seconds to start a VM                                                                                              |
| `xen_vman_stop_timeout`                  | integer                        | `300`                                                                                                                                 | systemd timeout in seconds to stop a VM, after timeout is reached, systemd kills the VM                                               |
| `xen_vman_iscsi_login_timeout`           | integer                        | `40`                                                                                                                                  | Maximum time in seconds to wait for an iSCSI login request to complete                                                                |
| `xen_vman_iscsi_logout_timeout`          | integer                        | `40`                                                                                                                                  | Maximum time in seconds to wait for an iSCSI logout request to complete                                                               |
| `xen_vman_mount_timeout`                 | integer                        | `20`                                                                                                                                  | Maximum time in seconds to wait for mount before systemd unit enters failed state                                                     |
| `xen_vman_umount_timeout`                | integer                        | `20`                                                                                                                                  | Maximum time in seconds to wait for umount before systemd unit enters failed state                                                    |
| `xen_vman_public_keys`                   | list of strings                | `[]`                                                                                                                                  | List of public keys to add to the newly generated user in the VM                                                                      |
| `xen_vman_nfsroot_base`                  | string                         | _required for NFS_                                                                                                                    | Path to prepend to all NFS paths                                                                                                      |
| `xen_vman_iscsi_server`                  | string                         | _required for iSCSI_                                                                                                                  | iSCSI server to connect to                                                                                                            |
| `xen_vman_iscsi_base_wwn`                | string                         | _required for iSCSI_                                                                                                                  | Base of the WWN name                                                                                                                  |
| `xen_vman_debootstrap_packages`          | list of strings                | `[systemd,systemd-sysv,udev,openssh-server,sudo,python,iproute2,ifupdown]`                                                            | Packages to install into a newly installed VM using debootstrap                                                                       |
| `xen_vman_default_memory`                | integer                        | `1024`                                                                                                                                | Memory to assign to a Xen domU in MiB [²](#xen_doc)                                                                                   |
| `xen_vman_default_vcpus`                 | integer                        | `2`                                                                                                                                   | Number of virtual CPU cores to assign to a VM [²](#xen_doc)                                                                           |
| `xen_vman_default_cpu_str`               | string                         | `all,^1`                                                                                                                              | List of which CPUs the guest is allowed to use [²](#xen_doc)                                                                          |
| `xen_vman_default_storage_type`          | string                         | `nfs`                                                                                                                                 | Default filesystem backend to use. Valid choices are `nfs` and `iscsi`                                                                |
| `xen_vman_default_disks`                 | list of strings                | `[]`                                                                                                                                  | Xen disk specification (see also [xl-disk-configuration.txt](http://xenbits.xen.org/docs/4.8-testing/misc/xl-disk-configuration.txt)) |
| `xen_vman_default_manual_root_disk`      | boolean                        | `False`                                                                                                                               | Use only disk specified by `xen_vman_default_disks` or `vm.disks`                                                                     |
| `xen_vman_default_filesystem`            | string                         | `ext4`                                                                                                                                | Filesystem to use for VMs with iSCSI root disk                                                                                        |
| `xen_vman_default_gateway`               | string                         | _required for a default route on newly installed vms_                                                                                 | Default gateway of the new VM                                                                                                         |
| `xen_vman_default_mtu`                   | integer                        |                                                                                                                                       | Default MTU for the new VM                                                                                                            |
| `xen_vman_default_nameserver`            | list of strings                | `[8.8.8.8, 8.8.4.4]`                                                                                                                  | Nameservers to initially set for the new VMs                                                                                          |
| `xen_vman_default_boot_after_creation`   | boolean                        | `True`                                                                                                                                | Start the VM after installation                                                                                                       |
| `xen_vman_default_auto_boot`             | boolean                        | `True`                                                                                                                                | Start VM when hypervisor boots                                                                                                        |
| `xen_vman_default_builder`               | string                         | `generic`                                                                                                                             | Select the domU guest type, valid values are `generic` (para virtualization) and `hvm` (all hardware is emulated)                     |
| `xen_vman_default_kernel`                | string                         | **auto determined**                                                                                                                   | Path to the kernel to use for generic VMs                                                                                             |
| `xen_vman_default_initrd`                | string                         | `/etc/xen/initrd.img`                                                                                                                 | Path to the initrd to use for generic VMs                                                                                             |
| `xen_vman_default_cmdline`               | string                         | _not required_                                                                                                                        | Command line to pass to the kernel, this option also disables auto generated kernel command lines                                     |
| `xen_vman_default_spice`                 | boolean                        | `False`                                                                                                                               | Allow access to the display via the SPICE protocol [²](#xen_doc)                                                                      |
| `xen_vman_default_spicehost`             | string (IP-Address)            | `0.0.0.0`                                                                                                                             | Specify the interface address to listen on if given, defaults to any interface [²](#xen_doc)                                          |
| `xen_vman_default_spiceport`             | integer                        | `3100`                                                                                                                                | Specify the port to listen on by the SPICE server if SPICE is enabled [²](#xen_doc)                                                   |
| `xen_vman_default_mouse`                 | boolean                        | `True`                                                                                                                                | Whether SPICE agent is used for client mouse mode [²](#xen_doc)                                                                       |
| `xen_vman_default_share_clipboard`       | boolean                        | `True`                                                                                                                                | Enables SPICE clipboard sharing [²](#xen_doc)                                                                                         |
| `xen_vman_default_spicevdagent`          | boolean                        | `True`                                                                                                                                | Enables SPICE vdagent, this is automatically enabled when clipboard sharing is enabled [²](#xen_doc)                                  |
| `xen_vman_default_max_usb_redirections`  | integer (0-4)                  | `4`                                                                                                                                   | Enable SPICE USB redirections [²](#xen_doc)                                                                                           |
| `xen_vman_default_additonal_xen_options` | xl.cfg key value dict          | `[]`                                                                                                                                  | Add additional Xen options                                                                                                            |
| `xen_vman_default_auto_install`          | boolean                        | `True`                                                                                                                                | Install specified OS automatically, when no xen configuration is found                                                                |
| `xen_vman_additional_network_scripts`    | list of strings                | `[]`                                                                                                                                  | List of [additional network scripts](#additional-network-scripts)                                                                     |
| `xen_vman_default_network_script`        | string                         | `vif-bridge`                                                                                                                          | Default xen-script used to setup vm network interfaces, for valid values see `ls /etc/xen/scripts/vif*` on the hypervisor             |
| `xen_vman_default_network_bridge`        | string                         | _required for some `xen_vman_default_network_bridge` settings_                                                                        | [Default network bridge](#default-network-bridge) to use for vm network interfaces                                                    |
| `xen_vman_lazy_default_network_bridge`   | string                         | _not required_                                                                                                                        | [Default network bridge](#default-network-bridge) (lazily evaluated)                                                                  |
| `xen_vman_default_xen_str`               | string                         | _not required_                                                                                                                        | Default for [`xen_str`](#vm-interfaces)                                                                                               |
| `xen_vman_default_vifname_prefix`        | string                         | _not required_                                                                                                                        | Default for [`vifname`](#vm-interfaces)                                                                                               |


<a id="__required">¹</a> Variable is not required unless no default is given or other specified
<a id="xen_doc">²</a> For further reference see [man xl.cfg](http://xenbits.xen.org/docs/4.8-testing/man/xl.cfg.5.html)

### Additional network scripts

This role currently supports a custom network script called `vif-p2p`.
It is like (the official) `vif-route`, except that it configures the vif
interface using a P2P configuration from dom0 to the guest.
That is, if `vif-route` configures the interface like so:

```bash
ip address add 10.0.0.1/32 dev vif10.0
ip route add 10.0.0.5/32 dev vif10.0 src 10.0.0.1
```

Then `vif-p2p` configures the interface like so instead:

```bash
ip address add 10.0.0.1/32 peer 10.0.0.5 dev vif10.0
```

This way, the kernel will automatically add the necessary route.

In order to install `vif-p2p` you have to add the string `"vif-p2p"` to
`xen_vman_additional_network_scripts`.

#### Custom additional network scripts

In order to install your own network script (that is not supplied by this role),
you can add arbitrary paths to  `xen_vman_additional_network_scripts`.
Relative paths are relative to your `files` folder.
Only the file name is preserved:
For example, adding `"xen_vman/vif-custom"` to
`xen_vman_additional_network_scripts` will instruct this role to copy
`files/xen_vman/vif-custom` (on localhost) to `/etc/xen/scripts/vif-custom`
(on the remote host).

For doing so, it is recommended to use a subfolder like `xen_vman`, as in the
previous example.
Otherwise your file name might collide with a supplied network script in a
future version of this role.
In the case of such a collision, the script supplied by this role takes
precedence.

### Default network bridge

`xen_vman_default_network_bridge` is written to Xen's config file, while
`xen_vman_lazy_default_network_bridge` is lazily evaluated in order to default the `bridge` of
[VM interfaces](#vm-interfaces).
Therefore you can access the [VM variables](#vm-variables) from within
`xen_vman_lazy_default_network_bridge`.

### VM variables

These variables can be specified for each VM.
VMs are defined as elements of the `xen_vman_vms` list.

| Name                     | Type                            | Required | Description                                                                             |
|--------------------------|---------------------------------|:--------:|-----------------------------------------------------------------------------------------|
| `org`                    | string                          |     Y    | Organization which owns this VM                                                         |
| `name`                   | string                          |     Y    | Name of this VM                                                                         |
| `memory`                 | integer                         |     N    | Amount of memory to assign to this VM. The number is treated as amount of megabytes     |
| `vcpus`                  | integer                         |     N    | Amount of virtual CPU cores                                                             |
| `cpu_str`                | string                          |     N    | List of which CPU cores the VM is allowed to use. See above for more information        |
| `interfaces`             | [list of dicts](#vm-interfaces) |     Y    | Virtual network interfaces. See [below](#vm-interfaces)                                 |
| `connection_ip`          | string                          |     N    | IP of this VM. Will be configured as static IP. First interface IP is used when omitted |
| `storage_type`           | string                          |     N    | Storage backend of the root filesystem. See above for valid options                     |
| `disks`                  | list of strings                 |     N    | Additional Xen disk specifications. See above for more information                      |
| `manual_root_disk`       | boolean                         |     N    | Use only disk specified by `xen_vman_default_disks` or `vm.disk`                        |
| `filesystem`             | string                          |     N    | When using iSCSI, this filesystem is used for the root                                  |
| `os.type`                | string                          |     Y    | Type of the operating system                                                            |
| `os.version`             | string                          |     Y    | Version of the chosen operating system                                                  |
| `boot_after_creation`    | boolean                         |     N    | Whether to boot the VM after creation                                                   |
| `auto_boot`              | boolean                         |     N    | Whether to boot the VM on hypervisor boot                                               |
| `builder`                | string                          |     N    | Select the builder for this VM (`generic` or `hvm`)                                     |
| `kernel`                 | string                          |     N    | (For generic VMs only) Path to the kernel to boot                                       |
| `initrd`                 | string                          |     N    | (For generic VMs only) Path to the initrd to pass to the kernel                         |
| `cmdline`                | string                          |     N    | (For generic VMs only) Command line to pass to the kernel                               |
| `spice`                  | boolean                         |     N    | (For generic VMs only) Enable SPICE                                                     |
| `spicehost`              | string                          |     N    | (For HVM VMs only) Address for the SPICE server to listen on                            |
| `spiceport`              | integer                         |     N    | (For HVM VMs only) Port for the SPICE server to listen on                               |
| `spicepasswd`            | string                          |     N    | (For HVM VMs only) Password required when connecting to the SPICE server                |
| `share_clipboard`        | boolean                         |     N    | (For HVM VMs only) Share clipboard via SPICE                                            |
| `spicevdagent`           | boolean                         |     N    | (For HVM VMs only) Enable the SPICE vdagent                                             |
| `max_usb_redirections`   | integer                         |     N    | (For HVM VMs only) Maximum amount of USB redirections                                   |
| `additional_xen_options` | xl.cfg key value dict           |     N    | Add additional Xen options[²](#xen_doc)                                                 |
| `auto_install`           | boolean                         |     N    | Install specified OS automatically, when no Xen configuration is found                  |
| `reinstall`              | boolean                         |     N    | Reinstall the VM every time Ansible is running when set to `True`                       |


### VM interfaces

Each network interface may have the following options.
All of them are optional.

| Name      | Type      | Description                                  |
|-----------|-----------|----------------------------------------------|
| `ip`      | `string`  | IP address of this interface                 |
| `mac`     | `string`  | MAC address of this interface                |
| `mtu`     | `integer` | MTU of this interface                        |
| `bridge`  | `string`  | Bridge to attach the interface to            |
| `vifname` | `string`  | Name of the vif on dom0                      |
| `xen_str` | `string`  | Additional Xen parameters for this interface |

Note that if you use `xen_vman_default_vifname_prefix` (and don't overwrite
it using the `vifname` option), then `.DEVID` is appended to the vif name,
whereby `DEVID` is the index of the interface.
(Hence the name `..._prefix`.)
For example, when using `xen_vman_default_vifname_prefix: "example"`, then
the vifs are named `example.0`, `example.1` and so on.

## Example Playbook

```yml
- hosts: hypervisor
  roles:
    - xen_vman
      xen_vman_vms:
        - name: testvm
          interfaces:
          - ip: 10.0.0.15
            mac: 00:14:22:01:23:45
            bridge: mybridge
            os:
              type: ubuntu
              version: 18.04
```


## License

This work is licensed under a [Creative Commons Attribution-ShareAlike 4.0 International License](http://creativecommons.org/licenses/by-sa/4.0/).


## Author Information

- [Markus Mroch (Mr. Pi)](https://github.com/Mr-Pi) _markus.mroch@stuvus.uni-stuttgart.de_
