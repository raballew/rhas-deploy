# Review Tasks: Codebase Soundness Review

**Branch**: `001-codebase-soundness-review` | **Date**: 2026-04-16  
**Spec**: `specs/001-codebase-soundness-review/spec.md`  
**Plan**: `specs/001-codebase-soundness-review/plan.md`

## Task Summary

| Task | Description | FR Coverage | Priority | Status |
|------|-------------|-------------|----------|--------|
| T-001 | Variable reference integrity scan | FR-001 | P1 | pending |
| T-002 | Cross-role variable consistency check | FR-002 | P1 | pending |
| T-003 | Hardcoded namespace audit | FR-003, FR-013 | P1 | pending |
| T-004 | Credential and secrets security scan | FR-004, FR-015 | P1 | pending |
| T-005 | ignore_errors usage audit | FR-005 | P2 | pending |
| T-006 | Idempotency check for shell/command tasks | FR-006 | P2 | pending |
| T-007 | Cloud provider lifecycle parity check | FR-007 | P2 | pending |
| T-008 | Azure VM filtering verification | FR-008 | P2 | pending |
| T-009 | GCP zone coverage verification | FR-009 | P2 | pending |
| T-010 | Role structure compliance check | FR-010 | P3 | pending |
| T-011 | Uninstall completeness audit | FR-011 | P3 | pending |
| T-012 | lifecycle-cluster dependency verification | FR-012 | P2 | pending |
| T-013 | Kubernetes manifest syntax validation | FR-014 | P3 | pending |

## Task Details

### T-001: Variable Reference Integrity Scan (P1)
**Covers**: FR-001  
**Action**: For every `{{ var_name }}` reference in .j2 and .yml files (excluding .specify/ and specs/), trace the variable to a definition in role defaults, inventory, or set_fact. Report any undefined variables.  
**Verification**: Zero undefined variable references.

### T-002: Cross-Role Variable Consistency Check (P1)
**Covers**: FR-002  
**Action**: Identify variables defined in multiple roles' defaults/main.yml. Compare their default values. Flag any inconsistencies.  
**Known issue**: `cluster_ssh_key_dir` is `$HOME/.ssh` in create-cluster but `~/.ssh` in gather-cluster-facts.  
**Verification**: All shared variables have identical defaults or are documented as intentionally different.

### T-003: Hardcoded Namespace Audit (P1)
**Covers**: FR-003, FR-013  
**Action**: Compare namespace names in `manifests/platform/namespaces.yml` (hardcoded `auto-` prefix) against `platform_prefix` variable usage in Ansible. Check ArgoCD Application templates for hardcoded vs templated namespace references. Verify ArgoCD project name consistency.  
**Verification**: All hardcoded values are documented or parameterized.

### T-004: Credential and Secrets Security Scan (P1)
**Covers**: FR-004, FR-015  
**Action**: Scan inventory examples and role defaults for default passwords or API keys. Check template files for plaintext credential values. Review secret-generating templates for proper handling.  
**Verification**: Zero insecure default credentials in committed code.

### T-005: ignore_errors Usage Audit (P2)
**Covers**: FR-005  
**Action**: Find all `ignore_errors: true` or `ignore_errors: yes` occurrences. For each, determine whether the error is harmless, handled downstream, or masking a real issue.  
**Verification**: Every ignore_errors has a documented justification.

### T-006: Idempotency Check for Shell/Command Tasks (P2)
**Covers**: FR-006  
**Action**: Find all `shell:`, `command:`, and `raw:` tasks. Check each for idempotency guards (creates, removes, when conditions, changed_when).  
**Verification**: All shell/command tasks are idempotent or documented as intentionally non-idempotent.

### T-007: Cloud Provider Lifecycle Parity Check (P2)
**Covers**: FR-007  
**Action**: Compare the task files for start_aws.yml, start_azure.yml, start_gcp.yml and their stop counterparts. Verify equivalent error checking, verification steps, and async handling across all providers.  
**Verification**: All providers have equivalent lifecycle coverage.

### T-008: Azure VM Filtering Verification (P2)
**Covers**: FR-008  
**Action**: Review the Azure stop/start tasks. Verify whether VMs are filtered by cluster name or resource group. The FIXME comment in stop_azure.yml indicates this is a known issue.  
**Verification**: Azure tasks filter VMs by cluster identity.

### T-009: GCP Zone Coverage Verification (P2)
**Covers**: FR-009  
**Action**: Review GCP lifecycle tasks. The current implementation hardcodes zones a, b, f. Verify this covers all zones OpenShift installer may use. Check if the zone list should be configurable.  
**Verification**: GCP zone coverage is complete or configurable.

### T-010: Role Structure Compliance Check (P3)
**Covers**: FR-010  
**Action**: For each role, verify the presence of main.yml, defaults/main.yml, and lifecycle task files. Check that main.yml properly dispatches to lifecycle files.  
**Verification**: All roles follow the established pattern.

### T-011: Uninstall Completeness Audit (P3)
**Covers**: FR-011  
**Action**: For each role, compare install tasks with uninstall tasks. Verify that every resource created in install is cleaned up in uninstall.  
**Verification**: Uninstall is the inverse of install for every role.

### T-012: lifecycle-cluster Dependency Verification (P2)
**Covers**: FR-012  
**Action**: Check whether lifecycle-cluster role depends on variables set by gather-cluster-facts (e.g. cluster_fqn). Verify the dependency is declared or documented.  
**Verification**: Dependency is explicit and cannot be silently missed.

### T-013: Kubernetes Manifest Syntax Validation (P3)
**Covers**: FR-014  
**Action**: Validate all static YAML manifests in the manifests/ directory for correct Kubernetes resource syntax (valid apiVersion, kind, required metadata fields).  
**Verification**: All manifests pass basic schema validation.
