# Tasks: Azure Landing Zone Infrastructure as Code

**Feature**: `001-terraform-iac`  
**Input**: [SPEC.md](./SPEC.md), [PLAN.md](./PLAN.md)  
**Created**: 2026-01-09

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: User story mapping (US1-US5 from SPEC.md)

---

## Phase 0: Bootstrap (Prerequisites)

**Purpose**: One-time setup for state management and authentication

- [ ] T001 Create Azure Storage Account for Terraform state in `bootstrap/state-storage/main.tf`
- [ ] T002 Configure blob container with versioning and soft delete enabled
- [ ] T003 Create service principal for Terraform with Contributor + User Access Administrator roles
- [ ] T004 [P] Setup CI/CD pipeline scaffolding in `.github/workflows/terraform-plan.yml`
- [ ] T005 [P] Configure backend configuration template in `terraform/environments/template-backend.tf`
- [ ] T006 [P] Create project `.gitignore` with Terraform patterns (.terraform/, *.tfstate, *.tfvars)
- [ ] T007 Document bootstrap process in `docs/runbooks/bootstrap.md`

**Checkpoint**: State backend ready, service principal configured, CI/CD foundation established

---

## Phase 1: Foundation Modules (Blocking Prerequisites)

**Purpose**: Core reusable modules that all environments depend on

- [ ] T008 [P] Create naming module in `terraform/modules/naming/main.tf` with CAF naming logic
- [ ] T009 [P] Add naming module variables in `terraform/modules/naming/variables.tf`
- [ ] T010 [P] Add naming module outputs in `terraform/modules/naming/outputs.tf`
- [ ] T011 [P] Document naming conventions in `terraform/modules/naming/README.md`
- [ ] T012 [P] Create tagging module in `terraform/modules/naming/tags.tf` for mandatory tags
- [ ] T013 Create management groups module in `terraform/modules/management-groups/main.tf`
- [ ] T014 Add management group hierarchy (Platform, Landing Zones) with parent/child relationships
- [ ] T015 Add management group module variables and outputs
- [ ] T016 [P] Create logging module in `terraform/modules/logging/main.tf` (Log Analytics workspace)
- [ ] T017 [P] Configure log retention, pricing tier, and solutions in logging module
- [ ] T018 [P] Add Azure Sentinel configuration to logging module
- [ ] T019 [P] Add logging module variables and outputs
- [ ] T020 Validate all foundation modules with `terraform validate`

**Checkpoint**: Foundation modules complete and validated - environment deployment can begin

---

## Phase 2: User Story 3 - Manage Infrastructure State Securely (Priority: P1) 🎯 

**Goal**: Implement secure remote state management with locking

**Independent Test**: Multiple engineers run terraform commands simultaneously and verify lock behavior

### Implementation for User Story 3

- [ ] T021 [US3] Configure azurerm backend in `terraform/environments/dev/backend.tf`
- [ ] T022 [US3] Add backend configuration for staging in `terraform/environments/staging/backend.tf`
- [ ] T023 [US3] Add backend configuration for production in `terraform/environments/prod/backend.tf`
- [ ] T024 [US3] Enable blob lease locking in storage account configuration
- [ ] T025 [US3] Test state locking by running concurrent terraform operations
- [ ] T026 [US3] Verify blob versioning and soft delete are enabled on state containers
- [ ] T027 [US3] Document state recovery procedures in `docs/runbooks/state-recovery.md`

**Checkpoint**: Remote state with locking functional for all environments

---

## Phase 3: User Story 1 - Deploy Core Landing Zone Infrastructure (Priority: P1) 🎯

**Goal**: Deploy hub network, management groups, policies, and monitoring

**Independent Test**: Run `terraform apply` in dev environment and verify all hub resources created

### Part A: Management Groups & Policies

- [ ] T028 [P] [US1] Create policy assignments module in `terraform/modules/policy-assignments/main.tf`
- [ ] T029 [P] [US1] Add CIS Azure Foundations Benchmark policy initiative assignment
- [ ] T030 [P] [US1] Add policy module variables for scope and parameters
- [ ] T031 [P] [US1] Add policy module outputs with assignment IDs

### Part B: Hub Network Infrastructure

