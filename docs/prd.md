# Product Requirements Document

| Field   | Value                                       |
|---------|---------------------------------------------|
| Product | Red Hat Automotive Suite — Deployment Tool  |
| Version | 1.0                                         |
| Date    | 2026-02-23                                  |
| Status  | Reverse-engineered from codebase            |

## 1. Product Overview

RHAS Deploy is an infrastructure-as-code tool that provisions a complete cloud-native automotive software development platform in a single command. It targets automotive engineering teams who need a ready-to-use environment — including browser-based IDEs, automated image-build pipelines, hardware-in-the-loop testing, and GitOps-based deployment management — running on a managed cloud cluster across AWS, Azure, or GCP. The tool handles the full lifecycle: initial provisioning, platform configuration, cost-saving pause/resume, and teardown.

## 2. Goals & Non-Goals

**Goals**

- ✅ Enable an operator to deploy a fully configured automotive development platform in under 90 minutes with a single command
- ✅ Support multi-cloud deployment (AWS, Azure, GCP), including mixed-architecture clusters (ARM and x86)
- ✅ Provide browser-based development workspaces pre-configured with automotive toolchains
- ✅ Automate building and publishing automotive operating system images (AutoSD/RHIVOS)
- ✅ Enable automated testing on real and virtual hardware via a hardware-in-the-loop testing framework
- ✅ Deliver enterprise-grade authentication (SSO via identity provider) and role-based access control out of the box
- ✅ Manage all platform components declaratively through GitOps
- ✅ Allow operators to stop and start clusters to control cloud costs

**Non-Goals**

- 🚫 Application-level development — the tool provisions the platform, not the automotive software itself
- 🚫 On-premises or private-cloud deployment — only the three major public clouds are supported
- 🚫 Multi-cluster management — each deployment manages a single cluster
- 🚫 End-user self-service provisioning — an operator with cloud credentials performs deployment

## 3. User Personas

| Persona            | Description                                                                                      | Primary Need                                                                                   |
|--------------------|--------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------|
| Platform Operator  | An infrastructure or DevOps engineer responsible for provisioning and maintaining the platform    | Deploy, update, and tear down the full platform reliably with minimal manual steps              |
| Platform Admin     | A team lead or engineering manager who oversees developer access and platform configuration       | Control who can access the platform, manage groups and permissions, and monitor platform health |
| Automotive Developer | A software engineer building automotive OS images, drivers, or in-vehicle applications          | Access a browser-based workspace with pre-configured toolchains and an automated build pipeline |
| Test Engineer      | An engineer responsible for validating automotive images against real or emulated hardware        | Run automated tests on target hardware or virtual devices through CI/CD integration             |

## 4. User Workflows

**Workflow: Initial Platform Deployment**

1. Operator clones the project repository and sets up a local environment with the required dependencies.
2. Operator obtains cloud provider credentials (AWS, Azure, or GCP) and a Red Hat pull secret.
3. Operator copies the provided configuration templates and edits them, selecting the cloud provider, cluster architecture, cluster name, domain, and which platform features to enable.
4. Operator copies the provided secrets template and fills in cloud credentials, authentication details, and certificate email.
5. Operator runs the bootstrap command.
6. The system provisions cloud infrastructure (networking, compute instances, storage), installs the container platform, configures TLS certificates via Let's Encrypt, sets up SSO authentication, installs GitOps and CI/CD operators, and deploys the selected platform components.
7. Upon completion (approximately 60–90 minutes), the system displays the web console URL.
8. Outcome: The operator has a fully operational automotive development platform accessible via browser.

**Workflow: Developer Onboarding**

1. Platform Admin adds the developer to the appropriate access group (platform-users, platform-admins, or jumpstarter-users).
2. Developer navigates to the web console URL and signs in through the SSO identity provider or a local account.
3. Developer opens the browser-based IDE (Dev Spaces), which provisions a personal workspace with pre-configured automotive development tools.
4. Outcome: The developer has a ready-to-use workspace with access to source control, build pipelines, and testing infrastructure.

**Workflow: Automated Image Build**

1. Developer pushes code to a monitored source repository.
2. The system detects the push event via webhook and triggers an automated build pipeline.
3. The pipeline clones the source, prepares the build manifest, builds the automotive OS image, and uploads the resulting artifact to cloud storage.
4. Outcome: A new automotive OS image is built and published without manual intervention.

**Workflow: Hardware-in-the-Loop Testing**

1. Test Engineer triggers a test run targeting a hardware device (physical or emulated).
2. The system provisions a virtual or physical test device, flashes the image, and executes the test suite.
3. Test results are reported back through the CI/CD pipeline.
4. Outcome: The image is validated against real or emulated target hardware.

**Workflow: Cost Management (Stop/Start)**

