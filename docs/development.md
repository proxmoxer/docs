# Development

--8<-- "stub.md"

## Installing from Source

Sometimes there are features that have not been added to a published version in PyPI but are
in the development source code in Git. You can install directly from the source code branches
to test these features or develop more on them.

!!! warning "Stability not guaranteed"
    Installing from source control (Git) branches is inherently less stable than installing
    from PyPI. Installing like this will also not update when new versions or commits are published.
    Code in source control may be incomplete or broken. No support will be given
    for issues that arise from installing in this way, however bug reports of broken code
    are appreciated.

There are two main branches to install from:

* master
  * This branch is what the PyPI version are built from
  * This branch should be as stable as the PyPI versions, so installing from here is unnecessary
* develop
  * This branch contains the improvements and new features which will be bundled into a future version
  * This branch should be mostly stable, but can have bugs which are removed before moved to a version
  * Installing from this branch allows using the latest features and bug testing them

To install from source control, use `git+https://github.com/proxmoxer/proxmoxer.git@<branch>`. You will most likely need to specify the `-U` (`--upgrade`) flag to overwrite the existing version with
the source control version.
For example:

```shell
pip install --upgrade git+https://github.com/proxmoxer/proxmoxer.git@develop
```

More information from installing from source control can be found in the [VCS Support](https://pip.pypa.io/en/stable/topics/vcs-support/) pip documentation.
