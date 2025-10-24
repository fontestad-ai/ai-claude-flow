# GitHub Issue Best Practices Guide

## Table of Contents

1. [Overview](#overview)
2. [Issue Types & Categories](#issue-types--categories)
3. [Issue Structure & Formatting](#issue-structure--formatting)
4. [Common Sections & Organization](#common-sections--organization)
5. [Naming Conventions](#naming-conventions)
6. [Labeling Strategy](#labeling-strategy)
7. [Dos and Don'ts](#dos-and-donts)
8. [Swarm Coordination Patterns](#swarm-coordination-patterns)
9. [Automation & Integration](#automation--integration)
10. [Quality Checklist](#quality-checklist)

---

## Overview

This guide provides comprehensive best practices for creating, managing, and organizing GitHub issues in the ai-claude-flow repository. These practices are optimized for Claude Code integration and support automated workflows, swarm coordination, and intelligent project management.

### Key Principles

1. **Clarity First**: Every issue should be immediately understandable
2. **Actionable Content**: Include specific, measurable objectives
3. **Structured Organization**: Use consistent sections and formatting
4. **Automation-Ready**: Format for machine parsing and workflow integration
5. **Swarm-Friendly**: Support multi-agent coordination and tracking

---

## Issue Types & Categories

### 1. Bug Reports 🐛

**Purpose**: Document defects, errors, or unexpected behavior

**When to Use**:
- Code produces incorrect results
- Features not working as designed
- Performance degradation
- Data integrity issues
- Security vulnerabilities

**Template**: See [bug-report.md](./templates/bug-report.md)

**Example Title**: `[BUG] MCP Pattern Store Not Persisting Data`

---

### 2. Feature Requests ✨

**Purpose**: Propose new functionality or enhancements

**When to Use**:
- Adding new capabilities
- Extending existing features
- Improving user experience
- Integration requests

**Template**: See [feature-request.md](./templates/feature-request.md)

**Example Title**: `[FEATURE] Real-time Collaboration with WebSockets`

---

### 3. Enhancement Proposals 🔧

**Purpose**: Improvements to existing functionality

**When to Use**:
- Performance optimizations
- Code refactoring
- Documentation improvements
- Developer experience enhancements

**Template**: See [enhancement.md](./templates/enhancement.md)

**Example Title**: `[ENHANCEMENT] Optimize Swarm Coordination Performance`

---

### 4. Documentation 📚

**Purpose**: Documentation updates, additions, or corrections

**When to Use**:
- Missing documentation
- Outdated guides
- Tutorial improvements
- API documentation

**Template**: See [documentation.md](./templates/documentation.md)

**Example Title**: `[DOCS] Update Integration Guides for v2.7.1`

---

### 5. Integration Tasks 🔄

**Purpose**: Complex integration between components or systems

**When to Use**:
- Multi-component integration
- Third-party service integration
- Cross-repository coordination
- System architecture changes

**Template**: See [integration-task.md](./templates/integration-task.md)

**Example Title**: `[INTEGRATION] Complete Claude-Flow and Ruv-Swarm Integration`

---

### 6. Incident Reports 🚨

**Purpose**: Document incidents requiring immediate attention or rollback

**When to Use**:
- Production issues
- Rollback requirements
- Critical failures
- Security incidents

**Template**: See [rollback-incident.md](./templates/rollback-incident.md)

**Example Title**: `🔄 Rollback Incident: v2.7.0 Deployment Failure`

---

### 7. Swarm Tasks 🐝

**Purpose**: Issues designed for multi-agent swarm coordination

**When to Use**:
- Complex multi-step tasks
- Parallel agent execution
- Coordinated workflows
- Large-scale automation

**Template**: See [swarm-task.md](./templates/swarm-task.md)

**Example Title**: `[SWARM] Multi-Repo Version Alignment and Synchronization`

---

## Issue Structure & Formatting

### Standard Issue Structure

Every issue should follow this hierarchy:

```markdown
---
title: "[TYPE] Clear, Descriptive Title"
labels: type, priority, area
assignees: username
---

## 🎯 [Status/Type Indicator] Overview

[Brief 1-2 sentence summary]

## Objectives

- [ ] Specific, measurable objective 1
- [ ] Specific, measurable objective 2
- [ ] Specific, measurable objective 3

## [Context-Specific Sections]

### Section 1
[Detailed content]

### Section 2
[Detailed content]

## Swarm Coordination (if applicable)

**Topology**: mesh | hierarchical | ring | star
**Agents**: agent-type-1, agent-type-2, agent-type-3

## Checklist

- [ ] Implementation complete
- [ ] Tests passing
- [ ] Documentation updated
- [ ] PR created

## Related Issues/PRs

- Relates to #123
- Depends on #456
- Blocks #789

---

**Metadata**
- **Version**: vX.Y.Z
- **Priority**: Critical | High | Medium | Low
- **Complexity**: High | Medium | Low
```

---

### Formatting Best Practices

#### 1. Use Emojis for Visual Organization

**Standard Emoji Conventions**:
- 🐛 Bug reports
- ✨ Feature requests
- 🔧 Enhancements
- 📚 Documentation
- 🔄 Integration tasks
- 🚨 Incidents/Rollbacks
- 🐝 Swarm tasks
- ✅ Completed items
- ❌ Failed/Blocked items
- 🔄 In Progress
- ⏳ Pending
- 📊 Metrics/Analytics
- 🎯 Goals/Objectives
- 🏗️ Architecture
- 🔒 Security
- ⚡ Performance
- 💡 Ideas/Suggestions

#### 2. Checkbox Lists for Task Tracking

```markdown
## Objectives
- [ ] Uncompleted task
- [x] Completed task
- [ ] Pending task with subtasks:
  - [x] Subtask 1
  - [ ] Subtask 2
```

#### 3. Code Blocks with Language Hints

```markdown
### Implementation Example
\`\`\`javascript
// Always specify language for syntax highlighting
const example = "code here";
\`\`\`

### Command Example
\`\`\`bash
# Include command examples
npx claude-flow command --option
\`\`\`
```

#### 4. Tables for Structured Data

```markdown
| Status | Before | After |
|--------|--------|-------|
| Feature A | ⚠️ Partial | ✅ Complete |
| Feature B | ❌ Broken | ✅ Fixed |
```

#### 5. Collapsible Sections for Long Content

```markdown
<details>
<summary><strong>Click to expand: Detailed Analysis</strong></summary>

[Long detailed content here]

</details>
```

---

## Common Sections & Organization

### Required Sections (All Issues)

#### 1. Title
- **Format**: `[TYPE] Clear, Descriptive Title (50-80 chars)`
- **Guidelines**:
  - Start with issue type in brackets
  - Use action verbs
  - Be specific and searchable
  - Include key components/features

**Good Examples**:
- `[BUG] MCP Pattern Store Not Persisting to Memory`
- `[FEATURE] Real-time WebSocket Collaboration`
- `[DOCS] Update API Documentation for v2.7.1`

**Bad Examples**:
- `Bug in code` (too vague)
- `Fix the thing` (not descriptive)
- `Update docs` (missing specificity)

---

#### 2. Overview/Summary
- **Purpose**: Provide immediate context
- **Length**: 1-3 sentences
- **Content**: What, why, and impact

```markdown
## Overview

[Component/Feature] is experiencing [issue/need] which affects [users/systems].
This [issue/feature] is needed to [goal/outcome].
```

---

#### 3. Objectives
- **Purpose**: Define clear, measurable goals
- **Format**: Checkbox list
- **Guidelines**: Use SMART criteria (Specific, Measurable, Achievable, Relevant, Time-bound)

```markdown
## Objectives

- [ ] Implement core functionality with 90% test coverage
- [ ] Update documentation with usage examples
- [ ] Deploy to staging environment
- [ ] Verify production readiness
```

---

### Context-Specific Sections

#### For Bug Reports

```markdown
## Bug Description
[Clear description of the issue]

## Impact
- 🔴 **Severity**: Critical | High | Medium | Low
- 👥 **Affected Users**: [number or percentage]
- 📊 **Data Loss Risk**: Yes | No
- 🔍 **Discoverability**: [How was it found]

## Root Cause Analysis
[Technical explanation of why the bug occurs]

## Reproduction Steps
1. [Step 1]
2. [Step 2]
3. [Step 3]

**Expected Behavior**: [What should happen]
**Actual Behavior**: [What actually happens]

## Environment
- **Version**: vX.Y.Z
- **Node.js**: vX.Y.Z
- **OS**: Operating system
- **Additional Context**: [Any relevant details]

## Solution Implemented
[Detailed explanation of the fix]

## Testing
- **Test Files**: [paths to test files]
- **Coverage**: X% (target: 90%+)
- **Manual Testing**: [scenarios tested]

## Verification
[Steps to verify the fix works]
```

---

#### For Feature Requests

```markdown
## Feature Description
[Clear, detailed description of the proposed feature]

## Use Cases
1. **Use Case 1**: [Description]
   - **Actor**: [Who uses this]
   - **Action**: [What they do]
   - **Outcome**: [Expected result]

2. **Use Case 2**: [Description]

## Acceptance Criteria
- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

## Implementation Approach

### Design
- [ ] Architecture design
- [ ] API design
- [ ] UI/UX mockups (if applicable)

### Development
- [ ] Core implementation
- [ ] Integration with existing features
- [ ] Performance optimization

### Testing
- [ ] Unit tests (90%+ coverage)
- [ ] Integration tests
- [ ] User acceptance testing

## Performance Impact
- **Expected Load**: [metrics]
- **Resource Requirements**: [CPU, memory, etc.]
- **Scalability**: [How it scales]
```

---

#### For Integration Tasks

```markdown
## Integration Overview
[High-level description of what's being integrated]

## Integration Areas

### Dependencies
- [ ] Package.json updates
- [ ] Version compatibility checks
- [ ] Import statement verification

### Functionality
- [ ] Core feature integration
- [ ] API compatibility
- [ ] Data flow validation

### Testing
- [ ] Unit tests for each component
- [ ] Integration tests for data flow
- [ ] End-to-end validation

## Integration Challenges
[Known challenges and mitigation strategies]

## Migration Path
[How to migrate from current state to integrated state]
```

---

#### For Swarm Tasks

```markdown
## Swarm Configuration

### Topology
**Selected**: mesh | hierarchical | ring | star
**Reason**: [Why this topology was chosen]

### Agent Assignment
- **Coordinator**: Overall progress tracking
- **Analyst**: Technical validation and analysis
- **Coder**: Implementation
- **Tester**: Quality assurance
- **Documenter**: Documentation updates

### Task Decomposition
1. **Task 1** (Priority: High, Agent: coder)
   - [ ] Subtask 1.1
   - [ ] Subtask 1.2

2. **Task 2** (Priority: Medium, Agent: tester)
   - [ ] Subtask 2.1
   - [ ] Subtask 2.2

## Coordination Protocol
- **Pre-hooks**: [Setup operations]
- **During**: [Progress tracking]
- **Post-hooks**: [Cleanup and reporting]

## Progress Tracking
[Automated updates will be posted by swarm agents]
```

---

## Naming Conventions

### Title Conventions

**Format**: `[TYPE] Action Verb + Component + Specific Detail`

**Examples by Type**:
- **Bug**: `[BUG] Swarm Coordinator Fails on Large Topologies`
- **Feature**: `[FEATURE] Add Real-time Collaboration Support`
- **Enhancement**: `[ENHANCEMENT] Optimize Memory Usage in Pattern Store`
- **Docs**: `[DOCS] Add Getting Started Guide for New Users`
- **Integration**: `[INTEGRATION] Connect AgentDB with ReasoningBank`
- **Incident**: `🔄 Rollback Incident: v2.7.0 Memory Leak`
- **Swarm**: `[SWARM] Multi-Agent Code Review Pipeline`

### Branch Naming

When creating branches from issues:

```bash
# Format: type/issue-description-session-id
git checkout -b claude/issue-bug-pattern-persistence-011CUQtutJdEPu3jTnB7QzRH
```

**Pattern**: `claude/issue-<type>-<brief-description>-<session-id>`

---

## Labeling Strategy

### Standard Labels

#### By Type
- `bug` - Code defects
- `enhancement` - Improvements to existing features
- `feature` - New functionality
- `documentation` - Documentation updates
- `integration` - Integration tasks
- `incident` - Critical incidents
- `swarm-task` - Multi-agent coordination

#### By Priority
- `critical` - Immediate attention required
- `high-priority` - Important, schedule soon
- `medium-priority` - Normal priority
- `low-priority` - Nice to have

#### By Status
- `in-progress` - Currently being worked on
- `blocked` - Waiting on dependencies
- `needs-review` - Ready for review
- `ready-for-merge` - Approved and ready
- `stale` - Inactive for 30+ days

#### By Area
- `mcp` - MCP server related
- `neural` - Neural pattern features
- `github` - GitHub integration
- `swarm` - Swarm coordination
- `hooks` - Hook system
- `memory` - Memory management
- `performance` - Performance related

#### By Agent Type
- `swarm-debugger` - Requires debugger agent
- `swarm-feature` - Requires feature agents
- `swarm-optimizer` - Requires optimizer agent
- `swarm-ready` - Ready for swarm processing

### Label Combinations

**Examples**:
- `bug, critical, mcp, high-priority` - Critical MCP bug
- `feature, enhancement, swarm, swarm-ready` - Swarm feature enhancement
- `documentation, low-priority, needs-review` - Docs needing review

---

## Dos and Don'ts

### ✅ DO

1. **DO write clear, descriptive titles**
   - Good: `[BUG] Memory leak in SwarmCoordinator after 100+ agents`
   - Reasoning: Specific, searchable, includes component and symptom

2. **DO include reproduction steps for bugs**
   - Numbered steps with expected vs actual behavior
   - Include environment details
   - Add minimal code examples

3. **DO break down complex tasks**
   - Use checkbox lists for subtasks
   - Assign priorities to each subtask
   - Track progress with updates

4. **DO link related issues and PRs**
   - Use `Relates to #123`, `Depends on #456`, `Blocks #789`
   - Create relationships for dependency tracking
   - Update when new connections discovered

5. **DO use consistent formatting**
   - Follow template structure
   - Use standard emoji conventions
   - Apply proper markdown syntax

6. **DO include acceptance criteria**
   - Define clear success metrics
   - Specify test coverage requirements
   - Set performance targets

7. **DO update progress regularly**
   - Post comments with status updates
   - Update checkboxes as tasks complete
   - Notify on blockers or changes

8. **DO assign appropriate labels**
   - Apply type, priority, and area labels
   - Update labels as status changes
   - Use swarm-related labels when applicable

9. **DO consider swarm coordination**
   - Identify if multi-agent approach is beneficial
   - Specify topology and agent types
   - Include coordination protocol

10. **DO provide complete context**
    - Include relevant background information
    - Link to related documentation
    - Add screenshots or examples when helpful

---

### ❌ DON'T

1. **DON'T use vague titles**
   - Bad: `Fix bug`
   - Why: Not searchable, no context, no component identified

2. **DON'T skip environment details for bugs**
   - Bad: "It doesn't work"
   - Why: Cannot reproduce, waste time investigating

3. **DON'T create duplicate issues**
   - Search existing issues first
   - Comment on existing issue if related
   - Use "Duplicate of #123" and close

4. **DON'T mix multiple unrelated issues**
   - Bad: "Fix bug X and add feature Y"
   - Why: Difficult to track, impossible to close properly

5. **DON'T use issues for questions**
   - Use GitHub Discussions for questions
   - Use issues for actionable items only
   - Convert discussions to issues when actionable

6. **DON'T forget to update status**
   - Update checkboxes as you progress
   - Post blockers immediately
   - Close when complete

7. **DON'T skip testing requirements**
   - Always specify expected test coverage
   - Include manual testing steps
   - Verify before closing

8. **DON'T ignore related work**
   - Link to related PRs
   - Reference prior art
   - Acknowledge dependencies

9. **DON'T use informal language**
   - Be professional and clear
   - Use technical terminology appropriately
   - Avoid ambiguous phrasing

10. **DON'T leave orphaned issues**
    - Close completed issues
    - Archive stale issues
    - Maintain issue hygiene

---

## Swarm Coordination Patterns

### When to Use Swarm Coordination

**Complex Multi-Step Tasks** (3+ major steps)
**Parallel Execution Opportunities** (independent subtasks)
**Multiple Skill Requirements** (different agent types needed)
**Large-Scale Operations** (affecting multiple components)

### Swarm Topology Selection

#### Mesh Topology
**Best For**: Parallel, independent tasks
**Example**: Feature development with frontend, backend, and testing in parallel

```markdown
## Swarm Configuration
**Topology**: mesh
**Max Agents**: 6
**Coordination**: Peer-to-peer with shared memory

### Agents
- Frontend Developer (agent-1)
- Backend Developer (agent-2)
- Database Architect (agent-3)
- Test Engineer (agent-4)
- Security Auditor (agent-5)
- Documenter (agent-6)
```

#### Hierarchical Topology
**Best For**: Coordinated workflows with dependencies
**Example**: Release management with stage gates

```markdown
## Swarm Configuration
**Topology**: hierarchical
**Max Agents**: 8
**Coordination**: Queen-led with worker specialization

### Hierarchy
- **Queen (Coordinator)**: Overall orchestration
  - **Worker (Tester)**: Test execution
  - **Worker (Builder)**: Build and package
  - **Worker (Deployer)**: Deployment operations
  - **Worker (Monitor)**: Health checks
```

#### Ring Topology
**Best For**: Sequential processing with handoffs
**Example**: CI/CD pipeline stages

```markdown
## Swarm Configuration
**Topology**: ring
**Max Agents**: 4
**Coordination**: Sequential with state passing

### Processing Order
1. **Linter** → Validates code style
2. **Tester** → Runs test suite
3. **Builder** → Creates artifacts
4. **Deployer** → Pushes to staging
```

#### Star Topology
**Best For**: Centralized coordination with distributed workers
**Example**: Issue triage and routing

```markdown
## Swarm Configuration
**Topology**: star
**Max Agents**: 5
**Coordination**: Central hub with specialized agents

### Structure
- **Hub (Triager)**: Route issues to specialists
  - **Bug Specialist**: Handle bug reports
  - **Feature Specialist**: Process feature requests
  - **Docs Specialist**: Manage documentation
  - **Integration Specialist**: Coordinate integrations
```

### Agent Assignment Guidelines

**Coordinator/Queen**: Overall orchestration and progress tracking
**Analyst**: Technical analysis, code review, performance profiling
**Coder/Developer**: Implementation work
**Tester/QA**: Test creation and validation
**Security**: Security audits and vulnerability scanning
**Documenter**: Documentation creation and updates
**Debugger**: Bug investigation and root cause analysis
**Optimizer**: Performance optimization
**Architect**: System design and architecture

### Swarm Hooks Integration

```markdown
## Coordination Protocol

### Pre-Task Hooks
\`\`\`bash
# Initialize swarm environment
npx claude-flow@alpha hooks pre-task \\
  --description "Feature: OAuth2 Integration" \\
  --session-id "swarm-oauth-integration"

# Restore prior context
npx claude-flow@alpha hooks session-restore \\
  --session-id "swarm-oauth-integration"
\`\`\`

### During Task
\`\`\`bash
# Track progress
npx claude-flow@alpha hooks notify \\
  --message "Completed OAuth provider integration" \\
  --memory-key "swarm/oauth/progress"

# Store intermediate results
npx claude-flow@alpha hooks post-edit \\
  --file "src/auth/oauth.js" \\
  --memory-key "swarm/oauth/implementation"
\`\`\`

### Post-Task Hooks
\`\`\`bash
# Finalize and report
npx claude-flow@alpha hooks post-task \\
  --task-id "oauth-integration" \\
  --export-metrics true

# Store completion state
npx claude-flow@alpha hooks session-end \\
  --session-id "swarm-oauth-integration" \\
  --export-path "./swarm-reports"
\`\`\`
```

---

## Automation & Integration

### GitHub Actions Integration

```yaml
# .github/workflows/issue-automation.yml
name: Issue Automation
on:
  issues:
    types: [opened, labeled, edited]

jobs:
  auto-triage:
    runs-on: ubuntu-latest
    steps:
      - name: Analyze Issue
        if: contains(github.event.issue.labels.*.name, 'swarm-ready')
        run: |
          ISSUE_DATA=$(gh issue view ${{ github.event.issue.number }} \\
            --json title,body,labels)

          npx ruv-swarm github issue-analyze \\
            --issue-data "$ISSUE_DATA" \\
            --suggest-agents \\
            --auto-decompose

      - name: Add Progress Tracking
        run: |
          gh issue comment ${{ github.event.issue.number }} \\
            --body "🐝 Swarm coordination initialized for this issue"
```

### Claude Code Command Integration

Create custom command for issue management:

```markdown
# .claude/commands/github/create-issue.md

Create a comprehensive GitHub issue with swarm coordination.

## Usage
Create an issue of type: bug | feature | enhancement | docs | integration

## Process
1. Gather requirements from user
2. Select appropriate template
3. Fill in all required sections
4. Suggest labels based on content
5. Recommend swarm topology if complex
6. Create issue via gh CLI
7. Initialize swarm if needed

## Example
\`\`\`bash
gh issue create \\
  --title "[FEATURE] Real-time Collaboration" \\
  --body "$(cat issue-body.md)" \\
  --label "feature,high-priority,swarm-ready"
\`\`\`
```

### Claude Code Skill Integration

Leverage the `github-project-management` skill:

```bash
# Activate the skill to access comprehensive issue management
# The skill provides automated:
# - Issue creation with templates
# - Progress tracking
# - Swarm coordination
# - Project board sync
```

---

## Quality Checklist

### Before Creating an Issue

- [ ] Searched existing issues for duplicates
- [ ] Selected appropriate issue type and template
- [ ] Written clear, descriptive title (50-80 chars)
- [ ] Included comprehensive overview/summary
- [ ] Defined specific, measurable objectives
- [ ] Added all required context sections
- [ ] Included reproduction steps (if bug)
- [ ] Specified acceptance criteria
- [ ] Applied appropriate labels
- [ ] Linked related issues/PRs
- [ ] Considered swarm coordination needs
- [ ] Added priority and severity indicators
- [ ] Included environment details (if applicable)
- [ ] Proofread for clarity and completeness

### During Issue Lifecycle

- [ ] Updated status regularly via comments
- [ ] Checked off completed objectives
- [ ] Posted blockers immediately
- [ ] Updated labels as status changes
- [ ] Responded to questions promptly
- [ ] Linked PRs when created
- [ ] Updated swarm progress
- [ ] Maintained professional communication

### Before Closing an Issue

- [ ] All objectives completed
- [ ] Tests written and passing
- [ ] Documentation updated
- [ ] PR merged (if applicable)
- [ ] Verified in production/staging
- [ ] Posted completion summary
- [ ] Linked final PR(s)
- [ ] Updated related issues
- [ ] Exported any relevant metrics
- [ ] Archived swarm coordination data

---

## Templates Reference

All templates are available in `./templates/`:

- [bug-report.md](./templates/bug-report.md) - Bug reports with root cause analysis
- [feature-request.md](./templates/feature-request.md) - New feature proposals
- [enhancement.md](./templates/enhancement.md) - Improvements to existing features
- [documentation.md](./templates/documentation.md) - Documentation updates
- [integration-task.md](./templates/integration-task.md) - Complex integrations
- [rollback-incident.md](./templates/rollback-incident.md) - Incident reports
- [swarm-task.md](./templates/swarm-task.md) - Multi-agent coordination tasks

---

## Additional Resources

- [GitHub Project Management Skill](../../.claude/skills/github-project-management/SKILL.md)
- [Issue Tracker Agent](../../.claude/agents/github/issue-tracker.md)
- [Swarm Issue Agent](../../.claude/agents/github/swarm-issue.md)
- [GitHub Workflow Automation](../../.claude/skills/github-workflow-automation/SKILL.md)

---

**Version**: 1.0.0
**Last Updated**: 2025-10-23
**Maintainer**: Claude Code
