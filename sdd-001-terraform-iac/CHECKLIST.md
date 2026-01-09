# Pre-Deployment Checklist: Azure Landing Zone Infrastructure

**Feature**: `001-terraform-iac`  
**Purpose**: Verify all prerequisites before deploying infrastructure  
**Created**: 2026-01-09  

## Prerequisites

### Azure Environment

- [ ] CHK001 Azure subscription created with sufficient quota
- [ ] CHK002 Verified available quota for VMs, public IPs, VNets in target region
- [ ] CHK003 Azure AD tenant access confirmed
- [ ] CHK004 Global Administrator or equivalent access available for management group creation
- [ ] CHK005 Contributor role assigned on target subscription(s)
- [ ] CHK006 User Access Administrator role assigned for RBAC management

### Network Planning

- [ ] CHK007 IP address plan documented (hub: 10.0.0.0/16, spokes: 10.x.0.0/16)
- [ ] CHK008 No IP conflicts with existing on-premises or Azure networks
- [ ] CHK009 Target Azure region selected (e.g., East US, West Europe)
- [ ] CHK010 Secondary region selected for disaster recovery (if required)
- [ ] CHK011 DNS strategy defined (Azure DNS or external provider)
- [ ] CHK012 On-premises connectivity requirements documented (ExpressRoute/VPN)

### Service Principal Setup

- [ ] CHK013 Service principal created for Terraform
- [ ] CHK014 Service principal has Contributor role on subscription
- [ ] CHK015 Service principal has User Access Administrator role
- [ ] CHK016 Service principal client ID documented
- [ ] CHK017 Service principal client secret stored securely
- [ ] CHK018 Service principal tenant ID documented
- [ ] CHK019 Service principal credentials tested with Azure CLI login

### State Storage Bootstrap

- [ ] CHK020 Azure Storage Account created for Terraform state
- [ ] CHK021 Storage account has blob versioning enabled
- [ ] CHK022 Storage account has soft delete enabled (30-day retention)
- [ ] CHK023 Separate blob containers created for dev/staging/prod
- [ ] CHK024 Storage account access restricted to service principal
- [ ] CHK025 Storage account encrypted at rest (default Microsoft-managed keys)
- [ ] CHK026 Storage account connection string documented securely

### Local Development Environment

- [ ] CHK027 Terraform CLI installed (version 1.6.0 or higher)
- [ ] CHK028 Azure CLI installed (version 2.0 or higher)
- [ ] CHK029 Git installed and configured
- [ ] CHK030 Code editor/IDE setup (VS Code recommended with Terraform extension)
- [ ] CHK031 Azure CLI authenticated: `az login`
- [ ] CHK032 Correct subscription selected: `az account set --subscription <id>`
- [ ] CHK033 Terraform authenticated: `az login` or service principal configured

### CI/CD Pipeline

- [ ] CHK034 GitHub repository or Azure DevOps project created
- [ ] CHK035 Repository access granted to team members
- [ ] CHK036 Service principal credentials added as secrets/variables
- [ ] CHK037 Branch protection rules configured for main branch
- [ ] CHK038 Approval gates configured for production deployments
- [ ] CHK039 Notification channels setup (Teams/Slack/Email)

### Compliance & Governance

- [ ] CHK040 Compliance frameworks identified (CIS, PCI-DSS, HIPAA, etc.)
- [ ] CHK041 Required Azure Policy initiatives documented
- [ ] CHK042 Naming convention standard agreed upon
- [ ] CHK043 Tagging strategy defined (mandatory tags documented)
- [ ] CHK044 RBAC model designed and documented
- [ ] CHK045 Cost center codes obtained for tagging
- [ ] CHK046 Data residency requirements documented

### Security

- [ ] CHK047 Azure Key Vault created for secrets management (if needed)
- [ ] CHK048 Security contact email configured
- [ ] CHK049 Azure Security Center/Defender enabled on subscription
- [ ] CHK050 Security scanning tools configured (checkov/tfsec)
- [ ] CHK051 Security incident response team identified
- [ ] CHK052 Encryption requirements documented

### Cost Management

- [ ] CHK053 Budget allocated per environment (dev/staging/prod)
- [ ] CHK054 Azure Cost Management budgets created
- [ ] CHK055 Budget alert thresholds configured
- [ ] CHK056 Cost approval process defined for production
- [ ] CHK057 Resource sizing guidelines documented per environment

