vm-revert
=========

Rollsback and reboots a KVM/libvirt virtual machine to a previously created
snapshot.


Requirements
------------

This role requires root access to the server which acts as the host to the
inventory host. It also uses the raw `virsh` commands provided by
`libvirt-client` rather than the API as is done by `community.libvirt`, since at
times, it is undesirable to make the API accessable across the network.

Because of what it does, it is intended to be used as entry in a `pre_task:`
block of a playbook, like this:

```yaml
---
- hosts: a-vm-guest
  pre_tasks:
    - name: Reboot and roll back a-vm-guest to snapshot1
      include_role:
        name: vm-revert
	tasks_from: pre-tasks
      vars:
        vm_host: a-vm-server
```

Be sure to read the role variables carefully! You have been warned!!!


Role Variables
--------------

While rather limited, the available variables are listed below, along with
default values (see `defaults/main.yml`).

```yaml
vm_revert: false
```

Whether or not the VM guest should be rebooted and reverted to a snapshot
(rolled back). This is defaulted to false for a reason as a protection. Set to
true only if you know what you are doing.

[!CAUTION]
Do not enable this setting unless you want a virtual machine to reboot and
reverted back to a previously created snapshot. Because of the nature of this
variable, it ***should never be placed in a `group_vars` file***. Indeed, it is
strongly advised that it not be put in a `host_vars` file or playbook either,
especially one specifying a group of hosts. Instead, it should be used like this
example:

and ideally, run using a command line such as the following:

```bash
ansible-playbook --extra-vars '{"vm_revert": true}' --limit a-vm-guest a-playbook.yml
```

The JSON format is necessary to get the value to be an actual boolean rather
than a string, which while the string `"true"` is considered truthy and
evaluates to `True` implicitly by Python and Jinja2, its use is highly
discouraged.


```yaml
vm_host: undef()
```

This is the server acting as the host server for the virtual machine. Keeping in
mind the warning about the `vm_revert` variable, a perfect place for this and
other variables ***except for `vm_revert`*** is in the host_vars file for the
inventory item. Commands are delegated to this host for execution. The actual
default for this does supply a hint, but is not shown here for clarity.


```yaml
vm_revert_snapshot: "snapshot1"
```

This is the name of the snapshot which will be used in the `virsh
snapshot-revert` command, which is in fact the default name the first time you
create a snapshot using `virt-manager`.

```yaml
vm_wait_delay: 10
```

How long to delay in seconds before starting to check to see if the machine has
restarted. Unlikely to be needed, but provided for ease of overriding should it
ever become necessary.

```yaml
vm_wait_timeout: 90
```

How long to keep trying in seconds after the delay before we consider the
restart to have failed. Unlikely to be needed, but provided for ease of
overriding should it ever become necessary.


```yaml
vm_regather_facts: true
```

Whether or not to regather facts after the restart. Default is `true`, since if
we revert to a snapshot, our previously gathered facts are now out-of-date.  It
is unlikely that you will ever want to override this, but provided for ease just
in case.


Whether or not to regather facts after the restart. Default is `true`, since
if we revert to a snapshot, our previously gathered facts are now out-of-date.
It is unlikely that you will ever want to override this, but provided for ease
just in case.
vm_regather_facts: true


```yaml
vm_debug_variables: true
```

Whether or not to do a debug display of the variables after re-gathering
them. This combines with a verbosity of 1 for just the host name to indicate
that they have been regathered, and 3 for a more complete display.


Dependencies
------------

This role is dependent upon the [community.libvirt.virt
module](https://docs.ansible.com/projects/ansible/latest/collections/community/libvirt/virt_module.html)
to execute many of the commands associated with controlling the virtual
machine. As such, it shares the following requirements on the system hosting the
virtual machine.

- python >= 3.9
- libvirt python bindings (libvirt-python>=10.10.0, e.g. python3-libvirt on RHEL 9 systems)
- lxml >= 4.6.5 (e.g. python3-lxml on RHEL 9 systems)

Example Playbook
----------------


Here is an example playbook. Please note the lack of vm_revert in the variables.

```yaml
---
- hosts: a-vm-guest
  pre_tasks:
    - name: Reboot and roll back a-vm-guest to snapshot1
      include_role:
        name: vm-revert
	tasks_from: pre-tasks
      vars:
        vm_host: a-vm-server
```

To properly run a playlist using this role, it is highly recommended to use the
following command, explicitly set `vm_revert` on the command line, and limit the
execution to a single machine, using a command such as the following:

```bash
ansible-playbook --extra-vars '{"vm_revert": true}' --limit a-vm-guest a-playbook.yml
```


License
-------

MIT/BSD

Author Information
------------------

This role was originally created in 2024 by [Douglas Wade
Needham](https://www.ka8zrt.com) and first publicly published in May 2026.
