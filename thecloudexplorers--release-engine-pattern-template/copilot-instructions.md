## release-engine-pattern-template

> This repository serves as the **Abstraction Layer** in the Release Engine three-tier solution. It contains reusable workload patterns that define "how" infrastructure and applications are deployed through Infrastructure as Code (Bicep) templates and pipeline configurations.

# AI Agent Instructions for Release Engine Workload Patterns

## 🏗️ Repository Overview

This repository serves as the **Abstraction Layer** in the Release Engine three-tier solution. It contains reusable workload patterns that define "how" infrastructure and applications are deployed through Infrastructure as Code (Bicep) templates and pipeline configurations.

## Repository Role in Release Engine Solution

### Three-Tier Architecture
```text
Configuration Layer (Simple) → Abstraction Layer (THIS REPO) → Core Layer (Pipelines)
```

### This Repository's Purpose
- **Template Repository**: Organizations clone this repo to create their own workload pattern repositories
- **Pattern Library**: Contains reusable deployment patterns for common workloads
- **Infrastructure as Code**: Bicep templates with Azure Verified Modules integration
- **Pipeline Orchestration**: Workload-specific pipeline configurations

## 📁 Repository Structure

### Pattern Organization
```text
patterns/
├── multi_stage_pattern/          # Complex multi-stage deployments
│   ├── workload.yml             # Pipeline configuration
│   ├── multi_stage_pattern.prerequisite.bicep
│   └── multi_stage_pattern.dependent.bicep
├── resource_group_scope_pattern/ # Simple single-resource deployments within resource group
│   ├── workload.yml             # Pipeline configuration
│   └── resource_group_scope_pattern.bicep
└── subscription_scope_pattern/   # Subscription-level deployments
    ├── workload.yml             # Pipeline configuration
    └── subscription_scope_pattern.bicep
```

### Required Files Per Pattern
Each pattern directory must contain:
- **`workload.yml`**: Pipeline configuration and deployment orchestration
- **`workload.bicep`** (or multiple .bicep files): Infrastructure as Code definitions
- **`README.md`** (recommended): Pattern documentation and usage instructions

## 🔧 Development Guidelines

### Creating New Patterns

1. **Pattern Naming Convention**
   - Use descriptive names: `webapp_with_database`, `function_app_premium`, `aks_cluster_basic`
   - Use underscores for separation
   - Keep names concise but clear

2. **Bicep Template Standards**
   ```bicep
   metadata resources = {
     version: '0.1.0'
     author: '<Author Name>'
     company: '<Organization Name>'
     description: '<Pattern Description>'
   }

   targetScope = 'subscription' // or 'resourceGroup'

   @allowed([
     'westeurope'
     'uksouth'
     'eastus'
   ])
   @description('Region in which resources should be deployed')
   param resourceLocation string

   param tags object = {}
   ```

3. **Azure Verified Modules (AVM) Priority**
   - **Always check AVM availability first**: Use `br/public:avm/res/` registry
   - **Document AVM unavailability**: If AVM doesn't exist, document why direct resources are used
   - **Version pinning**: Always pin to specific AVM versions for consistency
   - **Parameter mapping**: Map pattern parameters to AVM module parameters

4. **Parameter Management**
   - Use consistent parameter naming across patterns
   - Provide default values where appropriate
   - Document all parameters with descriptions
   - Use validation rules (@allowed, @minLength, etc.)

### Pipeline Configuration (`workload.yml`)

#### Template Structure
```yaml
parameters:
  - name: deploymentSettings
    type: object

variables:
  - name: serviceConnection
    value: <default-service-connection> # Override per environment

stages:
  - template: /pipelines/01-orchestrators/pattern.orchestrator.yml@release-engine-core
    parameters:
      patternSettings:
        name: <pattern_name>
        configurationFilePath: ${{ parameters.deploymentSettings.configurationFilePath }}
        environments: ${{ parameters.deploymentSettings.environments }}
        patternArtifactsPath: /patterns/<pattern_name>
        stages:
          - infrastructure:
              iac:
                name: <deployment_stage_name>
                displayName: <human_readable_name>
                deploymentScope: <Subscription|ResourceGroup|Tenant>
                serviceConnection: $(serviceConnection)
                iacMainFileName: <bicep_file_name>.bicep
                iacParameterFileName: ${{ parameters.deploymentSettings.iacParameterFileName }}
                dependsOn: <optional_dependency_stage>
                lastInStage: <true|false>
```

#### Multi-Stage Dependencies
For complex patterns with multiple deployment stages:
```yaml
stages:
  # Prerequisites stage (no dependencies)
  - infrastructure:
      iac:
        name: prerequisite_stage
        displayName: Prerequisites
        deploymentScope: Subscription
        iacMainFileName: prerequisite.bicep
        
  # Dependent stage 1 (depends on prerequisites)
  - infrastructure:
      iac:
        name: dependent_stage1
        displayName: Main Infrastructure
        deploymentScope: ResourceGroup
        iacMainFileName: main.bicep
        dependsOn: prerequisite_stage
        
  # Final stage (depends on stage 1, marks end of environment)
  - infrastructure:
      iac:
        name: final_stage
        displayName: Application Deployment
        deploymentScope: ResourceGroup
        iacMainFileName: application.bicep
        dependsOn: dependent_stage1
        lastInStage: true
```