## During Deployment

### Terraform Execution

- [ ] CHK058 `terraform fmt` passes on all .tf files
- [ ] CHK059 `terraform validate` passes on all modules
- [ ] CHK060 `terraform plan` reviewed before apply
- [ ] CHK061 Plan shows expected resource count
- [ ] CHK062 No unexpected resource deletions in plan
- [ ] CHK063 Security scan (checkov/tfsec) passes with no critical issues
- [ ] CHK064 `terraform apply` completed successfully
- [ ] CHK065 State file stored in remote backend
- [ ] CHK066 State lock released after apply

### Resource Validation

- [ ] CHK067 Management group hierarchy created correctly
- [ ] CHK068 Hub VNet created with correct address space
- [ ] CHK069 All required subnets present (Gateway, Firewall, Bastion)
- [ ] CHK070 Azure Firewall deployed and running
- [ ] CHK071 VPN Gateway deployed (if required)
- [ ] CHK072 Azure Bastion deployed and accessible
- [ ] CHK073 Log Analytics workspace created
- [ ] CHK074 Azure Sentinel enabled (if required)
- [ ] CHK075 Policy assignments applied to correct scopes
- [ ] CHK076 All resources follow naming convention
- [ ] CHK077 All resources have mandatory tags
- [ ] CHK078 NSGs configured with default deny rules

### Network Connectivity

- [ ] CHK079 Hub VNet connectivity verified
- [ ] CHK080 Spoke VNet(s) created with correct address space
- [ ] CHK081 VNet peering established between hub and spokes
- [ ] CHK082 Route tables configured for spoke-to-hub routing
- [ ] CHK083 Traffic routes through Azure Firewall
- [ ] CHK084 Azure Bastion can connect to VMs (if VMs deployed)
- [ ] CHK085 DNS resolution working correctly

### Monitoring & Logging

- [ ] CHK086 Diagnostic settings enabled on all resources
- [ ] CHK087 Logs flowing to Log Analytics workspace
- [ ] CHK088 Azure Monitor alerts configured
- [ ] CHK089 Activity log retention configured
- [ ] CHK090 Workbooks deployed for visualization (if applicable)

## Post-Deployment

### Testing

- [ ] CHK091 Automated tests executed (Terratest)
- [ ] CHK092 All tests passed
- [ ] CHK093 Policy compliance score checked (target >90%)
- [ ] CHK094 Security posture reviewed in Security Center
- [ ] CHK095 Cost analysis reviewed for unexpected charges
- [ ] CHK096 Drift detection tested (manual change + terraform plan)

### Documentation

- [ ] CHK097 Architecture diagram created and reviewed
- [ ] CHK098 Deployment runbook created
- [ ] CHK099 Disaster recovery procedures documented
- [ ] CHK100 Troubleshooting guide created
- [ ] CHK101 Module documentation generated with terraform-docs
- [ ] CHK102 RBAC model documented
- [ ] CHK103 Compliance mapping documented

### Handoff

- [ ] CHK104 Team walkthrough conducted
- [ ] CHK105 Access credentials shared securely with team
- [ ] CHK106 On-call procedures defined
- [ ] CHK107 Escalation contacts documented
- [ ] CHK108 Change management process communicated
- [ ] CHK109 Backup and recovery procedures tested
- [ ] CHK110 Stakeholder approval obtained for production

### Continuous Operations

- [ ] CHK111 Drift detection scheduled (daily recommended)
- [ ] CHK112 Security scanning integrated in CI/CD
- [ ] CHK113 Regular review cadence established for policies
- [ ] CHK114 Terraform version upgrade strategy defined
- [ ] CHK115 Module update strategy defined
- [ ] CHK116 Incident response plan documented
- [ ] CHK117 Performance monitoring baselines established
- [ ] CHK118 Cost optimization review scheduled (monthly/quarterly)

## Notes

- Check items off as completed: `[x]`
- Document any deviations or exceptions inline
- Link to relevant documentation or runbooks where applicable
- Review this checklist before each environment deployment (dev, staging, prod)

## Sign-off

**Completed By**: _________________  
**Date**: _________________  
**Environment**: [ ] Dev [ ] Staging [ ] Production  
**Approved By**: _________________  
**Date**: _________________
