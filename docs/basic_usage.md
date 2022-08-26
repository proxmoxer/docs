<!-- spell-checker:ignore userid vmid pythonic configurability exitcode diskread diskwrite cpus -->
<!-- spell-checker:ignore netin netout maxswap maxmem maxdisk -->
# Basic Usage

Proxmoxer is easy to use. After importing the library and creating an instance, interacting with the Proxmox service is simple and follows the API documentation.

## Importing and Creating an Instance

??? info "Example Data"
    In the examples below (and in the rest of the documentation), placeholders are used for values which will be different between installations. Anything within `<>` is a value you will need to change for your environment. There may be some additional examples of what a command might look like with actual values.

    Any time `<>` will be used not as placeholders, a note like this will explain the usage for that case.

To use Proxmoxer, the library must be imported and a ProxmoxerAPI instance created. This instance takes care of all the authentication, abstraction, and de-serialization of the API calls.

```python
from proxmoxer import ProxmoxAPI

proxmox = ProxmoxAPI('<host_ip_or_domain>', user='<username>@<realm>', password='<password>', verify_ssl=False)
```

!!! tip "`verify_ssl` Parameter"
    If you have set up valid SSL certificates, you can remove the `verify_ssl=False`, but the default self-signed certificates will throw an error without `verify_ssl=False`.

### Switching Services or Backends

