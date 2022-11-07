# Proxmoxer Documentation

Documentation for the proxmoxer python library

This documentation is available ato <https://proxmoxer.github.io/docs>

[![Deploy main ("dev") branch](https://github.com/proxmoxer/docs/actions/workflows/gh-pages_dev.yaml/badge.svg?branch=main)](https://proxmoxer.github.io/docs/dev/)

## Developing

To build and serve the documentation locally, follow the steps below

1. `pip install -r requirements.txt` - install dependencies
2. `mkdocs serve` - serve the documentation
3. `mike serve` - OPTIONAL: serve the multi-version documentation

If you are using Visual Studio Code, you can instead just use the included devcontainer definition to develop within a Docker container. This allows automatically installs all needed dependencies and provides a consistent, repeatable development environment which does not conflict with anything else and can be easily removed.
