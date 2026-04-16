# Feature Specification: Codebase Soundness Review

**Feature Branch**: `001-codebase-soundness-review`  
**Created**: 2026-04-16  
**Status**: Draft  
**Input**: Comprehensive codebase review for rhas-deploy

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Variable Consistency Across Roles (Priority: P1)

As a platform operator, I need all Ansible variables referenced in templates and tasks to be properly defined in defaults or inventory, so that playbook runs do not fail due to undefined variables.

**Why this priority**: Undefined or inconsistent variables cause immediate runtime failures that block all deployment operations.

**Independent Test**: Can be verified by scanning every Jinja2 variable reference and confirming it has a corresponding definition in defaults/main.yml, inventory, or a preceding set_fact task.

**Acceptance Scenarios**:

1. **Given** a Jinja2 variable reference `{{ var_name }}` in any template or task file, **When** the variable origin is traced, **Then** it MUST be defined in the role's defaults/main.yml, the inventory example, or set via set_fact in a preceding task within the same play.
2. **Given** a variable that appears in multiple roles' defaults (e.g. `cluster_ssh_key_dir`), **When** the default values are compared, **Then** they MUST be semantically equivalent (e.g. `$HOME/.ssh` and `~/.ssh` should be unified to a single form).
3. **Given** a `when` conditional in a task, **When** the referenced variable is traced, **Then** it MUST be guaranteed to exist at that point in the execution flow.

---

### User Story 2 - Hardcoded Values Match Configurable Variables (Priority: P1)

As a platform operator, I need hardcoded values in Kubernetes manifests to match the configurable variables in inventory and defaults, so that changing a variable in inventory actually takes effect across all resources.

**Why this priority**: Mismatches between hardcoded manifest values and configurable variables cause silent configuration drift that is difficult to debug.

**Independent Test**: Can be verified by comparing hardcoded namespace names and labels in static manifests against the corresponding Ansible variables.

**Acceptance Scenarios**:

1. **Given** namespace names in `manifests/platform/namespaces.yml` (e.g. `auto-platform`, `auto-builder`), **When** compared with the `platform_prefix` variable in inventory, **Then** the hardcoded prefix "auto-" MUST either be generated from the variable or documented as intentionally static.
2. **Given** references to `openshift-gitops` namespace in manifests and templates, **When** compared with the `gitops_namespace` variable, **Then** all references MUST use the variable rather than hardcoded strings.
3. **Given** ArgoCD project names in manifests (e.g. `rhas`), **When** compared with Ansible configuration, **Then** they MUST be consistent or parameterized.

---

### User Story 3 - Secure Credential Management (Priority: P1)

As a security reviewer, I need all credentials and secrets to be handled securely, so that sensitive data is not exposed in plain text or through insecure defaults.

**Why this priority**: Credential exposure is a critical security risk that can compromise entire infrastructure.

**Independent Test**: Can be verified by scanning for hardcoded passwords, plain-text secrets in templates, and insecure default configurations.

**Acceptance Scenarios**:

1. **Given** default password values in inventory examples (e.g. `default_admin_password`, `keycloak_db_password`), **When** evaluated for security, **Then** they MUST either be marked as required-to-override or use a secure generation mechanism.
2. **Given** environment variables in templates (e.g. `JUMPSTARTER_GRPC_INSECURE=1`), **When** evaluated for security, **Then** insecure flags MUST be documented with their risk and limited to development contexts.
3. **Given** the use of `ignore_errors` in tasks, **When** the suppressed error is security-relevant (e.g. certificate validation, authentication), **Then** the error suppression MUST be justified and documented.

---

### User Story 4 - Idempotent and Error-Resilient Playbooks (Priority: P2)

As a platform operator, I need all playbooks and tasks to be idempotent and handle errors appropriately, so that re-running a playbook does not cause failures or duplicate resources.

**Why this priority**: Non-idempotent tasks cause failures on re-runs and make recovery from partial failures difficult.

**Independent Test**: Can be verified by reviewing shell/command tasks for idempotency guards and checking error-handling patterns.

**Acceptance Scenarios**:

1. **Given** a shell or command task that creates a resource, **When** the task is run a second time, **Then** it MUST either skip (when resource exists) or succeed without side effects.
2. **Given** a task with `ignore_errors: true`, **When** the error condition is analyzed, **Then** the error MUST be either harmless (checked by a subsequent task) or properly handled with a rescue block.
3. **Given** a `pause` task with a fixed duration, **When** the purpose is analyzed, **Then** it SHOULD be replaced with a proper wait_for or until condition where possible.

---

### User Story 5 - Cross-Cloud Provider Consistency (Priority: P2)

As a platform operator, I need all three cloud providers (AWS, Azure, GCP) to be handled consistently in lifecycle operations, so that start/stop/destroy works reliably regardless of provider.

**Why this priority**: Inconsistent provider handling causes failures that only appear in specific cloud environments.

**Independent Test**: Can be verified by comparing the structure and completeness of provider-specific task files.

**Acceptance Scenarios**:

