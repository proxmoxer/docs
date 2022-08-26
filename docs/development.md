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
  * This branch is what the PyPI versions are built from
  * This branch should be as stable as the PyPI versions, so installing from here is unnecessary
* develop
  * This branch contains the improvements and new features which will be bundled into a future version
  * This branch should be mostly stable, but can have bugs which are removed before bundled into a version
  * Installing from this branch allows using the latest features and bug testing them

To install from source control, use `git+https://github.com/proxmoxer/proxmoxer.git@<branch>`. You will most likely need to specify the `-U` (`--upgrade`) flag to overwrite the existing version with
the source control version.
For example:

```shell
pip install --upgrade git+https://github.com/proxmoxer/proxmoxer.git@develop
```

More information from installing from source control can be found in the [VCS Support](https://pip.pypa.io/en/stable/topics/vcs-support/) pip documentation.

## Developing using Dev Containers

If you use [Visual Studio Code]() for development, you can utilize the great Containers remote to use a containerize development environment. This allows you to use a specific python version or install any packages without interfering with any other development environments you may have. And if you mess anything up or are done contributing, you can simply delete the container and volume and the entire codebase and dependencies are cleaned up.

1. Install Docker ([Instructions](https://www.docker.com/get-started))
2. Install Visual Studio Code ([Instructions](https://code.visualstudio.com/download))
3. Install the ["Remote - Containers" extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)
4. Clone this repository into a container ([Instructions](https://code.visualstudio.com/docs/remote/containers#_quick-start-open-a-git-repository-or-github-pr-in-an-isolated-container-volume))
5. Wait for the configuration to be loaded, dependencies installed, setup completed, and suggested extensions installed

Now you can use the included tasks, linters, and testing support to easily develop on any platform.

## Developing using Manual Installation

If you use another development environment or just don't want to use devcontainers, use the steps below to get started developing Proxmoxer.

1. Install Python 3 ([Instructions](https://realpython.com/installing-python/))
2. Install Pip for Python 3 ([Instructions](https://pip.pypa.io/en/stable/installing/))
3. clone this repo with `git clone https://github.com/proxmoxer/proxmoxer.git` and change into the `proxmoxer` directory
4. Install dependencies with `pip3 install --user -r test_requirements.txt -r dev_requirements.txt`
