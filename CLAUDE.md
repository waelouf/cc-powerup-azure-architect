# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the **Azure Architect powerup** for [Claude Code Powerups](https://github.com/waelouf/claude-code-powerups) - a marketplace of specialized plugins that extend Claude Code's capabilities.

This powerup provides expert Azure cloud architecture and DevOps guidance. It helps users build, deploy, and manage production-grade Azure infrastructure through the entire lifecycle - from initial provisioning through production operations.

## Repository Structure

```
cc-powerup-azure-architect/
├── .claude-plugin/
│   └── plugin.json              # Plugin metadata
├── skills/
│   └── azure-architect/
│       ├── SKILL.md             # Main powerup definition with routing logic
│       ├── commands/            # Fast-track slash commands (13 files)
│       ├── workflows/           # Step-by-step workflow guides (13 files)
│       └── references/          # Deep domain knowledge files (15 files)
├── CLAUDE.md                    # Development guidelines (this file)
├── README.md                    # User-facing documentation
├── CHANGELOG.md                 # Version history and changes
└── STATUS.md                    # Current project status
```

### Key Architecture Concepts

**Three-Layer Structure:**
1. **SKILL.md** - Central orchestrator with intake menu and routing logic
2. **commands/** - Direct slash command entry points (e.g., `/azure-arch:provision`, `/azure-arch:deploy`)
3. **workflows/** - Comprehensive step-by-step guides for each task
4. **references/** - Domain expertise documents (IaC, security, networking, etc.)

**Powerup Flow:**
- User invokes `/azure-architect` → SKILL.md shows intake menu
- User selects option (1-13) → Routes to appropriate workflow
- OR user invokes slash command (e.g., `/provision`) → Bypasses menu, goes directly to workflow
- Workflow references domain knowledge from references/ as needed

## Powerup Development Guidelines

### When Modifying SKILL.md

- **Intake section**: Keep menu concise (13 options max for usability)
- **Routing table**: Must map user input patterns to correct workflow files
- **Essential principles**: These 7 principles are the foundation - only modify with careful consideration
- **Reference index**: Update when adding/removing reference files

### When Creating/Modifying Workflows

Workflows follow a consistent structure:
1. **Objective** - Clear statement of what this workflow accomplishes
2. **Requirements gathering** - Interactive questions to understand context
3. **Step-by-step instructions** - Actionable commands with Azure CLI
4. **Code examples** - Production-ready Bicep/Terraform templates
5. **Verification steps** - Commands to validate the deployment
6. **Next steps** - Recommendations for follow-up actions

**Critical principles for all workflows:**
- Use Azure CLI (`az`) commands, not portal instructions
- Include both Bicep and Terraform examples where applicable
- Provide real 2024-2025 pricing information for cost-related decisions
- Show verification commands after every deployment
- Reference anti-patterns from `references/anti-patterns.md`

### When Creating/Modifying Commands

Commands in `commands/` are thin wrappers that:
1. Have YAML frontmatter with name, description, and arguments
2. Set context for the skill invocation
3. Route directly to the appropriate workflow
4. Skip the intake menu for efficiency

**Template structure:**
```markdown
---
name: command-name
description: Brief description
arguments: "[optional-args]"
---

<objective>
What this command accomplishes
</objective>

<invocation>
Execute azure-architect with specific context
Arguments: $ARGUMENTS
Route to: workflows/workflow-name.md
</invocation>
```

### When Creating/Modifying References

Reference files contain deep domain expertise:
- **Target audience**: Experienced developers who need Azure-specific guidance
- **Structure**: Pure XML structure - use semantic XML tags, NOT markdown headings (##, ###)
- **Subsections**: Use bold text (**Section Name**) for subsections within XML tags
- **Code examples**: Must be production-ready, not simplified demos
- **External links**: Only link to official Microsoft docs or well-established resources
- **Pricing**: Use actual 2024-2025 Azure pricing when discussing costs

**CRITICAL**: All reference files MUST use pure XML structure. This means:
- ✅ Use `<section_name>` tags for major sections
- ✅ Use `**Bold text**` for subsection headings
- ❌ Never use markdown headings (`##` or `###`) outside of code blocks
- ✅ Close all XML tags properly
- ✅ Use semantic tag names that describe content purpose

**Key reference files:**
- `infrastructure-as-code.md` - Bicep vs Terraform, Azure Verified Modules
- `security-identity.md` - Managed Identity, RBAC, Key Vault patterns
- `cicd-pipelines.md` - GitHub Actions with OIDC, Azure DevOps
- `anti-patterns.md` - What NOT to do (critical for quality guidance)

## Core Principles (Do Not Violate)

These 7 principles are the foundation of all guidance in this skill:

1. **Infrastructure as Code is the Source of Truth** - All resources defined in Bicep/Terraform
2. **Azure CLI First, Portal Never** - All operations via `az` commands
3. **Security by Default** - Managed Identity, Key Vault, RBAC, Private Endpoints
4. **Cost Consciousness** - Right-sizing, auto-scaling, Reserved Instances, tagging
5. **Multi-Environment Architecture** - Dev/Staging/Prod from day one
6. **Observability is Not Optional** - Application Insights, Log Analytics, alerts
7. **Deployment Automation** - All deployments through CI/CD pipelines

Any changes that contradict these principles require careful justification.

## Verification After Changes

When modifying skill files, verify:

```bash
# 1. YAML frontmatter is valid (for commands)
# No automated tool - manual inspection required

# 2. All file references exist
# Check that workflow/reference paths in SKILL.md routing table are correct

# 3. Test skill invocation (requires Claude Code runtime)
/azure-architect
# Should show intake menu

# 4. Test slash commands
/provision
# Should route directly to provision workflow
```

## Common Modification Patterns

### Adding a New Workflow

1. Create workflow file in `skills/azure-architect/workflows/new-workflow.md`
2. Add routing entry to SKILL.md `<routing>` table
3. Add menu option to SKILL.md `<intake>` section (if <13 options)
4. Create corresponding slash command in `commands/` (optional but recommended)
5. Update `workflows_index` in SKILL.md
6. Update README.md examples if this is a major feature

### Adding a New Reference File

1. Create reference file in `skills/azure-architect/references/new-reference.md`
2. Add to `<reference_index>` in SKILL.md
3. Reference it from relevant workflows using context like:
   ```markdown
   For detailed guidance on [topic], see references/new-reference.md
   ```

### Updating Azure API Versions

Azure API versions are referenced in Bicep/Terraform examples:
- Check current versions: https://learn.microsoft.com/en-us/azure/templates/
- Update examples in workflows to use latest stable (not preview) API versions
- Update version info in README.md footer

### Updating Pricing Information

Pricing is critical for cost optimization guidance:
- Use Azure Pricing Calculator: https://azure.microsoft.com/en-us/pricing/calculator/
- Verify prices are for US East region unless specified otherwise
- Update examples in `references/cost-optimization.md` and `workflows/optimize-costs.md`
- Document pricing date in comments (e.g., "as of January 2025")

## File Naming Conventions

- **Workflows**: Use verb-noun format (`provision-infrastructure.md`, `setup-monitoring.md`)
- **Commands**: Match the slash command name (`provision.md`, `deploy.md`)
- **References**: Use noun format describing the domain (`security-identity.md`, `networking.md`)
- All filenames: lowercase with hyphens (no spaces, no underscores)

## Testing Guidance

This powerup has no automated tests - verification is manual through Claude Code:

1. **Functional testing**: Invoke powerup and follow workflows with real Azure subscription
2. **Code validation**: Run provided Azure CLI commands and IaC templates
3. **Routing testing**: Verify all menu options and slash commands route correctly
4. **Reference testing**: Ensure all cross-references between files are accurate

## Documentation Standards

### File Purposes

- **README.md**: User-facing, marketing-friendly, includes examples and quick start
- **CLAUDE.md**: Development guidelines for maintainers (this file)
- **CHANGELOG.md**: Version history following Keep a Changelog format
- **STATUS.md**: Current project status, metrics, recent changes
- **SKILL.md**: System-facing, routing logic, essential principles
- **Workflow files**: Step-by-step instructions with verification commands
- **Reference files**: Deep technical knowledge, decision matrices, patterns (pure XML)

### Synchronization Rules

Keep these synchronized when making changes:
- If you add a workflow, update: SKILL.md routing, README.md examples, STATUS.md
- If you change essential principles, update: SKILL.md and README.md
- If you add/remove reference files, update: SKILL.md reference index
- When releasing a version, update: README.md version, CHANGELOG.md, STATUS.md
- Version number in README.md should match semantic versioning

### XML Structure Compliance

As of v1.0.0, all reference files use pure XML structure:
- **Migration completed**: February 2025
- **Audit status**: ✅ Passed (0 markdown headings remaining)
- **Compliance**: 100% with Agent Skills best practices
- **Verification**: Use `/audit-skill` to check compliance after changes

## Version Information

This skill targets:
- **Azure API versions**: 2023-2024 stable releases
- **Terraform azurerm provider**: 3.x+
- **Bicep**: 0.x latest
- **Azure CLI**: Latest stable
- **Knowledge cutoff**: January 2025

When updating, maintain backward compatibility or document breaking changes in README.md.
