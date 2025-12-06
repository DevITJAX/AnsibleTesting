# Ansible Testing Environment

This project sets up a multi-node Vagrant environment for testing Ansible automation. It includes a control node with Ansible installed and four managed nodes (database, web servers, and load balancer).

## Infrastructure Overview

The environment consists of 5 Ubuntu 22.04 VMs:

| Hostname      | IP Address       | SSH Port | Role                |
|---------------|------------------|----------|---------------------|
| controlnode   | 192.168.169.135  | 2214     | Ansible Control Node|
| db01          | 192.168.169.130  | 2210     | Database Server     |
| web01         | 192.168.169.131  | 2211     | Web Server 1        |
| web02         | 192.168.169.132  | 2212     | Web Server 2        |
| loadbalancer  | 192.168.169.134  | 2213     | Load Balancer       |

Each VM is allocated 2GB of RAM.

## Prerequisites

- [Vagrant](https://www.vagrantup.com/) installed
- [VirtualBox](https://www.virtualbox.org/) installed
- At least 10GB of free RAM
- At least 20GB of free disk space

## Quick Start

### 1. Clone and Start the Environment

```bash
# Clone the repository (if applicable)
git clone <repository-url>
cd AnsibleTesting

# Start all VMs
vagrant up
```

This will:
- Create and configure all 5 VMs
- Install Ansible on the controlnode
- Generate SSH keys on the controlnode
- Copy the Ansible inventory file to `/etc/ansible/hosts`
- Enable password authentication on managed nodes

### 2. Set Up SSH Key Authentication

SSH into the controlnode:

```bash
vagrant ssh controlnode
```

Distribute the SSH key to all managed nodes (password is `vagrant`):

```bash
ssh-copy-id vagrant@192.168.169.130  # db01
ssh-copy-id vagrant@192.168.169.131  # web01
ssh-copy-id vagrant@192.168.169.132  # web02
ssh-copy-id vagrant@192.168.169.134  # loadbalancer
```

### 3. Test Ansible Connectivity

Verify that Ansible can connect to all nodes:

```bash
ansible all -m ping
```

Expected output:
```
db01 | SUCCESS => { ... }
web01 | SUCCESS => { ... }
web02 | SUCCESS => { ... }
loadbalancer | SUCCESS => { ... }
```

## Ansible Inventory

The inventory file is located at `/etc/ansible/hosts` on the controlnode and organizes hosts into groups:

- **[database]**: db01
- **[webservers]**: web01, web02
- **[loadbalancers]**: loadbalancer

### Example Ansible Commands

```bash
# Ping all hosts
ansible all -m ping

# Ping only web servers
ansible webservers -m ping

# Check disk space on all hosts
ansible all -m shell -a "df -h"

# Update packages on database server
ansible database -m apt -a "update_cache=yes" --become

# Get system information from all hosts
ansible all -m setup
```

## Managing the Environment

### Start VMs
```bash
vagrant up
```

### Stop VMs
```bash
vagrant halt
```

### Restart a specific VM
```bash
vagrant reload <hostname>
```

### SSH into a VM
```bash
vagrant ssh <hostname>
# Example: vagrant ssh controlnode
```

### Destroy the environment
```bash
vagrant destroy -f
```

### Re-provision (apply configuration changes)
```bash
vagrant provision
# Or for a specific VM:
vagrant provision controlnode
```

## Project Structure

```
AnsibleTesting/
├── Vagrantfile          # VM configuration
├── hosts                # Ansible inventory file
├── .gitignore          # Git ignore rules
└── README.md           # This file
```

## Troubleshooting

### SSH Connection Issues

If you get "Permission denied" errors:

1. Verify SSH keys are distributed:
   ```bash
   ssh vagrant@192.168.169.130  # Should connect without password
   ```

2. Check the inventory file path:
   ```bash
   cat /etc/ansible/hosts
   ```

3. Re-provision the controlnode:
   ```bash
   vagrant provision controlnode
   ```

### VM Not Starting

1. Check VirtualBox is running
2. Verify you have enough RAM available
3. Try destroying and recreating:
   ```bash
   vagrant destroy -f
   vagrant up
   ```

### Ansible Not Found

If Ansible is not installed on controlnode:
```bash
vagrant provision controlnode
```

## Next Steps

Now that your environment is set up, you can:

1. Create Ansible playbooks in `/home/vagrant/` on the controlnode
2. Practice writing roles and tasks
3. Test automation scenarios across your infrastructure
4. Experiment with different Ansible modules

## Additional Resources

- [Ansible Documentation](https://docs.ansible.com/)
- [Vagrant Documentation](https://www.vagrantup.com/docs)
- [VirtualBox Documentation](https://www.virtualbox.org/manual/)

## License

This is a testing environment for learning purposes.
