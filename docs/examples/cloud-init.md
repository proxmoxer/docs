<!-- spell-checker:ignore nocloud Cloudbase configdrive ciuser cipassword ciupgrade cicustom sshkeys nameserver searchdomain nextid -->
<!-- spell-checker:ignore newid urlencoding urllib urlparse userdata -->
# Configuring new VMs with Cloud-init

## What does Cloud-init do?

[Cloud-init](https://cloudinit.readthedocs.io/en/latest/) is the industry
standard for customizing guest operating systems on virtual
infrastructure platforms such as Proxmox PVE.

This customization stage occurs during the first boot of the guest operating
system, where an initialization agent (such as cloud-init) looks for a so-called
"cloud-config" data source (usually
[just a YAML file](https://cloudinit.readthedocs.io/en/latest/reference/examples.html)
transparently served from a reserved IP class or locally attached ISO image),
and applies the various settings specified within said cloud config (e.g. creating a new user).

## Cloud-init settings on Proxmox PVE

The Proxmox PVE API allows users to set specific properties within the cloud config
to be used during the initial run of Cloud-init within the guest OSes on first boot,
such as:

* Specifying a new username/password/SSH keys which will get automatically
  created.
* Specifying IPv4/6 configurations to be applied within the guest OS.
* Some additional options, such as performing automatic package manager updates
  on Linux guests, static networking settings, and more.

## Template Prerequisites

In order for the Cloud-init instance options to be applied, the VM image being
booted must have a pre-installed agent which is able to read the the
[cloud config data format](https://cloudinit.readthedocs.io/en/latest/reference/examples.html).

### Linux templates

On Linux guests, the so-called [`nocloud` data source](https://cloudinit.readthedocs.io/en/latest/reference/datasources/nocloud.html)
is used to pass the cloud config to the guest OS, so the Cloud-init
installation within the VM image being booted must be configured to use it.

In most situations, using pre-baked Linux "golden images" (such as the upstream
[Canonical Ubuntu Server images](https://cloud-images.ubuntu.com)) should work
out of the box, though some additional pre-customization (such as installing the
`qemu-guest-agent`) might be required for better compatibility with PVE.

### Windows templates

For Windows guests, the industry-standard initialization agent implementation is
[Cloudbase-init](https://github.com/cloudbase/cloudbase-init).

On Windows guests, the so-called [`configdrive2` data source](https://cloudinit.readthedocs.io/en/latest/reference/datasources/configdrive.html#version-2)
is used to pass the cloud config to the guest OS, so the Windows image must
have Cloudbase-init configured to use it.

!!! warning "Issues on Windows"
    as of the time of writing, there are some known issues with
    the formatting of some fields of the cloud-config for Windows guests (most
    notably, the [inability set passwords through the cloud-config](https://bugzilla.proxmox.com/show_bug.cgi?id=4493)).

## Example code setting cloud-init options

The current PVE API allows for the transparent configuration of a number of
guest initialization options via the [VM's `config` properties](https://pve.proxmox.com/pve-docs/api-viewer/index.html#/nodes/{node}/qemu/{vmid}/config).

* `ciuser` (str): new username to create on first boot.
* `cipassword` (str): new password for the above `ciuser`.
* `sshkeys` (str): public SSH keys to be automatically added as authorized for
  the above `ciuser`.
* `ciupgrade` (bool): whether or not to automatically update packages on first
  boot.  
  **NOTE:** the default is `True`, and it may negatively impact applications
  expecting fixed dependency versions.
* `cicustom` (str): allows for specifying the entirety of the Cloud-init config file.
* `ipconfigN` (str): specifies IP settings for the guests network interface
  with index `N` (e.g. `ip=192.168.125.25/32,gw=192.168.0.1`). Omitting this will
  have cloud-init attempt DHCPv4 on said interface.
* `nameserver` (str): list of DNS nameservers to configure within the guest.
* `searchdomain` (str): DNS search domain to use. Omitting this will have the
  VM inherit the hosting Proxmox node's search domain. (if any)

```python
# NOTE: please remember that the template must have Cloud-init (or equivalent)
# pre-installed and pre-configured for first boot. (see above template sections)
TEMPLATE_ID = 200

clone_vm_id = int(proxmox.cluster.nextid.get())
clone_task = proxmox.nodes("<node_name>").qemu(TEMPLATE_ID).clone.create(newid=clone_vm_id)

# NOTE: should either wait a fixed time for the clone to complete, or check the status of the clone task
#time.sleep(10)
from proxmoxer.tools import Tasks
Tasks.blocking_status(proxmox, clone_task)

# NOTE: the `sshkeys` attribute requires manual urlencoding on our end.
# Please see https://github.com/proxmoxer/proxmoxer/issues/153.
from urllib import parse as urlparse

SSH_KEYS = "ssh-rsa ..."  # contents of '~/ssh/id_rsa.pub'
# NOTE: by default safe='/' so we have to pass safe='' to clear it.
ENCODED_KEYS = urlparse.quote(SSH_KEYS, safe='')

# Set the userdata as desired BEFORE starting the machine:
proxmox.nodes("<node_name>").qemu(clone_vm_id).config.set(
    ciuser="<new_username>",
    cipassword="<new_password>",
    sshkeys=ENCODED_KEYS,
    ciupgrade=False,
    cicustom="<explicit YAML cloud-config>",
    ipconfig0="ip=192.168.125.25/32,gw=192.168.0.1",
    nameserver="1.1.1.1",
    searchdomain="example.com",
)

# Start the machine normally, and give cloud-init a minute for its initial run.
# You should be able to see cloud-init's logs within the VM console in the PVE UI.
proxmox.nodes("<node_name>").qemu(clone_vm_id).status.start()
```
