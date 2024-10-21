# Authentication

To access a Proxmox service, you must first authenticate with the API. The method changes based on the chosen backend, user type, and credential type.

## Proxmox API Permission Scopes

To interact with a Proxmox service, the authentication credentials used must have the appropriate permissions.
For more information, view the Proxmox documentation on permissions (e.g. [PVE Permissions](https://pve.proxmox.com/wiki/User_Management#pveum_permission_management)).

## HTTPS Backend

The HTTPS is the universally accepted backend and supports the most authentication options. To ensure you are only sending your credentials encrypted and to the correct server, make sure to setup a valid SSL certificate and remove `verify_ssl=False` from your initialization of the ProxmoxAPI object.

### Username and Password

!!! info "Supported Services: PVE, PMG, PBS"

Password authentication is the default authentication method and what is used by the web UI. To use this authentication method, the following data is needed:

* username
* realm (`pam`, `pve`, etc.)
* password

For the `pve` realm, the password is set within the API/webUI. For other realms, the password is set in those tools (e.g. for the `pam` realm, the password is found in `/etc/passwd` (or really `/etc/shadow`) and can be set by running `passwd <username>` as root).
If your user has a One Time Password (OTP) enabled, you will need to add the `otp` parameter and with your current OTP code to be able to authenticate. This is only required when first authenticating, as long as the requirements for authentication renewal are met (see below).

```python
prox = ProxmoxAPI('<host_ip_or_domain>', user='<username>@<realm>', password='<password>', otp='<otp_code>', service='<proxmox_service>', verify_ssl=<True|False>, timeout=<timeout_in_seconds>)
```

For example:

```python
prox = ProxmoxAPI('10.10.10.10', user='root@pam', password='password', verify_ssl=False)

```

#### Renewing Authentication

The authentication ticket retrieved using your username/password pair expires after 2 hours. To allow longer usage of proxmoxer, when a request is made close to the expiry of the ticket, the ticket is automatically renewed for another 2 hours. No user interaction is required for renewal and it will automatically be triggered by any request.

If long-running usage of proxmoxer is required, the only requirement is to make a request within 2 hours of the most recent request. If the application will already make requests more regularly than 2 hours, no extra requests are needed to keep authentication valid. Any request will trigger an authentication renewal, so a basic `GET` (e.g. [getting version](https://pve.proxmox.com/pve-docs/api-viewer/index.html#/version)) every hour or so is enough to keep authentication current. For an alternative, see [API Tokens](#api-token) below for a stateless authentication method which allows fine-grain control over permissions and allows authentication expiry.

Renewal also does not require OTP codes, so once initially authenticated with an OTP, proxmoxer can continue without user interaction.

### API Token

!!! info "Supported Services: PVE, PBS"

The API Token allows stateless interaction with the Proxmox service as well as independent permissions and more flexibility in credential lifecycle management. Abused/leaked API Tokens can be disabled independent of the account with which they are associated and multiple API Tokens can be created for a user, allowing each use its own API Token with only its needed permissions which can be independently managed.

```python
prox = ProxmoxAPI('<host_ip_or_domain>', user='<username>@<realm>', token_name='<token_name>', token_value='<token_value>', service='<proxmox_service>', verify_ssl=<True|False>, timeout=<timeout_in_seconds>)
```

For example:

```python
prox = ProxmoxAPI('10.10.10.10', user='my_user@pam', token_name='testToken', token_value='41c97f11-b8c6-47db-9886-7fa841e64b6e', verify_ssl=False)
```

### OIDC credentials provided by proxmox-oidc-credential-helper

First install binary for [proxmox-oidc-credential-helper](https://github.com/camaeel/proxmox-oidc-credential-helper/) in $PATH. This small utility opens browser and uses OIDC authentication in proxmox UI to obtain Ticket. This ticket can be later used as password for proxmoxer. 

Example code:
```python
#!/usr/bin/env python3

from proxmoxer import ProxmoxAPI
import json
import subprocess

proxmox_host = 'proxmox.example.com'
proxmox_port = 8006
realm = "realm1"

creds_helper_output = subprocess.run(f"proxmox-oidc-credential-helper -proxmox-url https://{proxmox_host}:{proxmox_port} -realm {realm} -output=json", shell=True, capture_output=True)
creds_helper_json = json.loads(creds_helper_output.stdout)

pve = ProxmoxAPI(proxmox_host, user = creds_helper_json['data']['username'], password = creds_helper_json['data']['ticket'], verify_ssl=True)
print(pve.nodes.get())
```

## OpenSSH Backend

!!! info "Supported Services: PVE, PMG"

The OpenSSH backend uses OpenSSH to remote into the Proxmox service and run the command using the service's CLI interface. This backend will default to using the config and key of the ssh client (usually in `~/.ssh/`), so often only needs a host and a user. As this uses SSH to log into the service, it can be considered to use the "pam" realm and the specified user must have permission to connect and run the appropriate commands. If the user does not natively have permission to run the required commands, adding an argument of `sudo=True` will prepend `sudo ` to the command so the specified user can elevate privileges.

### Minimal Required Arguments

```python
prox = ProxmoxAPI('<host_ip_or_domain>', user='<username>', backend='openssh')
```

### All OpenSSH Arguments

```python
prox = ProxmoxAPI('<host_ip_or_domain>', user='<username>', port=<port_number>, sudo=<True|False>, forward_ssh_agent=<True|False>, config_file='<path_to_config_file>', identity_file='<path_to_identity_file>', timeout=<timeout_in_seconds>, backend='openssh')
```

For example:

```python
prox = ProxmoxAPI('10.10.10.10', user='my_user', sudo=True, backend='openssh')
```

## Paramiko

!!! info "Supported Services: PVE, PMG"

Paramiko is an implementation of SSH completely in Python. This reduces some of the available functionality, but increases portability and reduces dependencies on outside programs.
This backend supports password and key authentication.

Paramiko uses the following order to attempt authentication:[^1]

* private_key_file
* SSH Agent
* Auto-discovered key files
* Password

### Key

Paramiko defaults to looking for in `~/.ssh` for openssh-style key file names. If a `private_key_file` argument is present, it is used before any keys automatically found.

```python
prox = ProxmoxAPI('<host_ip_or_domain>', user='<username>', private_key_file='<path_to_file>', backend='ssh_paramiko')
```

### Password

If a password is provided, usage of an SSH agent is disabled.

```python
prox = ProxmoxAPI('<host_ip_or_domain>', user='<username>', password='<password>', backend='ssh_paramiko')
```

### All Paramiko Arguments

```python
prox = ProxmoxAPI('<host_ip_or_domain>', user='<username>', password='<password>', port=<port>, private_key_file='<path_to_file>', timeout=<timeout_in_seconds>, sudo=<True|False>, backend='ssh_paramiko')
```

[^1]: [https://docs.paramiko.org/en/stable/api/client.html#paramiko.client.SSHClient.connect](https://docs.paramiko.org/en/stable/api/client.html#paramiko.client.SSHClient.connect)

## Local

The local backend requires no authentication, other than optionally utilizing sudo.

```python
prox = ProxmoxAPI(sudo=<True|False>, timeout=<timeout_in_seconds>, backend='local')
```
