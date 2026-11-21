# Basic Ansible Configuration

A beginner-friendly Ansible project for connecting to Linux servers, checking connectivity, and installing a simple Nginx web server. The examples target **Amazon Linux 2023**.

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

- Ansible installed on the control machine (Linux, macOS, or WSL).
- One reachable Amazon Linux 2023 server.
- SSH access to the server and a private key.

Install Ansible on Ubuntu/Debian:

```bash
sudo apt update
sudo apt install ansible -y
```

On macOS with Homebrew:

```bash
brew install ansible
```

## Configure the inventory

Edit `inventory/hosts.ini` and replace `YOUR_SERVER_PUBLIC_IP` with the server's public IP address. The sample uses the default Amazon Linux user `ec2-user`.

Place your SSH private key at `aws/ansible.pem`, or change `private_key_file` in `ansible.cfg` to the correct path. Set restrictive permissions:

```bash
chmod 400 aws/ansible.pem
```

The private key is excluded from Git. Never commit private keys, passwords, or cloud credentials.

## Check the configuration and connectivity

Run these commands from the repository root:

```bash
ansible --version
ansible-inventory --graph
ansible all -m ansible.builtin.ping
ansible-playbook playbooks/ping.yml
```

A successful ping module result includes `"ping": "pong"`. This confirms Ansible can connect over SSH; it is not an ICMP network ping.

## Install and start Nginx

Review the target hosts, then run:

```bash
ansible-playbook playbooks/setup-webserver.yml --limit web
```

The playbook installs Nginx, enables and starts the service, and writes a small page. Open `http://YOUR_SERVER_PUBLIC_IP` to view it. Ensure the server security group/firewall allows inbound TCP port 80.

## Useful commands

```bash
# Show inventory details
ansible-inventory --list

# Run a one-off command
ansible web -m ansible.builtin.command -a "uname -a"

# Check playbook syntax
ansible-playbook --syntax-check playbooks/setup-webserver.yml

# Preview supported changes (where modules support check mode)
ansible-playbook --check playbooks/setup-webserver.yml
```

## Notes

- Keep the inventory and SSH key path specific to your environment.
- The web server example uses Amazon Linux 2023's Nginx package and document root.
- SSH access (usually TCP port 22) must be allowed from the control machine.
