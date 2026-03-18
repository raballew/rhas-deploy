# Red Hat Automotive Suite

> **One-command deployment of a complete automotive development platform**

RHAS provides Infrastructure as Code (IaC) for deploying the complete [Red Hat Automotive Suite](https://github.com/rhadp) — a cloud-native development environment purpose-built for automotive software development.

## What You Get

- **OpenShift Cluster** – Multi-cloud deployment (AWS, Azure, GCP) with hybrid ARM/x86 support
- **OpenShift Dev Spaces** – Browser-based IDEs with pre-configured automotive toolchains
- **CI/CD Pipeline** – Automated build system for automotive images (AutoSD/RHIVOS)
- **Hardware-in-the-Loop Testing** – Jumpstarter integration for real device testing
- **GitOps Ready** – ArgoCD pre-configured for declarative deployments
- **SSO & RBAC** – Keycloak-based authentication out of the box

**Deploy in ~60 minutes** with a single command.

## Getting started

### Prerequisites

- Cloud credentials (AWS/Azure/GCP)
- Red Hat account, OpenShift pull secret
- Python 3.9+ and Ansible

### Preparation

#### Clone the repository

```bash
git clone https://github.com/rhadp/rhas-deploy.git
cd rhas-deploy
```

Fork the repository if you plan to customize playbooks, roles, or configuration templates.

#### Setup the Python environment

```bash
# Create and activate virtual environment
python3 -m venv venv
source venv/bin/activate  # macOS/Linux

# Install dependencies
pip install -r requirements.txt

# Install Azure collection (if deploying to Azure)
ansible-galaxy collection install azure.azcollection --force
pip install -r ~/.ansible/collections/ansible_collections/azure/azcollection/requirements.txt
```

#### Obtain a Red Hat OpenShift pull-secret

1. Visit [Red Hat Hybrid Cloud Console](https://console.redhat.com/openshift/overview)
2. Log in with your Red Hat account
3. Select the OpenShift [cluster type](https://console.redhat.com/openshift/create)
4. Download the pull secret (JSON file)
5. Save to `ansible/inventory/pull-secret.txt`

### Deployment

#### Configure your deployment

```bash
cp ansible/inventory/main.yml.example ansible/inventory/main.yml
```

Edit `inventory/main.yml` to configure the deployment options.

#### Run the full end-to-end deployment

```bash
source venv/bin/activate
cd ansible

ansible-playbook -i inventory/ 1_bootstrap_cluster.yml
```

**Expected Duration:** 60-90 minutes

Once complete, access your platform:
- **OpenShift Console**: `https://console-openshift-console.apps.<cluster_name>.<domain>`
- **Dev Spaces**: Pre-configured workspaces for automotive development
- **ArgoCD**: Manage your GitOps deployments


## Managing Your Cluster

### Start/Stop

```bash
cd ansible

# Stop cluster (preserves state, reduces cloud costs)
ansible-playbook -i inventory/ stop.yml

# Start cluster
ansible-playbook -i inventory/ start.yml
```

### Destroy

```bash
cd ansible
ansible-playbook -i inventory/ 9_destroy_cluster.yml
```

## Contributing

Contributions welcome! Fork the repository and submit a pull request. Also check the [Issues](https://github.com/rhadp/rhas-deploy/issues) section of the this repository.

See the [project board](https://github.com/orgs/rhadp/projects/1) for planned features and open issues.

### Related Repositories

- [rhadp/containers](https://github.com/rhadp/containers) - Container images for the platform
- [jumpstarter-dev/jumpstarter](https://github.com/jumpstarter-dev/jumpstarter) - Automated testing on real and virtual hardware with CI/CD integration
- [AutoSD - Automotive Stream Distribution](https://sigs.centos.org/automotive/index.html) - AutoSD is the upstream binary distribution that serves as the public, in-development preview of Red Hat In-Vehicle Operating System (RHIVOS)
- [CentOS/automotive](https://gitlab.com/CentOS/automotive) - AutoSD code etc.


## Disclaimer

This is not an officially supported Red Hat product.

## License

See [LICENSE](LICENSE) file for details.