# GitHub Issue Best Practices

Comprehensive guide and templates for creating, managing, and organizing GitHub issues in ai-claude-flow with Claude Code integration.

## 📚 Documentation Index

### Core Documentation

- **[ISSUE-BEST-PRACTICES.md](./ISSUE-BEST-PRACTICES.md)** - Complete best practices guide
  - Issue types and categories
  - Structure and formatting guidelines
  - Naming conventions and labeling strategy
  - Dos and don'ts
  - Swarm coordination patterns
  - Quality checklist

### Templates

All templates are located in `./templates/`:

| Template | Use Case | Complexity |
|----------|----------|------------|
| [bug-report.md](./templates/bug-report.md) | Bug defects and errors | Medium |
| [feature-request.md](./templates/feature-request.md) | New features and capabilities | High |
| [enhancement.md](./templates/enhancement.md) | Improvements to existing features | Medium |
| [documentation.md](./templates/documentation.md) | Documentation updates | Low |
| [integration-task.md](./templates/integration-task.md) | Complex multi-component integrations | High |
| [rollback-incident.md](./templates/rollback-incident.md) | Incident reports and rollbacks | High |
| [swarm-task.md](./templates/swarm-task.md) | Multi-agent coordination tasks | High |

### Integration Guides

- **[integration/CLAUDE-CODE-INTEGRATION.md](./integration/CLAUDE-CODE-INTEGRATION.md)** - Claude Code integration
  - Plugin integration examples
  - Command integration patterns
  - Skill usage guidelines
  - Automation workflows

---

## 🚀 Quick Start

### Creating Your First Issue

1. **Choose the right template** based on your need:
   - Bug? Use `bug-report.md`
   - New feature? Use `feature-request.md`
   - Improvement? Use `enhancement.md`

2. **Follow the template structure**:
   - Fill in all required sections
   - Use checkboxes for task tracking
   - Apply appropriate labels
   - Consider swarm coordination for complex tasks

3. **Create the issue**:
   ```bash
   gh issue create \
     --title "[TYPE] Clear, Descriptive Title" \
     --body "$(cat filled-template.md)" \
     --label "type,priority,area"
   ```

### Using Claude Code Skills

Leverage the GitHub Project Management skill:

```bash
# The skill is automatically available in Claude Code
# Simply reference it in your request:
# "Create a comprehensive issue for [feature] using best practices"
```

### Swarm Coordination

For complex issues requiring multi-agent coordination:

```bash
# Initialize swarm
npx claude-flow@alpha hooks pre-task \
  --description "Issue: [title]" \
  --session-id "issue-[number]"

# During work
npx claude-flow@alpha hooks notify \
  --message "Completed [milestone]" \
  --memory-key "issue/[number]/progress"

# Finalize
npx claude-flow@alpha hooks post-task \
  --task-id "issue-[number]"
```

---

## 📋 Issue Type Quick Reference

### 🐛 Bug Reports
- **When**: Code defects, errors, unexpected behavior
- **Template**: [bug-report.md](./templates/bug-report.md)
- **Key Sections**: Root cause, reproduction steps, solution
- **Example**: `[BUG] MCP Pattern Store Not Persisting Data`

### ✨ Feature Requests
- **When**: New functionality or capabilities
- **Template**: [feature-request.md](./templates/feature-request.md)
- **Key Sections**: Use cases, implementation approach, acceptance criteria
- **Example**: `[FEATURE] Real-time Collaboration with WebSockets`

### 🔧 Enhancements
- **When**: Improvements to existing features
- **Template**: [enhancement.md](./templates/enhancement.md)
- **Key Sections**: Current limitations, proposed improvements, performance impact
- **Example**: `[ENHANCEMENT] Optimize Swarm Coordination Performance`

### 📚 Documentation
- **When**: Documentation updates, additions, corrections
- **Template**: [documentation.md](./templates/documentation.md)
- **Key Sections**: Documentation type, structure, examples
- **Example**: `[DOCS] Update Integration Guides for v2.7.1`

### 🔄 Integration Tasks
- **When**: Complex multi-component integrations
- **Template**: [integration-task.md](./templates/integration-task.md)
- **Key Sections**: Components, integration areas, testing strategy
- **Example**: `[INTEGRATION] Complete Claude-Flow and Ruv-Swarm Integration`

### 🚨 Incident Reports
- **When**: Production issues, rollbacks, critical failures
- **Template**: [rollback-incident.md](./templates/rollback-incident.md)
- **Key Sections**: Timeline, root cause, prevention measures
- **Example**: `🔄 Rollback Incident: v2.7.0 Deployment Failure`

### 🐝 Swarm Tasks
- **When**: Multi-agent coordination required
- **Template**: [swarm-task.md](./templates/swarm-task.md)
- **Key Sections**: Swarm configuration, agent assignment, coordination protocol
- **Example**: `[SWARM] Multi-Repo Version Alignment and Synchronization`

