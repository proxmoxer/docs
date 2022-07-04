<!-- spell-checker:ignore Proxmoxia -->
# Welcome to Proxmoxer

Proxmoxer is a wrapper around the APIs for Proxmox products.

It was inspired by slumber, but it is dedicated only to Proxmox. It allows not only REST API use over HTTPS, but the same api over SSH. Like [Proxmoxia](https://github.com/baseblack/Proxmoxia), it dynamically creates attributes which responds to the attributes you've attempted to reach.

## Supported Services

Below are the Proxmox services supported by this library.[^1]

* PVE ([API Spec](https://pve.proxmox.com/pve-docs/api-viewer/index.html))
* PMG ([API Spec](https://pmg.proxmox.com/pmg-docs/api-viewer/index.html))
* PBS ([API Spec](https://pbs.proxmox.com/docs/api-viewer/index.html))

## Supported Backends (Connection Methods) {#supported-backends}

Below are the backends supported by this library.[^1]

* HTTPS
* SSH (openssh)
* SSH (ssh_paramiko)
* local

View the [Setup](setup/) page for details on how to setup your environment for each backend.

[^1]: The names of the services and backends can be in any case as the library will standardize the case to what it requires. e.g. "hTtpS" is just as valid as "https" or "HTTPS".

--8<-- "abbreviations.md"