1. Operator runs the stop command.
2. The system shuts down all cloud compute instances while preserving cluster state.
3. When work resumes, the operator runs the start command.
4. The system restarts all compute instances and waits for the cluster to stabilize.
5. The system displays the web console URL.
6. Outcome: Cloud costs are reduced during periods of inactivity without losing cluster configuration.

**Workflow: Platform Update**

1. Operator updates platform configuration or manifests in the GitOps repository.
2. Operator runs the platform configuration command against the existing cluster.
3. The system reconciles the desired state through GitOps, deploying or updating platform components as needed.
4. Outcome: Platform components are updated without reprovisioning the cluster.

**Workflow: Cluster Teardown**

1. Operator runs the destroy command.
2. The system removes all cloud infrastructure (compute, networking, storage, DNS) and cleans up local metadata and credentials.
3. Outcome: All cloud resources are released; no residual costs accrue.

## 5. Functional Requirements

**Capability Area: Cluster Provisioning**

- [REQ-001] The system shall provision a container platform cluster on the operator's choice of AWS, Azure, or GCP.
- [REQ-002] The system shall support three cluster architectures: x86-only, ARM-only, and mixed ARM/x86 (mixed architecture supported on AWS and Azure only).
- [REQ-003] When provisioning a cluster, the system shall create three control-plane nodes distributed across availability zones and three worker nodes distributed across availability zones.
- [REQ-004] The system shall support two cluster topologies: a standard topology with separate control-plane and worker nodes, and a compact topology where control-plane nodes also serve as workers.
- [REQ-005] When cluster installation completes, the system shall label worker nodes with designated roles (infrastructure, builder) to control workload scheduling.
- [REQ-006] The system shall generate and manage SSH key pairs for cluster node access automatically.

**Capability Area: Bare-Metal and Virtualization (AWS Only)**

- [REQ-007] Where bare-metal worker node support is enabled, the system shall provision high-performance ARM bare-metal instances and label them for specialized workloads.
- [REQ-008] Where virtualization support is enabled (requires bare-metal), the system shall install a virtualization layer that allows virtual machines to run on bare-metal nodes.

**Capability Area: TLS Certificate Management**

- [REQ-009] The system shall install a certificate management operator and configure it to obtain TLS certificates from Let's Encrypt automatically.
- [REQ-010] The system shall provision a wildcard TLS certificate for all platform-hosted applications.
- [REQ-011] Certificates shall be configured with a 90-day validity period and automatic renewal one week before expiry.
- [REQ-012] The system shall configure the cloud provider's DNS service (Route 53, Azure DNS, or Cloud DNS) as the certificate validation method.

**Capability Area: Authentication and Access Control**

- [REQ-013] The system shall deploy an SSO identity provider (Keycloak) with a backing database for centralized authentication.
- [REQ-014] The system shall configure the platform to accept sign-in from both the SSO identity provider and a local password-based provider.
- [REQ-015] The system shall create five access groups: cluster administrators, platform administrators, platform users, testing-framework administrators, and testing-framework users.
- [REQ-016] The system shall assign the following permissions:
  - Cluster administrators receive full cluster control.
  - Platform administrators inherit all platform-user permissions.
  - Platform users receive basic access, cluster status visibility, and the ability to create projects.
- [REQ-017] Where configured, the system shall remove the default emergency-access administrator account after authentication setup completes.
- [REQ-018] The system shall add a direct link to the identity provider administration console from the platform web console.

**Capability Area: GitOps and CI/CD**

- [REQ-019] The system shall install a GitOps operator (ArgoCD) for declarative, Git-driven management of all platform components.
- [REQ-020] The system shall install a CI/CD pipelines operator (Tekton) for automated build and test workflows.
- [REQ-021] All platform components shall be deployed and managed as GitOps applications, synced from a configurable Git repository and branch.

**Capability Area: Developer Workspaces**

- [REQ-022] Where developer workspaces are enabled, the system shall deploy browser-based IDE workspaces (Eclipse Che / Dev Spaces) accessible to platform users.
- [REQ-023] Each user shall be limited to one running workspace at a time.
- [REQ-024] Idle workspaces shall be automatically stopped after 30 minutes of inactivity.
- [REQ-025] Each user workspace shall receive 10 GB of persistent storage.
- [REQ-026] Workspaces shall be scheduled on builder-role nodes only.
- [REQ-027] Workspace access shall be restricted to members of the platform-admins and platform-users groups.

**Capability Area: Automated Image Builder**

- [REQ-028] Where the builder is enabled, the system shall deploy an automated image-build pipeline triggered by source-code push events.
- [REQ-029] When a push event is received, the system shall clone the repository, determine the build configuration, execute the image build, and upload the resulting artifact to cloud storage.
- [REQ-030] The build pipeline shall expose a webhook endpoint for integration with source control hosting services.

**Capability Area: Hardware-in-the-Loop Testing**

