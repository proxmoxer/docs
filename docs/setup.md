# Setup

Proxmoxer is easy to setup and has minimal dependencies.

## Choosing a Backend

The required dependencies change depending on the backend (connection method) you will use. The HTTPS backend better matches the details in the Proxmox API documentation, but SSH can be used in an (mostly) interchangeable manner. SSH can be used with complex network environments and can be used with jump hosts. The HTTPS backend can be used through a reverse proxy or any other HTTPS manipulation scheme.

For most users, the HTTPS backend will be suitable. The HTTPS backend allows connections for non-PAM realm accounts and supports advanced authentication methods such as API Tokens. It is also universally supported across all current Proxmox products. Unless SSH is specifically required, it is advised to use the HTTPS backend.

## Installing Dependencies

In addition to installing the `proxmoxer` package via pip, the following packages are required for each backend.

=== "https"

    `pip install requests`

    If you will be uploading files, installing `requests_toolbelt` will automatically allow larger upload file sizes and reduce memory footprint of an upload

=== "openssh"

    `pip install openssh_wrapper`

=== "ssh_paramiko"

    `pip install paramiko`

---

You can, of course, install all the dependencies with `pip install requests requests_toolbelt openssh_wrapper paramiko` to be able to use any of the backends.