This will default to connecting to a 'PVE' service using the `https` backend. To change these defaults, specify a `service` or `backend` parameter of a [supported service](index.md#supported-services) and/or a [supported backend](index.md#supported-backends).

e.g. `proxmox = ProxmoxAPI('<host_ip_or_domain>', user='<username>', backend='openssh', service='pmg')`

### Changing Ports

Proxmoxer (when using the https backend) will use the default port for the selected service to connect to the service. If you need to connect on a different port, adding either the `port=<port_number>` parameter or adding `:<port>` to the host can be used. A port in the host will override the `port` parameter. The `port` parameter can also be used for the SSH-based backends to change the port used to establish a SSH connection.

## Making API Calls

There are two ways to build API requests: dotted notation and string notation. The format and benefits of each are presented below.

### Dotted Notation

Dotted notation can be thought of as treating the API call as a collection of dicts. This means that you can specify the API endpoint in a pythonic manner and let proxmoxer take care of converting to what the Proxmox service requires.

Lets say you want to see what users are available and what containers are on the "example-node" node and that `prox` is a correctly initialized ProxmoxerAPI object (for a PVE service in these examples).

```pycon
>>> # this is equivalent to https://pve.proxmox.com/pve-docs/api-viewer/index.html#/access/users
>>> prox.access.users.get()
[{'expire': 0, 'realm-type': 'pam', 'enable': 1, 'email': 'admin@example.com', 'userid': 'root@pam'}, {'expire': 0, 'realm-type': 'pve', 'enable': 1, 'userid': 'testing@pve'}]
```

To add variable values like the names of users or nodes, simply add parentheses to the section before the variable and pass in the string to use (as demonstrated below).

```pycon
>>> # this is equivalent to https://pve.proxmox.com/pve-docs/api-viewer/index.html#/nodes/{node}/lxc
>>> prox.nodes("example-node").lxc.get()
[{'cpu': 0, 'netin': 2122469071629, 'maxmem': 4294967296, 'diskread': 0, 'name': 'container-a', 'maxdisk': 4831838208, 'pid': 1098784, 'mem': 241815552, 'type': 'lxc', 'diskwrite': 0, 'netout': 573718142259, 'status': 'running', 'disk': 3100377088, 'swap': 0, 'maxswap': 536870912, 'vmid': '100', 'uptime': 7457751, 'cpus': 4},{'cpu': 0, 'netin': 1234, 'maxmem': 1234, 'diskread': 0, 'name': 'container-b', 'maxdisk': 1234, 'pid': 1234, 'mem': 1234, 'type': 'lxc', 'diskwrite': 0, 'netout': 1234, 'status': 'running', 'disk': 1234, 'swap': 0, 'maxswap': 1234, 'vmid': '101', 'uptime': 1234, 'cpus': 4}]
```

To get all the containers running on all the nodes in a cluster, you can iterate through all the nodes and get the containers running on each.

```pycon
>>> # get the list of all the nodes available from the connected node
>>> print(proxmox.nodes.get())
[{'type': 'node', 'node': 'example-node', 'ssl_fingerprint': '63:80:22:...:0D:12', 'id': 'node/example-node', 'status': 'unknown'}]
>>> # use the list of nodes to iterate through each and print the containers and their status
>>> for pve_node in proxmox.nodes.get():
...     print("{0}:".format(pve_node['node']))
...     for container in proxmox.nodes(pve_node['node']).lxc.get():
...         print("\t{0}. {1} => {2}".format(container['vmid'], container['name'], container['status']))
... 
example-node:
        100. container-a => running
        101. container-b => running
        105. container-c => running
```

### String Notation

String notation allows the developer to specify the exact URL path to be used for the API call. This puts more responsibility on the developer to correctly format the path but gives ultimate configurability. The dotted notation examples are displayed below converted into string notation.

Lets say you want to see what users are available and what containers are on the "example-node" node and that `prox` is a correctly initialized ProxmoxerAPI object (for a PVE service in these examples).

```pycon
>>> # this is equivalent to https://pve.proxmox.com/pve-docs/api-viewer/index.html#/access/users
>>> prox("access/users").get()
[{'expire': 0, 'realm-type': 'pam', 'enable': 1, 'email': 'admin@example.com', 'userid': 'root@pam'}, {'expire': 0, 'realm-type': 'pve', 'enable': 1, 'userid': 'testing@pve'}]
```

To add variable values like the names of users or nodes, simply add parentheses to the section before the variable and pass in the string to use (as demonstrated below).

```pycon
>>> # this is equivalent to https://pve.proxmox.com/pve-docs/api-viewer/index.html#/nodes/{node}/lxc
>>> prox("nodes/example-node/lxc").get()
[{'cpu': 0, 'netin': 2122469071629, 'maxmem': 4294967296, 'diskread': 0, 'name': 'container-a', 'maxdisk': 4831838208, 'pid': 1098784, 'mem': 241815552, 'type': 'lxc', 'diskwrite': 0, 'netout': 573718142259, 'status': 'running', 'disk': 3100377088, 'swap': 0, 'maxswap': 536870912, 'vmid': '100', 'uptime': 7457751, 'cpus': 4},{'cpu': 0, 'netin': 1234, 'maxmem': 1234, 'diskread': 0, 'name': 'container-b', 'maxdisk': 1234, 'pid': 1234, 'mem': 1234, 'type': 'lxc', 'diskwrite': 0, 'netout': 1234, 'status': 'running', 'disk': 1234, 'swap': 0, 'maxswap': 1234, 'vmid': '101', 'uptime': 1234, 'cpus': 4}]
```

To get all the containers running on all the nodes in a cluster, you can iterate through all the nodes and get the containers running on each.

```pycon
>>> # get the list of all the nodes available from the connected node
>>> print(proxmox("nodes").get())
[{'type': 'node', 'node': 'example-node', 'ssl_fingerprint': '63:80:22:...:0D:12', 'id': 'node/example-node', 'status': 'unknown'}]
>>> # use the list of nodes to iterate through each and print the containers and their status
>>> for pve_node in proxmox.nodes.get():
...     print("{0}:".format(pve_node['node']))
...     for container in proxmox("nodes/{0}/lxc".format(pve_node['node'])).get():
...         print("\t{0}. {1} => {2}".format(container['vmid'], container['name'], container['status']))
... 
example-node:
        100. container-a => running
        101. container-b => running
        105. container-c => running
```

### Combining Dotted and String Notation

There are a few situations where combining the notations is required or more effective.

#### Variable Data

As mentioned in the [Dotted Notation](#dotted-notation) section, there are times when string notation is needed in combination with dotted notation.

#### Invalid Names In Python

Some endpoints in the Proxmox APIs include a hyphen ("-"). When using dotted notation, python interprets this as subtraction rather than a hyphen in the name. In this case, string notation can be used for that section of the path and using dotted notation for the rest of the path.

For example, if you want to check the status of a command running in a VM, you would need to do the following

```pycon
>>> # using https://pve.proxmox.com/pve-docs/api-viewer/index.html#/nodes/{node}/qemu/{vmid}/agent/exec-status
>>> prox.nodes("example-node").qemu("103").agent("exec").post(command="echo hello")
{'pid': 5413}
>>> prox.nodes("example-node").qemu("103").agent("exec-status").get(pid=5413)
{'out-data': 'hello\n', 'exited': 1, 'exitcode': 0}
```

#### Examples

The following are all different ways of calling the same API path and will return the same result.

```python
prox.nodes(<node_name>).lxc.get()
prox.nodes(<node_name>).get('lxc')
prox.get('nodes/%s/lxc' % <node_name>)
prox.get('nodes', <node_name>, 'lxc')
prox('nodes')(<node_name>).lxc.get()
prox(['nodes', <node_name>]).lxc.get()
prox(['nodes', <node_name>]).get('lxc')
prox('nodes')(<node_name>)('lxc').get()
```