- [REQ-031] Where the testing framework is enabled, the system shall deploy a hardware testing controller that manages connections to physical and emulated devices.
- [REQ-032] The system shall support emulated ARM devices using QEMU virtualization on bare-metal nodes, providing console access, image flashing, and power control.
- [REQ-033] Emulated devices shall be configured with 1 CPU and 1 GB RAM by default.
- [REQ-034] The testing framework shall integrate with the CI/CD pipeline so that test tasks can be included in automated build workflows.

**Capability Area: Cluster Lifecycle Management**

- [REQ-035] The system shall allow the operator to stop all cluster compute instances to reduce cloud costs while preserving cluster state.
- [REQ-036] The system shall allow the operator to restart a previously stopped cluster.
- [REQ-037] After restarting, the system shall wait for the cluster to stabilize before reporting completion.
- [REQ-038] The system shall allow the operator to permanently destroy the cluster and all associated cloud infrastructure.

**Capability Area: Web Console Customization**

- [REQ-039] The system shall enable the developer perspective in the platform web console for platform users.
- [REQ-040] The system shall restrict the administrator perspective to cluster administrators only.

## 6. Configuration & Input Specification

### Cluster Configuration

| Option                        | Description                                                                 | Default         | Valid Values                          |
|-------------------------------|-----------------------------------------------------------------------------|-----------------|---------------------------------------|
| Cloud provider                | Target cloud platform for deployment                                        | —               | AWS, Azure, GCP                       |
| Cluster version               | Version of the container platform to install                                | —               | Valid OpenShift version (e.g., 4.20)  |
| Installer platform            | Operating system of the operator's machine                                  | —               | macOS, macOS ARM, Linux, Linux ARM    |
| Cluster architecture          | Processor architecture for the cluster                                      | —               | x86-only, ARM-only, Mixed             |
| Control-plane architecture    | Processor architecture for control-plane nodes (mixed-arch clusters)        | —               | AMD64, ARM64                          |
| Cluster topology              | Node layout                                                                 | Standard        | Standard, Compact                     |
| Cluster name                  | Unique identifier for the cluster                                           | —               | Alphanumeric string                   |
| Cluster subdomain             | DNS subdomain for platform applications                                     | —               | Valid domain name                     |
| Cluster top-level domain      | Top-level DNS domain                                                        | —               | Valid domain name                     |
| Default storage class         | Cloud-provider storage backend for persistent volumes                       | Provider default| Cloud-specific storage class name     |

### Feature Toggles

| Option                           | Description                                                              | Default  | Valid Values |
|----------------------------------|--------------------------------------------------------------------------|----------|--------------|
| Deploy developer workspaces      | Install browser-based IDE workspaces                                     | Enabled  | Yes / No     |
| Deploy developer hub             | Install developer portal                                                 | Enabled  | Yes / No     |
| Deploy image builder             | Install automated image build pipeline                                   | Enabled  | Yes / No     |
| Deploy testing framework         | Install hardware-in-the-loop testing controller                          | Enabled  | Yes / No     |
| Setup ARM worker nodes           | Add secondary ARM-architecture worker nodes                              | Disabled | Yes / No     |
| Setup bare-metal worker node     | Add a high-performance bare-metal ARM node (AWS only)                    | Disabled | Yes / No     |
| Setup virtualization             | Enable VM scheduling on bare-metal nodes (AWS only, requires bare-metal) | Disabled | Yes / No     |
| Remove default admin account     | Remove emergency-access admin after auth setup                           | Disabled | Yes / No     |

### GitOps Configuration

| Option                   | Description                                              | Default                              | Valid Values            |
|--------------------------|----------------------------------------------------------|--------------------------------------|-------------------------|
| GitOps repository URL    | Git repository containing platform manifests             | Project's own repository URL         | Valid Git URL           |
| GitOps repository branch | Branch or tag to sync from                               | main                                 | Valid Git ref           |

### Credentials (Secrets)

| Option                        | Description                                         |
|-------------------------------|-----------------------------------------------------|
| Cloud provider credentials    | Access keys / service account for the target cloud  |
| Red Hat pull secret            | Authentication token for Red Hat container registry |
| Let's Encrypt email           | Contact email for TLS certificate registration      |
| Default admin password        | Password for the initial platform admin account     |
| SSO database password         | Password for the identity provider's database       |
| SSO client secret             | OAuth client secret for SSO integration             |
| Source control OAuth credentials | OAuth app credentials for source control integration|
| Source control API token      | Personal access token for source control API access |
| Build webhook secret          | Shared secret for authenticating build webhooks     |

## 7. Output Specification

