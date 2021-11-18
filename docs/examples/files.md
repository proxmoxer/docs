<!-- spell-checker:ignore UPID -->
# Files

## Uploading Files to PVE

If you have files on your device which you want to upload to your PVE, it is easy to do.

First, to upload large file and to prevent high memory usage, install the `requests_toolbelt` pip package (`pip install requests_toolbelt`).

Next, you will need to open the file you want to upload.

```python
f = open("<file_path>", "rb")
```

Finally, upload the file specifying the correct `content` type ("template" or "iso").

```python
prox.proxmox.nodes('<node_name>').storage('<storage_name>').upload.post(content='<content_type>', filename=f)
```

## Downloading Files to PVE

An alternative to downloading a file to your computer and then uploading it to your PVE instance, you can request PVE directly
download the file from its URL.

Assuming `prox` is a valid `ProxmoxerAPI` object and `sourceURL` is a complete URL of a file to download. `content` is the same options as above
and `filename` is the name the file will be saved as on the PVE storage.

```python
proxmox.nodes("<node_name>").storage("<storage_name>")("download-url").post(url=sourceURL, content="<content_type>", filename="<file_name.extension>")
```

To ensure the file downloaded is the file you expect, you can also specify a hash and algorithm which PVE will check against the file it downloads.

```python
proxmox.nodes("<node_name>").storage("<storage_name>")("download-url").post(url=sourceURL, content="<content_type>", filename="<file_name.extension>", 
    checksum="<hash>", "checksum-algorithm": "<md5 | sha1 | sha224 | sha256 | sha384 | sha512>")
```

This post request will return the [UPID of the task](../tasks/#what-is-a-task-upid) created for that download.
To wait (block) until a download is complete, see the [Blocking Until Task is Complete](../tasks/#blocking-until-task-is-complete) section.
