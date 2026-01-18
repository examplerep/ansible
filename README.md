# Example Power - Ansible Automation

Ansible playbooks for managing Example Power infrastructure.

## Prerequisites

- Python 3.9+
- awscli

## Setup

```shell
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
ansible-galaxy collection install -r collections/requirements.yml
```

## Configuration

### AWS Credentials

AWS credentials are read automatically from `~/.aws/credentials`. Configure using the AWS CLI:

```shell
aws configure
```

You will be prompted for:

```text
AWS Access Key ID [None]: your-access-key
AWS Secret Access Key [None]: your-secret-key
Default region name [None]: us-east-2
Default output format [None]: json
```

This creates `~/.aws/credentials` and `~/.aws/config` automatically.

### Cloudflare API Token

Store your Cloudflare API token in `~/.cloudflare_api_token`:

```shell
echo "your-cloudflare-api-token" > ~/.cloudflare_api_token
chmod 600 ~/.cloudflare_api_token
```

### GitHub Personal Access Token

Store your GitHub PAT in `~/.github_pat`:

```shell
echo "your-github-pat" > ~/.github_pat
chmod 600 ~/.github_pat
```

### Environment Variables

The following environment variables must be set before running playbooks:

| Variable   | Description                |
| ---------- | -------------------------- |
| SANDBOX_ID | OpenTLC sandbox identifier |

```shell
export SANDBOX_ID="1234"
```

## Playbooks

### DNS Setup

Creates a Route53 hosted zone for `ocp.examplerep.com` and configures Cloudflare NS delegation.

```shell
source .venv/bin/activate
ansible-playbook playbooks/setup-dns.yml
```

**What it does:**

1. Creates Route53 hosted zone for `ocp.examplerep.com`
2. Creates NS records in Cloudflare pointing `ocp.examplerep.com` to Route53 name servers

## Directory Structure

```text
.venv/                       # Python virtual environment
.gitignore                   # Git ignore file
ansible.cfg                  # Ansible configuration
collections/
└── requirements.yml         # Required Ansible collections
inventory/
├── group_vars/
│   └── all.yml              # Global variables
└── hosts.yml                # Inventory configuration
playbooks/
└── setup-dns.yml            # DNS setup playbook
requirements.txt             # Python dependencies
README.md                    # This file
```

## Domain Architecture

```text
┌─────────────────────────────────────────────────────────────┐
│                      Cloudflare DNS                         │
│                    examplerep.com                           │
│                                                             │
│  ocp.examplerep.com  NS ──┐                                 │
└─────────────────────────────│───────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                      AWS Route53                            │
│                   ocp.examplerep.com                        │
│                                                             │
│  *.apps.ocp.examplerep.com  → OpenShift Ingress             │
│  api.ocp.examplerep.com     → OpenShift API                 │
└─────────────────────────────────────────────────────────────┘
```
