# Implementation Plan: Azure Landing Zone Infrastructure as Code

**Branch**: `001-terraform-iac` | **Date**: 2026-01-09 | **Spec**: [SPEC.md](./SPEC.md)

## Summary

Deploy a production-ready Azure Landing Zone using Terraform with hub-spoke network topology, comprehensive governance through Azure Policy, centralized logging, and secure remote state management. The implementation follows Cloud Adoption Framework (CAF) best practices with modular architecture for reusability and environment-specific configurations.

## Technical Context

**IaC Tool**: Terraform 1.6+ with Azure Provider (azurerm) 3.0+  
**Cloud Platform**: Microsoft Azure (Global Cloud)  
**Primary Dependencies**: Azure CLI 2.0+, Terraform azurerm provider, Azure AD  
**Storage**: Azure Storage Account for Terraform state with blob versioning  
**Testing**: Terratest for infrastructure testing, terraform validate, checkov/tfsec for security scanning  
**Target Platform**: Azure subscriptions with management group hierarchy  
**Project Type**: Infrastructure as Code (multi-module Terraform project)  
**Performance Goals**: Hub deployment <45 minutes, spoke deployment <15 minutes, state operations <30 seconds  
**Constraints**: Must follow CAF naming conventions, CIS Azure Foundations compliance, encryption at rest/transit  
**Scale/Scope**: Multiple environments (dev/staging/prod), 5-10 spoke landing zones per hub, 100+ Azure resources

## Constitution Check

*Core principles for this implementation:*

- **Infrastructure as Code First**: All infrastructure must be defined in Terraform - no manual Azure Portal changes
- **Security by Default**: Network security groups deny all by default, explicit allow rules required
- **Immutable Infrastructure**: Changes via code only, destroy and recreate over in-place modifications
- **Modular Design**: Reusable modules for common patterns, avoid duplication
- **State Management**: Remote state with locking, separate state per environment
- **Documentation as Code**: Auto-generate docs from Terraform using terraform-docs
- **Policy-Driven Governance**: Azure Policy for automated compliance, not manual enforcement

## Project Structure

### Documentation (this feature)

```text
sdd-001-terraform-iac/
├── SPEC.md              # Feature specification (completed)
├── PLAN.md              # This file - implementation plan
├── TASKS.md             # Task breakdown (to be generated)
├── CHECKLIST.md         # Deployment checklist (to be generated)
├── research/
│   ├── terraform-best-practices.md
│   ├── azure-caf-patterns.md
│   ├── state-management.md
│   └── security-compliance.md
└── contracts/
    ├── module-hub.md           # Hub module interface
    ├── module-spoke.md         # Spoke module interface
    ├── module-mgmt-groups.md   # Management group module interface
    └── module-policies.md      # Policy module interface
```

### Infrastructure Code (repository root)

```text
terraform/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   ├── terraform.tfvars
│   │   └── backend.tf
│   ├── staging/
│   │   └── [same structure as dev]
│   └── prod/
│       └── [same structure as dev]
│
├── modules/
│   ├── management-groups/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   └── README.md
│   ├── hub-network/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   └── README.md
│   ├── spoke-network/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   └── README.md
│   ├── policy-assignments/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   └── README.md
│   ├── logging/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   └── README.md
│   └── naming/
│       ├── main.tf
│       ├── variables.tf
│       ├── outputs.tf
│       └── README.md
│
├── bootstrap/
│   ├── state-storage/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── README.md
│   └── service-principal/
│       ├── main.tf
│       └── README.md
│
└── tests/
    ├── hub_test.go
    ├── spoke_test.go
    └── compliance_test.go

docs/
├── architecture/
│   ├── hub-spoke-topology.md
│   ├── management-group-structure.md
│   └── network-flow.md
├── runbooks/
│   ├── deployment.md
│   ├── disaster-recovery.md
│   └── troubleshooting.md
└── security/
    ├── rbac-model.md
    └── compliance-mapping.md

.github/
└── workflows/
    ├── terraform-plan.yml
    ├── terraform-apply.yml
    └── security-scan.yml
```

