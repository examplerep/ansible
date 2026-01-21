# Example Power - Ansible Automation

Ansible playbooks for managing Example Power infrastructure.

## Setup

```shell
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
ansible-galaxy collection install -r collections.yml
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

### OpenShift Pull Secret

Download your pull secret from [Red Hat Console](https://console.redhat.com/openshift/install/pull-secret) and store it:

```shell
# Save pull secret to ~/.pull-secret
chmod 600 ~/.pull-secret
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

### Setup RHDP Environment

Sets up the RHDP (Red Hat Demo Platform) environment including DNS and AWS Secrets Manager.

```shell
source .venv/bin/activate
ansible-playbook playbooks/setup-rhdp-environment.yml
```

**What it does:**

1. Creates Route53 hosted zone for `ocp.examplerep.com`
2. Creates NS records in Cloudflare pointing `ocp.examplerep.com` to Route53 name servers
3. Creates AWS Secrets Manager secrets for cluster use

### Create Hub Cluster

Creates a Single Node OpenShift (SNO) cluster for the hub at `hub.ocp.examplerep.com`.

```shell
source .venv/bin/activate
rm -rf .clusters/hub
ansible-playbook playbooks/create-hub-cluster.yml
```

**What it does:**

1. Generates install-config.yaml for SNO
2. Runs openshift-install to create the cluster
3. Outputs cluster access credentials

**Cluster Details:**

| Property      | Value                       |
| ------------- | --------------------------- |
| Cluster Name  | hub                         |
| Domain        | hub.ocp.examplerep.com      |
| Type          | Single Node OpenShift (SNO) |
| Architecture  | amd64                       |
| Instance Type | m6i.4xlarge                 |

### Create Spoke Cluster

Creates a Single Node OpenShift (SNO) spoke cluster (dev, test, or prod).

```shell
source .venv/bin/activate
ansible-playbook playbooks/create-spoke-cluster.yml -e cluster_name=dev
ansible-playbook playbooks/create-spoke-cluster.yml -e cluster_name=test
ansible-playbook playbooks/create-spoke-cluster.yml -e cluster_name=prod
```

**What it does:**

1. Generates install-config.yaml for SNO
2. Runs openshift-install to create the cluster
3. Outputs cluster access credentials

**Cluster Details:**

| Property      | Value                            |
| ------------- | -------------------------------- |
| Cluster Name  | dev / test / prod                |
| Domain        | {name}.ocp.examplerep.com        |
| Type          | Single Node OpenShift (SNO)      |
| Architecture  | amd64                            |
| Instance Type | m6i.4xlarge                      |

## Directory Structure

```text
.clusters/                      # Cluster installation files (gitignored)
.gitignore                      # Git ignore file
.venv/                          # Python virtual environment
ansible.cfg                     # Ansible configuration
collections.yml                 # Required Ansible collections
inventory/
├── group_vars/
│   └── all.yml                 # Global variables
└── hosts.yml                   # Inventory configuration
playbooks/
├── create-hub-cluster.yml      # Hub cluster creation playbook
├── create-spoke-cluster.yml    # Spoke cluster creation (dev/test/prod)
└── setup-rhdp-environment.yml  # RHDP environment setup (DNS, Secrets Manager)
README.md                       # This file
requirements.txt                # Python dependencies
templates/
└── install-config-sno.yaml.j2  # SNO install config template
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
