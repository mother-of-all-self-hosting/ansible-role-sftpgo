<!--
SPDX-FileCopyrightText: 2018-2025 Slavi Pantaleev
SPDX-FileCopyrightText: 2019-2022 Aaron Raimist
SPDX-FileCopyrightText: 2019-2023 MDAD project contributors
SPDX-FileCopyrightText: 2023 QEDeD
SPDX-FileCopyrightText: 2024 Fabio Bonelli
SPDX-FileCopyrightText: 2024 Nikita Chernyi
SPDX-FileCopyrightText: 2024-2026 Suguru Hirahara
SPDX-FileCopyrightText: 2026 spatterlight

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Molecule Testing

This role supports [Molecule](https://docs.ansible.com/projects/molecule/), an Ansible testing framework designed for developing and testing Ansible collections, playbooks, and roles.

## Prerequisites

To utilize Molecule you need to prepare several requirements:

- **x86** computer running one of these operating systems that make use of [systemd](https://systemd.io/):
  - **Archlinux**
  - **CentOS**, **Rocky Linux**, **AlmaLinux**, or possibly other RHEL alternatives (although your mileage may vary)
  - **Debian** (10/Buster or newer)
  - **Ubuntu** (18.04 or newer, although [20.04 may be problematic](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/ansible.md#supported-ansible-versions) if you run the Ansible playbook on it)
- `root` access on the computer which Molecule runs against
- [Ansible](http://ansible.com/) program
- [Python](https://www.python.org/)
  - Most distributions install Python by default, but some don't (e.g. Ubuntu 18.04) and require manual installation (something like `apt-get install python3`)
- [Docker](https://www.docker.com)
  - Access to Docker UNIX socket (`/var/run/docker.sock`) is required by default

## Installation

To set up the environment for using Molecule, run the command below on the terminal:

```bash
python3 -m venv ./molecule/venv
source ./molecule/venv/bin/activate
pip3 install -r ./molecule/requirements.txt
```

## Scenarios

Currently these testing scenarios are available:

Every scenario deploys SFTPGo, bootstraps an admin account from the role's `env` file, creates a user through the admin REST API and then uploads a file over a real SFTP session and reads it back off the host, so that what is verified is SFTPGo actually transferring files rather than merely answering HTTP. They differ in which data provider backs that, which HTTP interfaces are served, and which extra protocol or endpoint is exercised.

### `default`

A standard SFTPGo installation on SQLite, keeping the role's own HTTP surface: no WebAdmin, no WebClient, REST API only. Reads the SQLite data provider off the role's home path to confirm the admin and the user really landed there.

### `default-selfbuild`

The same, but building SFTPGo's own `Dockerfile` out of a checkout of the revision `sftpgo_version` pins, instead of pulling a published image. Only useful when that version changes, so CI gates it on a version bump (and on `workflow_dispatch`).

### `mariadb`

A standard SFTPGo installation with the MariaDB database, over a Unix socket. Opts into WebAdmin and WebClient and checks that the unauthenticated first-admin setup wizard is closed, enables the WebDAV server and does a WebDAV upload round trip, and queries MariaDB directly for the records - while asserting no SQLite file was created as a fallback.

### `postgres`

A standard SFTPGo installation with the Postgres database, over a Unix socket. Serves the WebClient without the administrative interface, turns the Prometheus telemetry endpoint on and asserts SFTPGo's own counters account for the transfers and report the provider reachable, and queries Postgres directly for the records - again while asserting no SQLite fallback.

## Running

By default it is configured to run the scenarios on Ubuntu 26.04.

```bash
molecule test --scenario-name default
```

You can utilize other distributions by setting one to the `MOLECULE_DISTRO` environment variable:

```bash
# Ubuntu 24.04
MOLECULE_DISTRO=ubuntu2404 molecule test --scenario-name default

# Debian 13
MOLECULE_DISTRO=debian13 molecule test --scenario-name default

# Debian 12
MOLECULE_DISTRO=debian12 molecule test --scenario-name default
```
