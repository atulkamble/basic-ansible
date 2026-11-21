# Basic Ansible Configuration

A beginner-friendly Ansible project for managing **Amazon Linux 2 and Amazon Linux 2023** EC2 instances. It demonstrates how Ansible reads project configuration, connects to hosts from an inventory, uses shared variables, and runs playbooks to check connectivity and configure Nginx.

## Project structure

```text
basic-ansible/
├── .gitignore
├── LICENSE
├── README.md
├── ansible.cfg
├── group_vars/
│   └── all.yml
├── inventory/
│   └── hosts.ini
└── playbooks/
    ├── ping.yml
    └── setup-webserver.yml
```

## What each file does

| File | Significance and use |
|---|---|
| `README.md` | Project guide. It explains prerequisites, SSH setup, commands, and how to run the examples. Start here when using the repository. |
| `ansible.cfg` | Project-level Ansible settings. It selects the inventory, default SSH user and key path, and privilege escalation settings. Run Ansible from this repository directory so Ansible finds this file. |
| `.gitignore` | Prevents private key files and generated Ansible retry files from being added to Git. It helps reduce accidental credential exposure; it does not replace secure handling of secrets. |
| `LICENSE` | States the terms under which others may use, copy, or modify the project. |
| `inventory/hosts.ini` | Lists the managed EC2 hosts and groups them under `[web]`. Replace the sample IP/DNS name with your instance. Inventory variables can also set connection details for hosts. |
| `group_vars/all.yml` | Defines shared variables for every inventory group, such as the Nginx package, service, document root, and page title. Playbooks use these values so common settings are kept in one place. |
| `playbooks/ping.yml` | A safe connectivity example. It uses Ansible's ping module to check that Ansible can connect over SSH and run Python on each target. |
| `playbooks/setup-webserver.yml` | A configuration example for the `web` group. It installs Nginx, enables and starts the service, and deploys a sample home page. It uses privilege escalation because package installation and writing the web root require administrator permissions. |

### How the files work together

Ansible reads `ansible.cfg` for defaults, loads hosts from `inventory/hosts.ini`, makes variables from `group_vars/all.yml` available to playbooks, and executes the selected YAML file in `playbooks/`. The inventory group determines which machines a playbook targets.

## Requirements

- Ansible installed on your control machine (Linux, macOS, or WSL).
- A reachable Amazon Linux 2 or Amazon Linux 2023 EC2 instance.
- SSH access and the matching EC2 private key.
- The EC2 security group must allow SSH (TCP 22) from your control machine.

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

1. In `inventory/hosts.ini`, replace `YOUR_SERVER_PUBLIC_IP` with your EC2 instance's public IP address or DNS name.
2. The sample uses Amazon Linux's default SSH user, `ec2-user`.
3. Create the local key directory and place your private key there:

```bash
mkdir -p aws
cp /path/to/your-key.pem aws/ansible.pem
chmod 400 aws/ansible.pem
```

Alternatively, change `private_key_file` in `ansible.cfg` to your key's location. Never commit private keys, passwords, or cloud credentials.

## Verify connectivity

Run these commands from the repository root:

```bash
ansible --version
ansible-inventory --graph
ansible all -m ansible.builtin.ping
ansible-playbook playbooks/ping.yml
```

A successful result includes `"ping": "pong"`. The Ansible ping module verifies SSH and Python connectivity; it is not an ICMP network ping.

## Install and start Nginx

The web server playbook uses Ansible's OS-aware package module and is written for Amazon Linux 2 and Amazon Linux 2023.

```bash
ansible-playbook playbooks/setup-webserver.yml --limit web
```

It installs Nginx, enables and starts the service, and writes a sample page to `/usr/share/nginx/html/index.html`. Allow inbound HTTP (TCP 80) in the EC2 security group and visit:

```text
http://YOUR_SERVER_PUBLIC_IP
```

## Useful commands

```bash
# Show inventory details
ansible-inventory --list

# Run a one-off command on the web group
ansible web -m ansible.builtin.command -a "uname -a"

# Check playbook syntax without contacting the host
ansible-playbook --syntax-check playbooks/setup-webserver.yml

# Preview supported changes
ansible-playbook --check playbooks/setup-webserver.yml
```

## Notes

- Keep the inventory host and SSH key path specific to your environment.
- If your EC2 instance uses a different SSH username, update `remote_user` in `ansible.cfg` or `ansible_user` in the inventory.
- Review the target hosts before running a playbook. The Nginx playbook changes the remote system.
