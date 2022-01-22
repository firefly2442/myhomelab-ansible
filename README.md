# myhomelab-ansible

My personal Ansible playbooks for configuring my homelab.

## Prerequisites

Setup WSL2 or have an existing Linux machine with Ansible installed as the control node.

Install Ansible

## Setup

Make sure from the control node, you've already SSH'ed into each node
and that the connection has been added to the `known_hosts` file.

## Simple Ad-Hoc Tests

Perform a simple "ping" test to see if each of the nodes in the inventory are online.

```shell
ansible ubuntu -m ping -i inventory.yml -k
```

## Running Playbooks

Initial configurations common across all systems

```shell
ansible-playbook playbooks/initial_setup_configuration.yml -f 10 -i inventory.yml -K -k
```

Install the common packages used across all systems

```shell
ansible-playbook playbooks/install_common_packages.yml -f 10 -i inventory.yml -K -k
```

Update APT metadata and look for package upgrades

```shell
ansible-playbook playbooks/upgrade_latest_packages.yml -f 10 -i inventory.yml -K -k
```

Install Steam

```shell
ansible-playbook playbooks/install_steam_playbook.yml -f 10 -i inventory.yml -K -k
```

## Testing

Validate the `inventory.yml` file

```shell
ansible-inventory -i inventory.yml --list
```

Unit Tests:

```shell
ansible-test units
```

Integration tests:

```shell
ansible-test integration
```

## References

* [Quick Start Video](https://www.ansible.com/resources/videos/quick-start-video)