#### Environment Considerations
- **Service Connections**: Will be overridden per environment at runtime
- **Parameter Files**: Reference parameter files that will exist in configuration repositories
- **Scope Selection**: Choose appropriate deployment scope (Subscription, ResourceGroup, Tenant)

## 🛠️ Infrastructure as Code Best Practices

### Bicep Template Guidelines

1. **Resource Naming**
   ```bicep
   // Use variables for consistent naming
   var resourceNames = {
     storageAccount: 'st${uniqueString(resourceGroup().id)}'
     keyVault: 'kv-${workloadName}-${environmentName}'
     appService: 'app-${workloadName}-${environmentName}'
   }
   ```

2. **Output Management**
   ```bicep
   // Always provide useful outputs
   output storageAccountId string = storageAccount.outputs.resourceId
   output keyVaultUri string = keyVault.outputs.uri
   output appServiceUrl string = 'https://${appService.outputs.defaultHostname}'
   ```

3. **Parameter Validation**
   ```bicep
   @description('Storage account SKU')
   @allowed([
     'Standard_LRS'
     'Standard_GRS'
     'Premium_LRS'
   ])
   param storageAccountSku string = 'Standard_LRS'

   @description('Application name')
   @minLength(3)
   @maxLength(20)
   param applicationName string
   ```

4. **Security Best Practices**
   ```bicep
   // Use system-assigned managed identities
   identity: {
     type: 'SystemAssigned'
   }

   // Enable HTTPS only
   httpsOnly: true

   // Use latest TLS version
   minTlsVersion: '1.2'
   ```

### Azure Verified Modules Integration

#### Finding AVM Modules
1. **Browse Registry**: Check `https://github.com/Azure/bicep-registry-modules`
2. **Version Selection**: Use latest stable version unless specific version required
3. **Parameter Mapping**: Review module parameters and map to pattern needs

#### AVM Usage Example
```bicep
// Good: Using AVM for storage account
module storageAccount 'br/public:avm/res/storage/storage-account:0.27.1' = {
  name: 'storageAccountDeployment'
  params: {
    name: storageAccountName
    location: resourceLocation
    skuName: storageAccountSku
    tags: tags
  }
}

// When AVM unavailable: Document reason
// NOTE: AVM module for Azure Container Apps not available as of version 0.27.1
// Using direct resource definition until AVM module is released
resource containerApp 'Microsoft.App/containerApps@2023-05-01' = {
  name: containerAppName
  location: resourceLocation
  // ... rest of configuration
}
```

## 📚 Pattern Documentation Standards

### README.md Template
Each pattern should include comprehensive documentation:

```markdown
# [Pattern Name] Pattern

## Overview
Brief description of what this pattern deploys and its intended use case.

## Architecture
Description or diagram of the deployed architecture.

## Prerequisites
- Required Azure permissions
- Dependencies on other patterns or resources
- Service principal requirements

## Parameters
| Parameter | Type | Description | Default | Required |
|-----------|------|-------------|---------|----------|
| param1 | string | Description | value | Yes |

## Deployment Scopes
- Subscription: [X] / [ ]  
- Resource Group: [X] / [ ]
- Tenant: [ ] / [X]

## Resources Created
- Resource Type 1 (AVM: module-name:version)
- Resource Type 2 (Direct: reason for not using AVM)

## Outputs
List of outputs provided by this pattern and their purposes.

## Usage Example
Reference to configuration repository setup and parameter file examples.

## Dependencies
If this pattern depends on other patterns, list them here.

## Maintenance Notes
Any special considerations for maintaining or updating this pattern.
```

## 🔄 Template Repository Management

### Using This Repository as Template

#### For Organizations
1. **Clone Template Repository**
   ```bash
   git clone https://github.com/thecloudexplorers/release-engine-example-workload-pattern.git
   cd release-engine-example-workload-pattern
   
   # Rename for your organization
   # Example: release-engine-myorg-workload-patterns
   ```

2. **Set Up Upstream Remote**
   ```bash
   git remote add upstream https://github.com/thecloudexplorers/release-engine-example-workload-pattern.git
   git remote -v
   ```

3. **Customize for Organization**
   - Update service connection names in workload.yml files
   - Modify patterns for organizational standards
   - Add organization-specific patterns
   - Update documentation

#### Upstream Synchronization
```bash
# Regular upstream sync (monthly recommended)
git fetch upstream
git checkout -b sync-upstream-$(date +%Y%m%d)
git merge upstream/main
# Resolve conflicts, test changes
git push origin sync-upstream-$(date +%Y%m%d)
# Create pull request to merge into main
```

