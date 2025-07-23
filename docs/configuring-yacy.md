<!--
SPDX-FileCopyrightText: 2020 - 2024 MDAD project contributors
SPDX-FileCopyrightText: 2020 - 2024 Slavi Pantaleev
SPDX-FileCopyrightText: 2020 Aaron Raimist
SPDX-FileCopyrightText: 2020 Chris van Dijk
SPDX-FileCopyrightText: 2020 Dominik Zajac
SPDX-FileCopyrightText: 2020 Mickaël Cornière
SPDX-FileCopyrightText: 2022 François Darveau
SPDX-FileCopyrightText: 2022 Julian Foad
SPDX-FileCopyrightText: 2022 Warren Bailey
SPDX-FileCopyrightText: 2023 Antonis Christofides
SPDX-FileCopyrightText: 2023 Felix Stupp
SPDX-FileCopyrightText: 2023 Pierre 'McFly' Marty
SPDX-FileCopyrightText: 2024 - 2025 Suguru Hirahara

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Setting up YaCy

This is an [Ansible](https://www.ansible.com/) role which installs [YaCy](https://yacy.net) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

YaCy is a distributed web search engine, based on a peer-to-peer network. It provides three different modes;

- Searching a shared global index on the P2P network
- Crawling web pages of domains you choose to create an individual index for searching
- Setting up a search portal for your intranet behind the firewall to search pages or files on the shared file system, without sharing data with a third party

See the project's [documentation](https://yacy.net/docs/) to learn what YaCy does and why it might be useful to you.

## Prerequisites

### Open ports

You may need to open the following ports on your server:

- `8090` — port number where the server should bind to
- `8443` — optional SSL port (HTTPS port) the server should bind to

Docker automatically opens these ports in the server's firewall, so you likely don't need to do anything. If you use another firewall in front of the server, you may need to adjust it.

See the upstream [documentation](https://yacy.net/operation/yacy_conf/#system) to learn more.

## Adjusting the playbook configuration

To enable YaCy with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# yacy                                                                 #
#                                                                      #
########################################################################

yacy_enabled: true

########################################################################
#                                                                      #
# /yacy                                                                #
#                                                                      #
########################################################################
```

### Set the hostname

To enable YaCy you need to set the hostname as well. To do so, add the following configuration to your `vars.yml` file. Make sure to replace `example.com` with your own value.

```yaml
yacy_hostname: "example.com"
```

After adjusting the hostname, make sure to adjust your DNS records to point the domain to your server.

### Extending the configuration

There are some additional things you may wish to configure about the component.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `yacy_environment_variables_additional_variables` variable

See [`yacy.init`](https://github.com/yacy/yacy_search_server/blob/master/defaults/yacy.init) for a complete list of YaCy's config options that you could put in `yacy_environment_variables_additional_variables`.

>[!NOTE]
> You can check [this section on the documentation](https://yacy.net/download_installation/#configuration-with-environment-variables) for the conversion rule of settings into environment variables. Note that not all settings are available as environment variables.

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, YaCy becomes available at the specified hostname like `https://example.com`.

### Change admin user password

**Do not forget to change the default login credential of the admin account** (username: `admin`, password: `yacy`). You can change it at `https://example.com/ConfigAccounts_p.html`. Alternatively, set a password of the `admin` user to the `yacy_conf_password` variable and run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=ensure-yacy-admin-password
```

>[!NOTE]
> The password cannot be changed with the playbook's tag if YaCy instance is not running.

### Change the search mode

Since the password of the default admin account is [hardcoded](https://github.com/yacy/yacy_search_server/blob/master/docker/Dockerfile), the YaCy instance is set to **the intranet search mode** by default for safety, so that it does not broadcast its existence to peers before you change the login credential.

After you have changed the password on the UI or by running the playbook with `ensure-yacy-reset-password` tag, you can change the search mode to another one such as "Community-based web search" (global index search mode on the P2P network) from the UI directly at `https://example.com/ConfigBasic.html` or by adding the following configuration to your `vars.yml` file:

```yaml
yacy_environment_variables_network_unit_definition: "defaults/yacy.network.freeworld.unit"
```

>[!WARNING]
> Search mode setting changed on the UI is temporary and reset to the mode specified on the configuration file after re-running the playbook. Set the configuration to your `vars.yml` file to make the setting persistent.

### Password-protect the instance

By default any page without the `_p` suffix on YaCy instance is accessible to anyone, whether the instance is broadcasted or not over the P2P network, while executing administrative tasks such as changing configuration requires logging in to the instance.

To require authorization with a password for accessing the instance, you can set `adminAccountAllPages` to `false` at `https://example.com/ConfigProperties_p.html`. It is also possible to configure it on the "Access Rules" section at `https://example.com/ConfigAccounts_p.html`.

## Troubleshooting

[The official documentation](https://yacy.net/docs/) and [FAQ](https://yacy.net/faq/) are available for troubleshooting.

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH and running `journalctl -fu yacy` (or how you/your playbook named the service, e.g. `mash-yacy`).