**Structure Decision**: Multi-module Terraform repository with environment-specific configurations. Modules are reusable across environments, with environment folders containing only variable overrides and backend configurations. Bootstrap directory for one-time setup of state storage and service principals. Tests directory for automated infrastructure validation.

## Technical Architecture

### Module Design

**1. Management Groups Module**
- Creates Azure management group hierarchy
- Inputs: Parent IDs, display names
- Outputs: Management group IDs for policy assignment
- Dependencies: None (foundational)

**2. Hub Network Module**
- Provisions hub VNet with required subnets
- Deploys Azure Firewall, VPN Gateway, Bastion
- Configures routing and NSGs
- Inputs: Address space, region, firewall SKU
- Outputs: VNet ID, firewall IP, gateway IDs
- Dependencies: Management groups (for subscription assignment)

**3. Spoke Network Module**
- Creates spoke VNet with custom subnets
- Establishes VNet peering to hub
- Configures route tables to force tunnel through firewall
- Inputs: Address space, hub VNet ID, subnet definitions
- Outputs: VNet ID, subnet IDs
- Dependencies: Hub network module

**4. Policy Assignments Module**
- Assigns Azure Policy initiatives to scopes
- Configures policy parameters
- Sets up exemptions where needed
- Inputs: Policy definition IDs, scope, parameters
- Outputs: Assignment IDs
- Dependencies: Management groups

**5. Logging Module**
- Creates Log Analytics workspace
- Configures data retention and quotas
- Sets up Azure Monitor solutions
- Enables Azure Sentinel
- Inputs: Retention period, region, pricing tier
- Outputs: Workspace ID, workspace key
- Dependencies: Management groups

**6. Naming Module**
- Generates resource names following CAF conventions
- Enforces naming standards
- Inputs: Environment, region, resource type
- Outputs: Formatted resource names
- Dependencies: None (utility module)

### State Management Strategy

**Backend Configuration**:
- Azure Storage Account with versioning enabled
- Separate blob containers per environment (dev/staging/prod)
- State locking via Azure Storage blob lease
- Encryption at rest with Microsoft-managed keys
- Network rules to restrict access
- Soft delete enabled with 30-day retention

**Workspace Isolation**:
- One state file per environment
- No workspace switching within same directory
- Separate backend configurations in each environment folder

### CI/CD Pipeline Flow

1. **Pull Request**: 
   - `terraform fmt -check`
   - `terraform validate`
   - Security scan with checkov
   - `terraform plan` (comment on PR)

2. **Merge to Main**:
   - Deploy to dev environment automatically
   - Generate deployment summary
   - Run automated tests

3. **Tagged Release**:
   - Manual approval for staging
   - Manual approval for production
   - Automated rollback on failure

### Security Model

**RBAC Design**:
- Service principal for Terraform with minimal permissions
- Contributor + User Access Administrator on subscription
- Separate service principals per environment
- No user-based deployments

**Network Security**:
- Default deny on all NSGs
- Azure Firewall for egress filtering
- No public IPs on VMs (Azure Bastion only)
- Private endpoints for PaaS services

**Secrets Management**:
- Azure Key Vault for sensitive values
- Terraform variables for service principal credentials
- State file encryption prevents credential exposure
- No secrets in version control

### Naming Convention

Format: `{resource-type-prefix}-{environment}-{region}-{workload}-{instance}`

Examples:
- `vnet-hub-eus-platform-001`
- `fw-hub-eus-platform-001`
- `vnet-spoke-eus-app1-001`
- `law-mgmt-eus-platform-001`

### Tagging Strategy

Mandatory tags for all resources:
- `Environment`: dev/staging/prod
- `CostCenter`: Business unit code
- `Owner`: Team or individual responsible
- `Project`: Project/application name
- `ManagedBy`: "Terraform"
- `DeployedDate`: Timestamp of creation

## Integration Points

**Azure Active Directory**:
- Service principal authentication for Terraform
- RBAC role assignments for user access
- Managed identities for Azure resources

**Azure DevOps / GitHub Actions**:
- CI/CD pipeline for automated deployments
- Secure variable storage for credentials
- Approval gates for production changes

