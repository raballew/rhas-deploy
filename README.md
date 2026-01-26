# Red Hat Automotive Suite

This repository provides Infrastructure as Code (IaC) for deploying the complete [Red Hat Automotive Suite](https://github.com/rhadp) (RHAS) — a comprehensive, cloud-native development environment specifically designed for automotive software development.

The platform automatically provisions OpenShift clusters across major cloud providers (AWS, Azure, or GCP) with hybrid ARM/x86 architecture support, and pre-configures an integrated suite of development tools to streamline the creation of automotive applications and the Red Hat In-Vehicle Operating System (RHIVOS).  

## Getting started

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

1. Visit [Red Hat Hybride Cloud Console](https://console.redhat.com/openshift/overview)
2. Log in with your Red Hat account
3. Select the OpenShift [cluster type](https://console.redhat.com/openshift/create)
4. Download the pull secret (JSON file)
5. Save to `inventory/pull-secret.txt`

#### Configure your deployment

```bash
cp ansible/inventory/main.yml.example ansible/inventory/main.yml
cp ansible/inventory/secrets.yml.example ansible/inventory/secrets.yml
```

Edit `inventory/main.yml` and `inventory/secrets.yml` and configure the deployment options.

### Deployment

#### Run the full end-to-end deployment

```bash
source venv/bin/activate
cd ansible

ansible-playbook -i inventory/ 1_bootstrap_cluster.yml
```

**Expected Duration:** 60-90 minutes

## Contributing

Fork the repository and submit a pull request.

## Development

A list of ideas, open issues etc related to the Red Hat Automotive Suite (RHAS) is [here](https://github.com/orgs/rhadp/projects/1).  

Also check the [Issues](https://github.com/rhadp/rhas-deploy/issues) section of the this repository.

## Disclaimer

This is not an officially supported Red Hat product.

## License

See [LICENSE](LICENSE) file for details.