1. **Given** lifecycle operations (start, stop, destroy) for each provider, **When** the task structure is compared, **Then** all providers MUST handle the same set of operations with equivalent error checking and verification.
2. **Given** the Azure stop task with the FIXME comment about VM filtering, **When** the filter logic is evaluated, **Then** it MUST filter VMs by cluster name to avoid affecting unrelated VMs.
3. **Given** the GCP zone handling (hardcoded to zones a, b, f), **When** evaluated against actual GCP zone availability, **Then** the zone list MUST either be configurable or cover all zones used by OpenShift installer.

---

### User Story 6 - Correct Ansible Role Structure (Priority: P3)

As a contributor, I need all roles to follow a consistent structure with proper lifecycle hooks, so that the codebase is maintainable and predictable.

**Why this priority**: Structural consistency reduces onboarding time and prevents maintenance errors.

**Independent Test**: Can be verified by checking that each role has the expected file structure (main.yml, initial.yml, install.yml, uninstall.yml, etc.).

**Acceptance Scenarios**:

1. **Given** the role lifecycle pattern (initial, pre_install, install, post_install, uninstall), **When** each role is checked, **Then** all lifecycle files MUST exist and contain valid YAML.
2. **Given** the `install-operator` reusable role, **When** its template variables are checked, **Then** all required variables MUST be documented and have sensible defaults.
3. **Given** a role's uninstall tasks, **When** compared with its install tasks, **Then** uninstall MUST clean up all resources that install creates.

---

### Edge Cases

- What happens when `platform_prefix` is changed after initial deployment (does it cause orphaned namespaces)?
- How does the system handle a partially failed cluster bootstrap (can it resume)?
- What happens when cloud provider credentials expire mid-playbook?
- What happens when the lifecycle-cluster role is called without gather-cluster-facts having run first?
- What happens when GCP instances are in zones other than a, b, or f?

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Every Jinja2 variable reference (`{{ var_name }}`) in templates and tasks MUST have a corresponding definition in the role's defaults/main.yml, the inventory example, or a preceding set_fact task.
- **FR-002**: Variables shared across multiple roles (e.g. `cluster_ssh_key_dir`, group names) MUST have identical default values or be defined in a single shared location.
- **FR-003**: Hardcoded namespace names in static manifests MUST be documented as intentionally static or replaced with templated values that reference configurable variables.
- **FR-004**: All credential variables (passwords, API keys, tokens) MUST NOT have insecure default values in committed code; they MUST be required inputs or use secure generation.
- **FR-005**: Every `ignore_errors: true` usage MUST be justified -- the error condition MUST be either harmless or handled by a subsequent task.
- **FR-006**: Shell and command tasks MUST be idempotent -- they MUST include guards (creates, removes, or when conditions) to prevent duplicate operations.
- **FR-007**: All three cloud providers (AWS, Azure, GCP) MUST have equivalent lifecycle operation coverage (start, stop, destroy) with proper resource filtering by cluster name.
- **FR-008**: The Azure lifecycle tasks MUST filter VMs by cluster name or resource group specific to the cluster, not enumerate all VMs in the subscription.
- **FR-009**: The GCP zone handling MUST either be configurable or cover all availability zones that OpenShift installer may use in the selected region.
- **FR-010**: Every role MUST follow the established file structure pattern (main.yml dispatching to lifecycle files).
- **FR-011**: Uninstall tasks for each role MUST clean up all resources created during install.
- **FR-012**: The `lifecycle-cluster` role MUST either include `gather-cluster-facts` as a dependency or document that it must be run after `gather-cluster-facts`.
- **FR-013**: ArgoCD Application manifests MUST reference namespaces and project names consistently with the values configured in Ansible variables.
- **FR-014**: Kubernetes manifest YAML MUST be syntactically valid and use correct apiVersion values for the target OpenShift version.
- **FR-015**: Template files MUST NOT contain credentials or secrets in plain text; secrets MUST be injected via Ansible variables at deploy time.

### Key Entities

- **Playbook**: Top-level Ansible playbook (1_bootstrap_cluster.yml, 2_config_platform.yml, etc.) that orchestrates role execution.
- **Role**: Ansible role with defaults, tasks, and templates following the lifecycle pattern.
- **Manifest**: Static Kubernetes/OpenShift YAML resource definition deployed via GitOps (ArgoCD).
- **Template**: Jinja2 template that generates Kubernetes resources or configuration files from Ansible variables.
- **Inventory**: User-provided configuration that overrides role defaults for a specific deployment.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Zero undefined variable references exist across all templates and tasks (FR-001 fully satisfied).
- **SC-002**: All cross-role variable defaults are consistent -- zero mismatches between role defaults for shared variables (FR-002 fully satisfied).
- **SC-003**: All hardcoded values in static manifests are either parameterized or documented as intentionally static (FR-003 fully satisfied).
- **SC-004**: Zero insecure default credentials exist in committed code (FR-004 fully satisfied).
- **SC-005**: Every `ignore_errors` usage has a documented justification or is replaced with proper error handling (FR-005 fully satisfied).
- **SC-006**: All cloud provider lifecycle operations filter resources by cluster identity (FR-007, FR-008 fully satisfied).
- **SC-007**: A complete list of findings is produced, categorized by severity (critical, high, medium, low), with actionable remediation guidance for each.
