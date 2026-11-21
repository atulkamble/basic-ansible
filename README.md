# Basic Ansible Configuration

A beginner-friendly Ansible project for managing **Amazon Linux 2 and Amazon Linux 2023** EC2 instances. It includes SSH inventory configuration, a connectivity check, and a basic Nginx setup.

## Project structure

```text
basic-ansible/
├── ansible.cfg
├── .gitignore
├── inventory/
│   └── hosts.ini
├── group_vars/
│   └── all.yml
└── playbooks/
    ├── ping.yml
    └── setup-webserver.yml
```

## Requirements

- Ansible installed on your control machine (Linux, macOS, or WSL).
- A reachable Amazon Linux 2 or Amazon Linux 2023 EC2 instance.
- SSH access and the matching EC2 private key.

Install Ansible on Ubuntu/Debian:

```bash
sudo apt update
sudo apt install ansible -y
```

On macOS with Homebrew:

```bash
brew install ansible
```

## Configure SSH access

Edit `inventory/hosts.ini` and replace `YOUR_SERVER_PUBLIC_IP` with your EC2 instance's public IP or DNS name. The inventory uses the default Amazon Linux SSH user, `ec2-user`.

Place your private key at `aws/ansible.pem`, or update `private_key_file` in `ansible.cfg` to point to the key's actual location. Set restrictive permissions:

```bash
chmod 400 aws/ansible.pem
```

The key is excluded from Git. Never commit private keys, passwords, or cloud credentials. In the EC2 security group, allow SSH (TCP 22) from your control machine.

## Verify connectivity

Run from the repository root:

```bash
ansible --version
ansible-inventory --graph
ansible all -m ansible.builtin.ping
ansible-playbook playbooks/ping.yml
```

A successful result includes `"ping": "pong"`. The Ansible ping module verifies SSH and Python connectivity; it is not an ICMP network ping.

## Install and start Nginx

The web server playbook uses Ansible's OS-aware package module, so it works with both Amazon Linux 2 and Amazon Linux 2023.

```bash
ansible-playbook playbooks/setup-webserver.yml --limit web
```

It installs Nginx, enables and starts the service, then deploys a sample home page. Allow inbound HTTP (TCP 80) in the EC2 security group and visit:

```text
http://YOUR_SERVER_PUBLIC_IP
```

## Useful commands

```bash
# Show inventory details
ansible-inventory --list

# Run a one-off command
ansible web -m ansible.builtin.command -a "uname -a"

# Check playbook syntax
ansible-playbook --syntax-check playbooks/setup-webserver.yml

# Preview supported changes
ansible-playbook --check playbooks/setup-webserver.yml
```

## Notes

- Keep the inventory IP/DNS name and SSH key path specific to your environment.
- Amazon Linux uses `ec2-user` by default; change it if your instance was configured differently.
- The web page is written to the standard Nginx document root, `/usr/share/nginx/html`.