**Azure Monitor**:
- Diagnostic settings on all resources
- Logs forwarded to Log Analytics
- Alert rules for critical events
- Workbooks for visualization

**Azure Security Center**:
- Automated security recommendations
- Compliance dashboard
- Integration with Azure Policy

**Azure Cost Management**:
- Budget alerts per environment
- Cost analysis by tag
- Recommendations for optimization

## Dependencies

**Prerequisites**:
- Azure subscription with sufficient quota
- Azure AD tenant with admin access
- Service principal with Contributor + User Access Administrator
- Azure Storage Account for state (can be bootstrapped)
- Local tools: Terraform CLI, Azure CLI, Git

**Network Planning**:
- Hub VNet: 10.0.0.0/16
- Spoke VNets: 10.1.0.0/16, 10.2.0.0/16, etc.
- Gateway Subnet: /27 minimum
- Firewall Subnet: /26 minimum
- Bastion Subnet: /27 minimum

## Open Questions to Resolve

1. **Azure Region**: Which primary region (e.g., East US, West Europe)?
2. **CIDR Allocation**: Confirm IP address ranges for hub and spokes
3. **Policy Initiatives**: Which compliance frameworks (CIS, PCI-DSS, HIPAA)?
4. **DDoS Protection**: Standard tier required (additional cost)?
5. **ExpressRoute/VPN**: On-premises connectivity needed?
6. **Disaster Recovery**: Multi-region deployment? RTO/RPO targets?
7. **Backup Strategy**: Which resources require Azure Backup?
8. **DNS Management**: Use Azure DNS or external provider?
9. **Cost Budget**: Per-environment spending limits?
10. **Approval Process**: Who approves production deployments?

## Success Criteria

- All Terraform code passes `terraform validate` and formatting checks
- Security scans show zero critical/high vulnerabilities
- Deployment completes within time targets (hub <45min, spoke <15min)
- Azure Policy compliance score >90%
- All resources follow naming conventions
- State operations complete within 30 seconds
- No unencrypted resources in any environment
- Documentation auto-generated and up-to-date

## Implementation Phases

**Phase 0: Bootstrap** (1-2 days)
- Setup Azure Storage Account for state
- Create service principals with proper permissions
- Configure local development environment
- Establish CI/CD pipeline structure

**Phase 1: Foundation** (3-5 days)
- Implement naming and tagging modules
- Create management group hierarchy module
- Setup logging and monitoring module
- Establish testing framework

**Phase 2: Hub Network** (5-7 days)
- Implement hub network module
- Deploy Azure Firewall
- Configure VPN/ExpressRoute gateways
- Setup Azure Bastion

**Phase 3: Spoke Networks** (3-5 days)
- Implement spoke network module
- Configure VNet peering
- Setup routing through hub

**Phase 4: Governance** (3-4 days)
- Implement policy assignment module
- Apply compliance policies
- Configure RBAC roles

**Phase 5: Environment Deployment** (2-3 days)
- Deploy dev environment
- Deploy staging environment
- Deploy production environment

**Phase 6: Testing & Documentation** (2-3 days)
- Run automated tests
- Generate module documentation
- Create runbooks
- Conduct security review

**Total Estimated Time**: 19-29 days (approximately 4-6 weeks)

## Risk Management

**Risk**: State file corruption or loss
**Mitigation**: Blob versioning, soft delete, regular backups

**Risk**: Quota limits during deployment
**Mitigation**: Pre-deployment quota validation, request increases in advance

**Risk**: Policy conflicts with existing assignments
**Mitigation**: Audit existing policies, plan migration strategy

**Risk**: Network address space conflicts
**Mitigation**: Document IP allocation plan before deployment

**Risk**: Drift from manual changes
**Mitigation**: Automated drift detection, read-only Azure Portal access

**Risk**: Security vulnerabilities in modules
**Mitigation**: Automated security scanning in CI/CD, regular updates

## Next Steps

1. Review and validate this plan
2. Answer open questions
3. Generate TASKS.md with `/speckit.tasks`
4. Generate CHECKLIST.md with `/speckit.checklist`
5. Begin Phase 0: Bootstrap implementation
