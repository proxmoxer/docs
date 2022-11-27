# Tasks Tools

## What is a task UPID?

Most actions on Proxmox services are backed by tasks. These are visible in the web UI and are created when automation or users request actions be completed.

Tasks are uniquely identified by a UPID which follows the format

```text
UPID:<node_name>:<pid_in_hex>:<pstart_in_hex>:<starttime_in_hex>:<type>:<id (optional)>:<user>@<realm>:
```

## Class Methods

The functionality of the Tasks tools class is available through static methods. This means you don not need to instantiate a Tasks object to be able to call the methods.

### blocking_status

Turns getting the status of a Proxmox task into a blocking call by polling the API until the task completes.

* `prox` (ProxmoxAPI) - The ProxmoxAPI object to use to query for status
* `task_id` (str) - the UPID of the task
* `timeout` (float, optional) - If the task does not complete in this time (in seconds) return None, defaults to 300
* `polling_interval` (float, optional) - the time to wait between checking for status updates, defaults to 0.01

#### Examples

```pycon
>>> from proxmoxer.tools import Tasks
>>> from proxmoxer import ProxmoxAPI
>>> prox = ProxmoxAPI("1.2.3.4", user="root@pam", password="password", verify_ssl=False)
>>> Tasks.blocking_status(prox, "UPID:example-node:002E096A:0E10F08E:6382D261:vzcreate:123:root@pam:")
{'pstart': 235991182, 'exitstatus': 'OK', 'pid': 3017066, 'user': 'root@pam', 'starttime': 1669517921, 'upid': 'UPID:example-node:002E096A:0E10F08E:6382D261:vzcreate:123:root@pam:', 'type': 'vzcreate', 'node': 'hydrogen-pve-0', 'id': '123', 'status': 'stopped'}
>>>
```

### decode_upid

Decodes the sections of a UPID into separate fields

* `upid` (str) - a UPID string
* returns dict of UPID sections
  * `upid` (str) - the original upid
  * `node` (str) - the name of the node which started/owns the task
  * `pid` (int) - the process ID of the task
  * `pstart` (int) - the relative timestamp of the start of the process
  * `starttime` (int) - the epoch timestamp of the start of the task
  * `type` (str)  - the type of task represented by the UPID
  * `id` (str) - an identifier additional to the type (usually the resource (e.g. VM) ID)
  * `user` (str) - The `<user>@<realm>` who initiated the task
  * `comment` (str) - an optional extra field, often not used

#### Examples

```pycon
>>> from proxmoxer.tools import Tasks
>>> Tasks.decode_upid("UPID:example-node:000AE992:00A21BA7:618C1D55:vzdump:100:root@pam:")
{'upid': 'UPID:example-node:000AE992:00A21BA7:618C1D55:vzdump:100:root@pam:', 'node': 'example-node', 'pid': 715154, 'pstart': 10623911, 'starttime': 1636572501, 'type': 'vzdump', 'id': '100', 'user': 'root@pam', 'comment': ''}
>>> Tasks.decode_upid("UPID:example-node:00213D37:01ECA9AD:618F6B8D:aptupdate::root@pam:")
{'upid': 'UPID:example-node:00213D37:01ECA9AD:618F6B8D:aptupdate::root@pam:', 'node': 'example-node', 'pid': 2178359, 'pstart': 32287149, 'starttime': 1636789133, 'type': 'aptupdate', 'id': '', 'user': 'root@pam', 'comment': ''}
>>>
```

### decode_logs

Decodes the JSON log of Proxmox tasks to a plain string. Joins the lines with `\n`.

`log_list` (dict[]) - the list of log lines which is returned from the Proxmox service
returns a string

#### Examples

