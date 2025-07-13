# myhomelab-ansible

My personal Ansible playbooks for configuring my homelab.

## Prerequisites

Setup WSL2 or have an existing Linux machine with Ansible installed as the control node.

Install Ansible

## Setup

Make sure from the control node, you've already SSH'ed into each node
and that the connection has been added to the `~/.ssh/known_hosts` file.

## Simple Ad-Hoc Tests

Perform a simple "ping" test to see if each of the nodes in the inventory are online.

```shell
ansible ubuntu -m ping -i inventory.yml -k
```

## Running Playbooks

### Computers and Laptops

Initial configurations common across all systems

```shell
ansible-playbook playbooks/initial_setup_configuration.yml -i inventory.yml -K -k
```

Install the common packages used across all systems

```shell
ansible-playbook playbooks/install_common_packages.yml -i inventory.yml -K -k
```

Update APT metadata and look for package upgrades

```shell
ansible-playbook playbooks/upgrade_latest_packages.yml -i inventory.yml -K -k
```

Install Steam

```shell
ansible-playbook playbooks/install_steam_playbook.yml -i inventory.yml -K -k
```

Install Chrome

```shell
ansible-playbook playbooks/install_chrome_playbook.yml -i inventory.yml -K -k
```

Install software specific to laptop

```shell
ansible-playbook playbooks/install_laptop_packages.yml -i inventory.yml -K -k
```

Install software specific to BlueAntec for Nvidia Docker integration

```shell
ansible-playbook playbooks/install_blueantec_packages.yml -i inventory.yml -K -k
```

Install k3s Kubernetes cluster

```shell
ansible-playbook playbooks/setup_k3s_kubernetes.yml -i inventory.yml -K -k --extra-vars "token=secret master=192.168.1.112 loadbalancer=192.168.1.226 dockerhubtoken=secret"
```

| Node         | Control Plane | etcd | Worker | Longhorn | Notes                                      |
|--------------|---------------|------|--------|--------|--------------------------------------------|
| gmktec       | ✅            | ✅   | ✅    | ✅    | Core node, part of quorum                  |
| dellxps      | ✅            | ✅   | ✅    | ✅    | Core node, part of quorum                  |
| antecbackup  | ✅            | ✅   | ✅    | ✅    | Core node, part of quorum                  |
| system76laptop | ✅          | ❌   | ❌    | ❌   | Control-plane only (no etcd or workloads)  |
| alienware    | ❌            | ❌   | ✅    | ❌    | Worker only                                |
| blueantec    | ❌            | ❌   | ✅    | ❌    | Worker only                                |

A `k3s` systemd service will be setup on the control-plane nodes.  On the worker only nodes,
it will be `k3s-agent` for the systemd service.  The workers are also setup as longhorn
clients meaning they do not directly store any data.  They can mount data from other
nodes over the network.  This way when they are shutdown, it doesn't impact having to
rebuild and sync data across the network.

Install applications to k3s Kubernetes cluster

```shell
ansible-playbook playbooks/deploy_k3s_applications.yml -i inventory.yml -K -k
```

Teardown entire k3s Kubernetes cluster

```shell
ansible-playbook playbooks/teardown_k3s_kubernetes.yml -i inventory.yml -K -k
```

### Single Machine

To run a playbook on a single machine add the `--limit` flag.  For example:

```shell
ansible-playbook playbooks/install_chrome_playbook.yml -i inventory.yml -K -k --limit system76laptop
```

### Wireless Router via OpenWRT

Manually install either a fresh copy of OpenWRT or install the upgrade firmware.  This
may require a reboot of the cable modem in order to get the WAN connectivity
to initialize again.

Copy the public and private keys from Windows into WSL2

```shell
cp -r /mnt/c/Users/Patrick/.ssh/ ~/
sudo chmod 700 ~/.ssh/
sudo chmod 644 ~/.ssh/*.pub
sudo chmod 600 ~/.ssh/id_rsa
```

Add the `id_rsa.pub` contents of the public key to the OpenWRT SSH-Keys page.

You may need to add this to `~/.ssh/config` if it's using an old SSH algorithm.

```contents
Host 192.168.1.1
  User root
  HostkeyAlgorithms +ssh-rsa
  PubkeyAcceptedAlgorithms +ssh-rsa
```

Ansible also needs Python3 so we need to manually install it on the router first.

```shell
opkg update
opkg install python3
```

Edit and evaluate `./playbooks/secrets/wireless` as needed.  This has our wifi radio
details as well as the password key.

Run the playbook

```shell
ansible-playbook playbooks/setup_router_playbook.yml -i inventory.yml --private-key=~/.ssh/id_rsa
```

## Testing

Validate the `inventory.yml` file

```shell
ansible-inventory -i inventory.yml --list
```

## Debugging

Use `-vvv` for more verbose logging from Ansible.

## References

* [Quick Start Video](https://www.ansible.com/resources/videos/quick-start-video)
* [Ansible Documentation](https://docs.ansible.com/)
