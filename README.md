# Ansible role to manage wireguard

This role allows you to deploy a fast, secure and provider agnostic private network between multiple servers. This is useful for providers that do not provide you with a private network or if you want to connect servers that are spread over multiple regions and providers.

Also you can use it to setup server -> clients wireguard.

Based on https://github.com/mawalu/wireguard-private-networking

## How

The role installs [wireguard](https://wireguard.com) on Debian, Ubuntu, Arch, Centos, creates a mesh between all servers by adding them all as peers and configures the wg-quick systemd service.

## Installation

Installation can be done using [ansible galaxy](https://galaxy.ansible.com/kshcherban/wireguard):

```
$ ansible-galaxy install kshcherban.wireguard
```

## Setup

Install this role, assign a `vpn_ip` variable to every host that should be part of the network and run the role. Here is a small example configuration:

Optionally, you can set a `public_addr` on each host. This address will be used to connect to the wireguard peer instead of the address in the inventory. Useful if you are configuring over a different network than wireguard is using. e.g. ansible connects over a LAN to your peer.

```yaml
# inventory host file

wireguard:
  hosts:
    192.168.1.1:
      vpn_ip: 10.1.0.1/32
      public_addr: "example.com" # optional
    192.168.2.2:
      vpn_ip: 10.1.0.2/32

```

```yaml
# playbook

- name: Configure wireguard mesh
  hosts: wireguard
  remote_user: root
  roles:
    - kshcherban.wireguard
```

```yaml
# playbook (with clients config)
- name: Configure wireguard mesh
  hosts: wireguard
  remote_user: root
  vars:
    clients:
      - name: user1
        ip: 10.1.0.11
      - name: user2
        ip: 10.1.0.12
        dns: 1.1.1.1
  roles:
    - kshcherban.wireguard
```

## Additional configuration

There are a small number of role variables that can be overwritten.

```yaml
wireguard_port: "5888" # the port to use for server to server connections
wireguard_path: "/etc/wireguard" # location of all wireguard configurations

wireguard_network_name: "wg0" # the name to use for the config file and wg-quick

wireguard_mtu: 1500 # Optionally a MTU to set in the wg-quick file. Not set by default. Can also be set per host

debian_enable_backports: true # if the debian backports repos should be added on debian machines

clients: [] # list of hashes with clients configuration
# clients configs will be saved locally into ~/wireguard-<client.name>.conf
# example block:
# clients:
#   - name: "foo"
#     ip: "10.0.0.2"
#     dns: "1.1.1.1"
# dns key is optional

# a list of additional peers that will be added to each server
wireguard_additional_peers:
  - comment: martin
    ip: 10.2.3.4
    key: your_wireguard_public_key
  - comment: other_network
    ip: 10.32.0.0/16
    key: their_wireguard_public_key
    keepalive: 20
    endpoint: some.endpoint:2230

wireguard_post_up: "iptables ..." # PostUp hook command, by default script from templates are used
wireguard_post_down: "iptables"   # PostDown hook command
```

## Contributing

Feel free to open issues or MRs if you find problems or have ideas for improvements. I'm especially open for MRs that add support for additional operating systems and more tests.

