# Implementation Plan: Codebase Soundness Review

**Branch**: `001-codebase-soundness-review` | **Date**: 2026-04-16 | **Spec**: `specs/001-codebase-soundness-review/spec.md`
**Input**: Feature specification from `/specs/001-codebase-soundness-review/spec.md`

## Summary

Audit the entire rhas-deploy codebase to verify soundness across six dimensions: variable consistency, hardcoded value alignment, credential security, idempotency, cross-cloud provider parity, and role structure compliance. The output is a categorized list of findings with severity ratings and remediation guidance. No new features are implemented; this is a verification-only exercise.

## Technical Context

**Language/Version**: Ansible (YAML), Jinja2 templates, Kubernetes/OpenShift manifests  
**Primary Dependencies**: kubernetes.core, azure.azcollection, google.cloud, amazon.aws Ansible collections  
**Storage**: N/A (IaC project, no application data storage)  
**Testing**: None (no automated tests, no ansible-lint config, no molecule tests detected)  
**Target Platform**: OpenShift clusters on AWS, Azure, GCP  
**Project Type**: Infrastructure as Code (IaC) deployment automation  
**Performance Goals**: N/A (review task, not runtime)  
**Constraints**: Review must be completable without access to a live cluster  
**Scale/Scope**: ~90 YAML/Jinja2 files across 10 roles, 6 playbooks, and 5 manifest directories

## Constitution Check

No constitution file detected. No gates to check.

## Project Structure

### Documentation (this feature)

```text
specs/001-codebase-soundness-review/
  spec.md              # Feature specification (completed)
  plan.md              # This file
  tasks.md             # Review task breakdown
```

### Source Code (repository root)

```text
ansible/
  1_bootstrap_cluster.yml     # Cluster creation playbook
  2_config_platform.yml       # Platform configuration playbook
  2_config_metal.yml          # Bare metal configuration playbook
  9_destroy_cluster.yml       # Cluster teardown playbook
  start.yml                   # Cluster start lifecycle playbook
  stop.yml                    # Cluster stop lifecycle playbook
  roles/
    create-cluster/           # OCP cluster provisioning (AWS/Azure/GCP)
    gather-cluster-facts/     # Collect cluster metadata post-install
    install-operator/         # Reusable OLM operator installer
    lifecycle-cluster/        # Start/stop operations per cloud provider
    setup-auth/               # Keycloak + RBAC configuration
    setup-cert-manager/       # Certificate manager operator setup
    setup-gitops/             # ArgoCD/GitOps configuration
    setup-metal/              # Bare metal node provisioning
    setup-platform/           # Platform component deployment via GitOps
    setup-registry/           # Container registry configuration

manifests/
  platform/                   # Namespace definitions (static)
  builder/                    # Tekton pipeline resources
  devspaces/                  # DevSpaces CheCluster config
  jumpstarter/                # Jumpstarter controller resources
  jumpstarter-qemu/           # Jumpstarter QEMU exporter resources

add-ons/                      # Optional add-on applications
```

**Structure Decision**: Existing repository structure. No changes planned -- this is a review-only exercise.

## Review Approach

### Stage 1: Variable Consistency and Reference Integrity (FR-001, FR-002)
- Scan all .j2 and .yml files for `{{ variable }}` references
- Cross-reference each variable against role defaults and inventory example
- Flag undefined variables, inconsistent defaults, and typos
- Known issue to verify: `cluster_ssh_key_dir` defaults differ between create-cluster (`$HOME/.ssh`) and gather-cluster-facts (`~/.ssh`)

### Stage 2: Hardcoded Namespace and Configuration Drift (FR-003, FR-013)
- Compare hardcoded values in `manifests/platform/namespaces.yml` against configurable variables
- Check ArgoCD Application templates for hardcoded vs parameterized namespace references
- Verify ArgoCD project names are consistent

### Stage 3: Security Review (FR-004, FR-015)
- Scan for hardcoded passwords and credentials in defaults and templates
- Review `ignore_errors` usage for security implications
- Check for insecure protocol flags (e.g. `JUMPSTARTER_GRPC_INSECURE`)
- Review secret templates for proper data handling

### Stage 4: Idempotency and Error Handling (FR-005, FR-006)
- Review all shell/command tasks for idempotency guards
- Audit `ignore_errors` usage patterns
- Check pause tasks for proper wait conditions
- Verify rescue/always blocks where appropriate

### Stage 5: Cross-Cloud Provider Consistency (FR-007, FR-008, FR-009, FR-012)
- Compare AWS/Azure/GCP lifecycle task files for structural parity
- Verify Azure VM filtering (known FIXME)
- Verify GCP zone coverage
- Check lifecycle-cluster dependency on gather-cluster-facts

## Complexity Tracking

No complexity violations. This is a review task with no new code to implement.
