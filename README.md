# Ansible SSH Tunnels - Systemd or Supervisor Managed Persistant SSH Tunnels

This is a simple Ansible role to help manage the deployment and cleanup of multiple SSH tunnels across your inventory. The SSH tunnels are run on your hosts using either systemd service or supervisor managed process. Both can be used simultaneously as well. 

ExitOnForwardFailure is enabled on the SSH tunnel and the process manager will automatically restart the tunnel when it fails.

## Getting Started

It is assumed that the connecting servers creating the tunnels have passwordless SSH access. How this is achieved is out of the scope of this Anisble role and we leave this for you to decide. The identity file can be manually set for each tunnel in the inventory if necessary.

Install this role using Ansible Galaxy:
```bash
ansible-galaxy install git+https://github.com/bernardko/ansible-ssh-tunnels.git
```

Configuration is straightforward, just add them to the host you want to create the tunnel from in your inventory:

```yaml
all:
  hosts:
    server1:
    server2:
      ssh_tunnels:
        systemd:
          - name: to_server1
            user: user
            process_group: user
            identity: /home/user/.ssh/id_rsa
            forward_mode: R
            forward: 2222:localhost:22
            host: user@server1
            port: 22

  children:
    tunnelservers:
      hosts:
        server1:
          ssh_tunnels:
            supervisor:
              # This is the minimum configuration.
              - name: to_server2
                user: user
                forward_mode: R
                forward: 3333:localhost:22
                host: user@server2

        server2:
```

An example of a simple playbook to run this is as follows:
```yaml
---
- name: SSH Tunnel Configuration
  hosts: tunnelservers
  become: yes
  roles:
    - role: 'ansible-ssh-tunnels'
```