| Output                    | Description                                                                                                                 |
|---------------------------|-----------------------------------------------------------------------------------------------------------------------------|
| Running cluster           | A fully provisioned container platform cluster accessible via web console at `https://console-openshift-console.apps.<cluster_name>.<domain>` |
| Platform web console      | Browser-accessible management interface with developer and administrator perspectives                                       |
| Developer workspaces      | Browser-based IDEs accessible to authorized users                                                                           |
| GitOps dashboard          | ArgoCD interface for viewing and managing declarative deployments                                                            |
| SSO admin console         | Identity provider administration interface linked from the platform web console                                              |
| Build pipeline webhook    | HTTP endpoint that accepts push events from source control to trigger automated builds                                      |
| Built artifacts           | Automotive OS images uploaded to cloud object storage upon successful pipeline runs                                          |
| Local credentials         | Cluster access configuration file and SSH keys stored on the operator's machine                                             |

## 8. Error Handling & User Feedback

- **Invalid or missing cloud credentials**: The provisioning process fails during the infrastructure creation phase. The operator sees an error from the cloud provider indicating authentication or authorization failure.
- **Cluster installation timeout**: If the cluster does not become operational within the expected time window (approximately 90 minutes), the process times out and reports the failure. The operator can re-run the bootstrap or destroy and retry.
- **Certificate issuance failure**: If TLS certificates cannot be obtained from Let's Encrypt (e.g., DNS propagation delay), the system retries for up to 10 minutes before failing. The operator can re-run the platform configuration step.
- **Operator installation timeout**: If a platform operator does not become ready within its retry window (approximately 10 minutes), the process halts and reports which component failed. The operator can investigate and re-run.
- **Node provisioning failure**: If bare-metal or additional worker nodes do not reach a ready state within the retry window (approximately 60 minutes), the system reports the failure.
- **Stop/start failures**: If cloud instances cannot be stopped or started, the system retries and reports the final instance states so the operator can intervene manually.
- **Partial deployment**: If the bootstrap completes infrastructure but fails during platform configuration, the cluster remains running. The operator can fix the issue and re-run the platform configuration step independently without reprovisioning.

## 9. Constraints & Assumptions

**Constraints**

- The operator must have an active Red Hat account with a valid pull secret to provision the container platform.
- Mixed-architecture clusters (ARM + x86) are only supported on AWS and Azure; GCP is limited to single-architecture deployments.
- Bare-metal node support and virtualization are only available on AWS.
- The system requires cloud provider credentials with sufficient permissions to create and manage compute instances, networking, DNS, and storage.
- A valid domain with DNS zone management is required for TLS certificate issuance.

**Assumptions**

- The operator has a working local environment with Python 3.9+ and Ansible installed.
- Cloud provider quotas are sufficient for the requested cluster size (minimum 6 instances for a standard topology).
- DNS zones for the configured domain are already created and delegated to the chosen cloud provider.
- The Let's Encrypt production endpoint is reachable and not rate-limited for the configured domain.
- Source control webhook endpoints are network-accessible from the external internet for automated build triggers.

## 10. Open Questions

| #  | Question                                                                                                        | Why it matters                                                                                         |
|----|-----------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------|
| 1  | The Developer Hub component is listed as a deployable feature in configuration, but no deployment manifests or ArgoCD application are defined for it. Is it fully implemented? | Operators may enable it expecting a working deployment but receive nothing.                             |
| 2  | The Azure stop/start workflow does not filter instances by cluster name — it targets all VMs in the subscription. Is this intentional? | An operator with multiple clusters or other VMs in the same subscription could accidentally stop unrelated workloads. |
| 3  | The GCP lifecycle management only queries three specific availability zones (a, b, f). What happens if the cluster spans different zones? | Instances in other zones would not be stopped or started, leading to partial lifecycle management.      |
| 4  | The README references `stop.yml` and `start.yml` for lifecycle management, which exist. However, no explicit documentation exists for `2_config_metal.yml` or `2_config_platform.yml`. Are these intended to be user-facing entry points? | Operators may not discover that they can update platform components or add bare-metal nodes independently. |
| 5  | Self-provisioning removal is configurable but the implementation only references removing `kubeadmin`. Is restricting project self-provisioning implemented? | The configuration option suggests it should be, but the behavior may not match operator expectations.   |

## 11. Future Considerations

- **Developer Hub / Internal Developer Portal**: Configuration scaffolding exists for a developer portal component (namespace, operator version channel), but deployment logic is not yet wired. This suggests a planned capability for providing a unified developer experience portal with software catalogs and templates.
- **Multi-region or multi-cluster**: The current design is single-cluster. The GitOps-centric architecture and separation of platform manifests into a dedicated repository would support extending to multi-cluster management in the future.
- **Additional cloud providers or on-premises**: The role structure cleanly separates cloud-specific logic, suggesting extensibility to additional providers, though none beyond AWS/Azure/GCP are currently supported.
- **QEMU device expansion**: The emulated hardware testing currently supports a single ARM device profile. The architecture supports adding additional device profiles or architectures.