---

## 🏷️ Labeling Strategy

### By Type
- `bug`, `enhancement`, `feature`, `documentation`, `integration`, `incident`, `swarm-task`

### By Priority
- `critical`, `high-priority`, `medium-priority`, `low-priority`

### By Status
- `in-progress`, `blocked`, `needs-review`, `ready-for-merge`, `stale`

### By Area
- `mcp`, `neural`, `github`, `swarm`, `hooks`, `memory`, `performance`

### By Agent Type
- `swarm-debugger`, `swarm-feature`, `swarm-optimizer`, `swarm-ready`

---

## 🎯 Best Practices Summary

### ✅ DO

1. Write clear, descriptive titles with type prefix
2. Include complete reproduction steps for bugs
3. Break down complex tasks with checkboxes
4. Link related issues and PRs
5. Use consistent formatting and emojis
6. Define clear acceptance criteria
7. Update progress regularly
8. Apply appropriate labels
9. Consider swarm coordination for complexity
10. Provide complete context

### ❌ DON'T

1. Use vague titles
2. Skip environment details for bugs
3. Create duplicate issues
4. Mix multiple unrelated issues
5. Use issues for questions
6. Forget to update status
7. Skip testing requirements
8. Ignore related work
9. Use informal language
10. Leave orphaned issues

---

## 🐝 Swarm Coordination Guide

### When to Use Swarm

- **Complex Multi-Step Tasks** (3+ major steps)
- **Parallel Execution Opportunities** (independent subtasks)
- **Multiple Skill Requirements** (different agent types)
- **Large-Scale Operations** (multiple components)

### Topology Selection

| Topology | Best For | Example Use Case |
|----------|----------|------------------|
| **Mesh** | Parallel, independent tasks | Feature development with frontend, backend, testing in parallel |
| **Hierarchical** | Coordinated workflows with dependencies | Release management with stage gates |
| **Ring** | Sequential processing with handoffs | CI/CD pipeline stages |
| **Star** | Centralized coordination with distributed workers | Issue triage and routing |

### Agent Types

- **Coordinator/Queen**: Overall orchestration
- **Analyst**: Technical analysis and code review
- **Coder/Developer**: Implementation work
- **Tester/QA**: Test creation and validation
- **Security**: Security audits
- **Documenter**: Documentation creation
- **Debugger**: Bug investigation
- **Optimizer**: Performance optimization
- **Architect**: System design

---

## 📊 Quality Checklist

### Before Creating an Issue

- [ ] Searched existing issues for duplicates
- [ ] Selected appropriate template
- [ ] Written clear, descriptive title (50-80 chars)
- [ ] Included comprehensive overview
- [ ] Defined specific objectives
- [ ] Added all required sections
- [ ] Applied appropriate labels
- [ ] Considered swarm coordination
- [ ] Proofread for clarity

### During Issue Lifecycle

- [ ] Updated status regularly
- [ ] Checked off completed objectives
- [ ] Posted blockers immediately
- [ ] Updated labels as needed
- [ ] Responded to questions
- [ ] Linked PRs when created
- [ ] Maintained professional communication

### Before Closing

- [ ] All objectives completed
- [ ] Tests written and passing
- [ ] Documentation updated
- [ ] PR merged (if applicable)
- [ ] Verified in production/staging
- [ ] Posted completion summary

---

## 🔗 Related Resources

### Internal Documentation
- [GitHub Project Management Skill](../../.claude/skills/github-project-management/SKILL.md)
- [Issue Tracker Agent](../../.claude/agents/github/issue-tracker.md)
- [Swarm Issue Agent](../../.claude/agents/github/swarm-issue.md)
- [GitHub Workflow Automation](../../.claude/skills/github-workflow-automation/SKILL.md)

### External Resources
- [GitHub Issues Documentation](https://docs.github.com/en/issues)
- [GitHub CLI Manual](https://cli.github.com/manual/)
- [Claude Flow Documentation](https://github.com/ruvnet/claude-flow)
- [Ruv Swarm Documentation](https://github.com/ruvnet/ruv-swarm)

---

## 🤝 Contributing

To improve these best practices and templates:

1. **Propose Changes**: Open an issue with your suggestions
2. **Submit Updates**: Create a PR with template improvements
3. **Share Examples**: Contribute real-world examples
4. **Report Issues**: Let us know if something isn't clear

---

## 📜 Version History

- **v1.0.0** (2025-10-23): Initial comprehensive best-practices documentation
  - 7 issue templates
  - Complete best-practices guide
  - Claude Code integration guide
  - Swarm coordination patterns

---

**Maintained By**: Claude Code
**Last Updated**: 2025-10-23
**Repository**: [ai-claude-flow](https://github.com/fontestad-ai/ai-claude-flow)