### Contributing Back to Template

#### Contribution Guidelines
1. **New Patterns**: Submit patterns that would benefit multiple organizations
2. **Bug Fixes**: Fix issues in existing patterns or pipeline configurations
3. **Documentation**: Improve pattern documentation and examples
4. **Best Practices**: Share improvements in Bicep templates or AVM usage

#### Contribution Process
1. Fork repository on GitHub
2. Create feature branch: `feature/new-pattern-name`
3. Develop and test changes
4. Update documentation
5. Submit pull request with:
   - Clear description of changes
   - Testing evidence
   - Documentation updates

## 🎯 Pattern Categories and Examples

### Platform Patterns
Infrastructure foundations and shared services:
- **logging_infrastructure**: Centralized logging with Log Analytics
- **monitoring_infrastructure**: Azure Monitor and Application Insights setup
- **networking_hub**: Hub network with firewall and gateway
- **security_baseline**: Key Vault, managed identities, and security configurations

### Application Patterns
Application-specific deployment patterns:
- **webapp_basic**: Simple web app with app service plan
- **webapp_with_database**: Web app with SQL database
- **function_app_premium**: Premium function app with storage and insights
- **container_app**: Containerized application with container apps

### Data Patterns
Data platform and analytics patterns:
- **sql_database**: SQL Database with backup and security
- **cosmos_database**: Cosmos DB with multiple consistency levels
- **synapse_workspace**: Synapse Analytics workspace and pools
- **data_factory**: Data Factory with linked services and pipelines

### Integration Patterns
Integration and messaging patterns:
- **service_bus**: Service Bus namespace with queues and topics
- **event_hub**: Event Hub namespace for stream processing
- **api_management**: API Management instance with policies
- **logic_app**: Logic App with connectors and workflows

## 🔍 Testing and Validation

### Pattern Testing
1. **Bicep Validation**
   ```bash
   az bicep build --file patterns/pattern_name/workload.bicep
   ```

2. **Parameter Validation**
   - Test with different parameter combinations
   - Validate required vs. optional parameters
   - Test parameter validation rules

3. **Deployment Testing**
   - Deploy in isolated test environment
   - Verify all resources created correctly
   - Test outputs and dependencies

### Pipeline Testing
1. **YAML Validation**
   - Validate YAML syntax
   - Check template references
   - Verify parameter passing

2. **Integration Testing**
   - Test with configuration repository
   - Verify environment-specific deployments
   - Test dependency resolution

## 🏷️ Tagging and Versioning

### Git Tagging Strategy
```bash
# Tag releases for version tracking
git tag -a v1.0.0 -m "Release v1.0.0: Initial pattern set"
git push origin v1.0.0
```

### Version Management
- **Semantic Versioning**: Use MAJOR.MINOR.PATCH format
- **Breaking Changes**: Increment MAJOR version
- **New Patterns**: Increment MINOR version  
- **Bug Fixes**: Increment PATCH version

### Change Documentation
Maintain CHANGELOG.md with:
- Added patterns
- Changed patterns
- Deprecated patterns
- Removed patterns
- Breaking changes

## 💡 Advanced Pattern Development

### Multi-Stage Pattern Design
For complex deployments requiring multiple stages:

1. **Prerequisite Stage**: Foundational resources (networking, identity)
2. **Infrastructure Stage**: Core infrastructure (databases, storage)
3. **Application Stage**: Application resources (web apps, functions)
4. **Configuration Stage**: Post-deployment configuration

### Cross-Pattern Dependencies
When patterns depend on other patterns:
- Document dependencies clearly
- Provide deployment order guidance
- Consider creating composite patterns for common combinations

### Environment-Specific Variations
Handle environment differences through:
- Conditional deployments based on environment parameters
- Environment-specific SKUs and configurations
- Different security requirements per environment

---

## 📞 Support and Resources

### Internal Resources
- **Core Release Engine**: https://github.com/thecloudexplorers/release-engine-core
- **Configuration Template**: https://github.com/thecloudexplorers/release-engine-config-template
- **Documentation**: Complete architectural documentation in core repository

### External Resources
- **Azure Bicep**: https://docs.microsoft.com/en-us/azure/azure-resource-manager/bicep/
- **Azure Verified Modules**: https://github.com/Azure/bicep-registry-modules
- **Azure DevOps YAML**: https://docs.microsoft.com/en-us/azure/devops/pipelines/yaml-schema

### Community
- **Discussions**: Use GitHub Discussions for questions and ideas
- **Issues**: Report bugs and request features through GitHub Issues
- **Contributions**: Follow contribution guidelines for pull requests

---

*These instructions help AI agents understand the workload pattern repository structure, development guidelines, and integration with the broader Release Engine solution.*

---
> Source: [thecloudexplorers/release-engine-pattern-template](https://github.com/thecloudexplorers/release-engine-pattern-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