- [ ] T032 [US1] Create hub network module in `terraform/modules/hub-network/main.tf`
- [ ] T033 [US1] Define hub VNet with address space (10.0.0.0/16) and subnets
- [ ] T034 [US1] Add GatewaySubnet (/27), AzureFirewallSubnet (/26), AzureBastionSubnet (/27)
- [ ] T035 [US1] Configure Azure Firewall with public IP and firewall policy
- [ ] T036 [US1] Add VPN Gateway to GatewaySubnet
- [ ] T037 [US1] Add Azure Bastion for secure VM access
- [ ] T038 [US1] Create default NSGs with deny-all rules for hub subnets
- [ ] T039 [US1] Configure route tables for subnet routing
- [ ] T040 [US1] Add hub network variables (address_space, region, firewall_sku)
- [ ] T041 [US1] Add hub network outputs (vnet_id, firewall_ip, gateway_ids)

### Part C: Dev Environment Integration

- [ ] T042 [US1] Create dev environment main.tf calling management-groups, hub-network, logging, policy modules
- [ ] T043 [US1] Create dev variables.tf with environment-specific values
- [ ] T044 [US1] Create dev terraform.tfvars with actual values
- [ ] T045 [US1] Create dev outputs.tf exposing key resource IDs
- [ ] T046 [US1] Run `terraform init` and `terraform plan` for dev environment
- [ ] T047 [US1] Apply hub infrastructure to dev environment
- [ ] T048 [US1] Verify Log Analytics workspace created with correct retention
- [ ] T049 [US1] Verify Azure Policy assignments applied to management groups
- [ ] T050 [US1] Verify hub VNet, firewall, bastion, and gateways deployed

**Checkpoint**: Hub landing zone fully deployed and functional in dev

---

## Phase 4: User Story 2 - Deploy Landing Zone for Application Workloads (Priority: P2)

**Goal**: Implement spoke network module and deploy first spoke

**Independent Test**: Deploy spoke module and verify peering, routing, and policy inheritance

### Implementation for User Story 2

- [ ] T051 [P] [US2] Create spoke network module in `terraform/modules/spoke-network/main.tf`
- [ ] T052 [P] [US2] Define spoke VNet with configurable address space
- [ ] T053 [P] [US2] Add spoke subnets with NSG associations
- [ ] T054 [P] [US2] Configure VNet peering from spoke to hub (both directions)
- [ ] T055 [P] [US2] Create route table forcing traffic through Azure Firewall
- [ ] T056 [P] [US2] Add spoke network variables (name, address_space, hub_vnet_id, subnets)
- [ ] T057 [P] [US2] Add spoke network outputs (vnet_id, subnet_ids)
- [ ] T058 [US2] Add spoke landing zone to dev environment (app1-spoke)
- [ ] T059 [US2] Configure spoke with address space 10.1.0.0/16
- [ ] T060 [US2] Apply spoke configuration to dev environment
- [ ] T061 [US2] Verify VNet peering established (hub <-> spoke)
- [ ] T062 [US2] Verify route table routes traffic through firewall
- [ ] T063 [US2] Test connectivity from spoke to hub resources
- [ ] T064 [US2] Verify policy inheritance from parent management group

**Checkpoint**: Spoke landing zone deployed and connected to hub

---

## Phase 5: User Story 4 - Deploy Environment-Specific Configurations (Priority: P2)

**Goal**: Deploy staging and production environments with appropriate configurations

**Independent Test**: Verify resource sizing and policies differ between dev and prod

### Implementation for User Story 4

- [ ] T065 [P] [US4] Create staging environment structure in `terraform/environments/staging/`
- [ ] T066 [P] [US4] Create staging main.tf with hub and spoke modules
- [ ] T067 [P] [US4] Create staging variables.tf with medium-sized resources
- [ ] T068 [P] [US4] Create staging terraform.tfvars with staging-specific values
- [ ] T069 [P] [US4] Create production environment structure in `terraform/environments/prod/`
- [ ] T070 [P] [US4] Create production main.tf with hub and spoke modules
- [ ] T071 [P] [US4] Create production variables.tf with production-grade sizing
- [ ] T072 [P] [US4] Create production terraform.tfvars with prod-specific values
- [ ] T073 [P] [US4] Add stricter policy assignments for production (PCI-DSS, HIPAA if applicable)
- [ ] T074 [US4] Deploy staging environment
- [ ] T075 [US4] Deploy production environment
- [ ] T076 [US4] Verify resource naming includes environment prefix (dev/staging/prod)
- [ ] T077 [US4] Verify dev has relaxed policies vs production
- [ ] T078 [US4] Verify production has high availability configurations

