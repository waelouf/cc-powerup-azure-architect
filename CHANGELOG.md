# Changelog

All notable changes to the Azure Architect Powerup will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2025-02-06

### Added
- Initial release of Azure Architect powerup for Claude Code
- 13 interactive workflows covering complete Azure lifecycle:
  - Provision infrastructure
  - Setup CI/CD pipeline
  - Deploy applications
  - Setup monitoring
  - Debug deployments
  - Optimize costs
  - Secure infrastructure
  - Setup networking
  - Design shared resources
  - Scale infrastructure
  - Setup environments
  - Implement disaster recovery
  - Migrate to Azure
- 13 fast-track slash commands for direct workflow access
- 15 comprehensive reference files with deep domain expertise:
  - Infrastructure as Code (Bicep vs Terraform)
  - Security & Identity (Managed Identity, RBAC, Key Vault)
  - CI/CD Pipelines (GitHub Actions, Azure DevOps)
  - Cost Optimization (FinOps principles)
  - Monitoring & Observability
  - Compute Services
  - Deployment Strategies
  - Resource Organization
  - Networking
  - Storage & Data
  - Kubernetes/AKS
  - Disaster Recovery
  - Architecture Patterns
  - Anti-Patterns
  - Multi-Tenancy Patterns
- Pure XML structure for all reference files (compliant with Agent Skills best practices)
- Production-ready code examples (Bicep and Terraform)
- Real 2024-2025 Azure pricing information
- Complete CLAUDE.md with development guidelines
- Comprehensive README.md with examples and scenarios
- MIT License for open source distribution
- Complete documentation suite (README, CHANGELOG, STATUS, LICENSE)

### Technical Details
- All reference files use semantic XML tags instead of markdown headings
- SKILL.md implements three-layer architecture (orchestrator → commands → workflows → references)
- Progressive disclosure pattern for managing complexity
- 22,241 lines of expert Azure guidance

[1.0.0]: https://github.com/waelouf/cc-powerup-azure-architect/releases/tag/v1.0.0