```pycon
>>> from proxmoxer.tools import Tasks
>>> from proxmoxer import ProxmoxAPI
>>> l = prox.nodes("example-node").tasks("UPID:example-node:002E096A:0E10F08E:6382D261:vzcreate:123:root@pam:").log.get()
>>> l
[{'t': "extracting archive '/mnt/pve/hl01-files/template/cache/ubuntu-22.04-standard_22.04-1_amd64.tar.zst'", 'n': 1}, {'t': 'Total bytes read: 508579840 (486MiB, 194MiB/s)', 'n': 2}, {'t': 'Detected container architecture: amd64', 'n': 3}, {'t': "Creating SSH host key 'ssh_host_rsa_key' - this may take some time ...", 'n': 4}, {'n': 5, 't': 'done: SHA256:zUpB9ln5sJLrBgjU97LBSNvYItH0ByEU8VXeURfnt1o root@test-container'}, {'n': 6, 't': "Creating SSH host key 'ssh_host_dsa_key' - this may take some time ..."}, {'n': 7, 't': 'done: SHA256:YAf2Od306G2ACLbJ2wbF1rrzo0q8rZNMZlr8wvCgsMY root@test-container'}, {'n': 8, 't': "Creating SSH host key 'ssh_host_ed25519_key' - this may take some time ..."}, {'n': 9, 't': 'done: SHA256:O2pIbtFCdPPiY+Q6PnDxJUAmEAc5vaSku+Vg0s6gxkQ root@test-container'}, {'n': 10, 't': "Creating SSH host key 'ssh_host_ecdsa_key' - this may take some time ..."}, {'n': 12, 't': 'TASK OK'}, {'t': 'done: SHA256:xz1iY4+3/0jTSZbxWxEkouZvD2VJOfkAsQEFNWh1rIY root@test-container', 'n': 11}]
>>> Tasks.decode_log(l)
"extracting archive '/mnt/pve/nas/template/cache/ubuntu-22.04-standard_22.04-1_amd64.tar.zst'\nTotal bytes read: 508579840 (486MiB, 194MiB/s)\nDetected container architecture: amd64\nCreating SSH host key 'ssh_host_rsa_key' - this may take some time ...\ndone: SHA256:zUpB9ln5sJLrBgjU97LBSNvYItH0ByEU8VXeURfnt1o root@test-container\nCreating SSH host key 'ssh_host_dsa_key' - this may take some time ...\ndone: SHA256:YAf2Od306G2ACLbJ2wbF1rrzo0q8rZNMZlr8wvCgsMY root@test-container\nCreating SSH host key 'ssh_host_ed25519_key' - this may take some time ...\ndone: SHA256:O2pIbtFCdPPiY+Q6PnDxJUAmEAc5vaSku+Vg0s6gxkQ root@test-container\nCreating SSH host key 'ssh_host_ecdsa_key' - this may take some time ...\ndone: SHA256:xz1iY4+3/0jTSZbxWxEkouZvD2VJOfkAsQEFNWh1rIY root@test-container\nTASK OK"
>>> print(Tasks.decode_log(l))
extracting archive '/mnt/pve/nas/template/cache/ubuntu-22.04-standard_22.04-1_amd64.tar.zst'
Total bytes read: 508579840 (486MiB, 194MiB/s)
Detected container architecture: amd64
Creating SSH host key 'ssh_host_rsa_key' - this may take some time ...
done: SHA256:zUpB9ln5sJLrBgjU97LBSNvYItH0ByEU8VXeURfnt1o root@test-container
Creating SSH host key 'ssh_host_dsa_key' - this may take some time ...
done: SHA256:YAf2Od306G2ACLbJ2wbF1rrzo0q8rZNMZlr8wvCgsMY root@test-container
Creating SSH host key 'ssh_host_ed25519_key' - this may take some time ...
done: SHA256:O2pIbtFCdPPiY+Q6PnDxJUAmEAc5vaSku+Vg0s6gxkQ root@test-container
Creating SSH host key 'ssh_host_ecdsa_key' - this may take some time ...
done: SHA256:xz1iY4+3/0jTSZbxWxEkouZvD2VJOfkAsQEFNWh1rIY root@test-container
TASK OK
>>>
```