**Checkpoint**: All three environments deployed with appropriate configurations

---

## Phase 6: User Story 5 - Implement Infrastructure Drift Detection (Priority: P3)

**Goal**: Setup automated drift detection and reporting

**Independent Test**: Make manual portal change and verify drift detected

### Implementation for User Story 5

- [ ] T079 [P] [US5] Create GitHub Actions workflow for drift detection in `.github/workflows/drift-detection.yml`
- [ ] T080 [P] [US5] Configure workflow to run `terraform plan` on schedule (daily)
- [ ] T081 [P] [US5] Add drift notification to Teams/Slack on detected changes
- [ ] T082 [P] [US5] Configure workflow to fail CI/CD if critical drift detected
- [ ] T083 [US5] Test drift detection by making manual change in Azure Portal
- [ ] T084 [US5] Verify drift appears in terraform plan output
- [ ] T085 [US5] Verify notification sent to team channel
- [ ] T086 [US5] Document drift remediation process in `docs/runbooks/drift-remediation.md`

**Checkpoint**: Drift detection automated and tested

---

## Phase 7: Testing & Quality Assurance

**Purpose**: Automated testing and security validation

- [ ] T087 [P] Setup Terratest in `tests/` directory with Go module
- [ ] T088 [P] Create hub network test in `tests/hub_test.go`
- [ ] T089 [P] Create spoke network test in `tests/spoke_test.go`
- [ ] T090 [P] Create compliance test verifying policy assignments in `tests/compliance_test.go`
- [ ] T091 [P] Integrate checkov security scanning in CI/CD pipeline
- [ ] T092 [P] Integrate tfsec security scanning in CI/CD pipeline
- [ ] T093 Run all Terratest tests against dev environment
- [ ] T094 Fix any test failures or security violations
- [ ] T095 Verify all tests pass before promoting to staging/prod

**Checkpoint**: All tests passing, no security violations

---

## Phase 8: Documentation & Knowledge Transfer

**Purpose**: Complete documentation for operation and maintenance

- [ ] T096 [P] Generate module documentation with terraform-docs for all modules
- [ ] T097 [P] Create architecture diagram in `docs/architecture/hub-spoke-topology.md`
- [ ] T098 [P] Document management group structure in `docs/architecture/management-group-structure.md`
- [ ] T099 [P] Create deployment runbook in `docs/runbooks/deployment.md`
- [ ] T100 [P] Create disaster recovery runbook in `docs/runbooks/disaster-recovery.md`
- [ ] T101 [P] Create troubleshooting guide in `docs/runbooks/troubleshooting.md`
- [ ] T102 [P] Document RBAC model in `docs/security/rbac-model.md`
- [ ] T103 [P] Create compliance mapping document in `docs/security/compliance-mapping.md`
- [ ] T104 Review all documentation for accuracy and completeness

**Checkpoint**: Documentation complete and reviewed

---

## Phase 9: Production Readiness

**Purpose**: Final validation before production handoff

- [ ] T105 Conduct security review of all infrastructure code
- [ ] T106 Verify all resources have mandatory tags
- [ ] T107 Verify all resources follow naming conventions
- [ ] T108 Check Azure Policy compliance score (target >90%)
- [ ] T109 Review cost estimates per environment
- [ ] T110 Validate disaster recovery procedures
- [ ] T111 Conduct team walkthrough of deployment process
- [ ] T112 Obtain stakeholder approval for production deployment

**Checkpoint**: Production ready for handoff

---

## Summary

- **Total Tasks**: 112
- **Estimated Duration**: 4-6 weeks
- **Parallel Opportunities**: Many tasks marked [P] can run concurrently
- **MVP Definition**: Phases 0-3 (Tasks 1-50) deliver minimum viable landing zone
- **Critical Path**: Bootstrap → Foundation → Hub Deployment → State Management
