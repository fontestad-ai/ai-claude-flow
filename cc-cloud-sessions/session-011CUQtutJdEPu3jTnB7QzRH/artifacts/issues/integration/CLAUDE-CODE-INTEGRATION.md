# Claude Code Integration Guide for Issue Management

## Table of Contents

1. [Overview](#overview)
2. [Integration with Claude Code Plugins](#integration-with-claude-code-plugins)
3. [Integration with Claude Code Commands](#integration-with-claude-code-commands)
4. [Integration with Claude Code Skills](#integration-with-claude-code-skills)
5. [Automation Examples](#automation-examples)
6. [Best Practices](#best-practices)

---

## Overview

This guide explains how to leverage Claude Code plugins, commands, and skills to implement the issue best practices defined in [ISSUE-BEST-PRACTICES.md](../ISSUE-BEST-PRACTICES.md).

### Key Integration Points

1. **Plugins**: Extend Claude Code functionality for issue management
2. **Commands**: Automate common issue operations
3. **Skills**: Comprehensive workflows for GitHub project management
4. **MCP Tools**: Deep integration with ruv-swarm and claude-flow

---

## Integration with Claude Code Plugins

### What Are Claude Code Plugins?

Claude Code plugins extend the core functionality of Claude Code. They can:
- Add new MCP tools
- Enhance existing workflows
- Provide specialized capabilities

### Issue Management Plugin Example

**Location**: `.claude-plugin/`

#### Plugin Structure

```
.claude-plugin/
├── package.json
├── README.md
├── src/
│   ├── tools/
│   │   ├── issue-create.js
│   │   ├── issue-triage.js
│   │   ├── issue-track.js
│   │   └── swarm-coordinate.js
│   └── index.js
└── docs/
    └── INSTALLATION.md
```

#### Example Plugin Implementation

**`.claude-plugin/src/tools/issue-create.js`**

```javascript
/**
 * Claude Code Plugin: Issue Creator
 * Implements best-practice issue creation with template selection
 */

export class IssueCreator {
  constructor(config) {
    this.config = config;
    this.templates = this.loadTemplates();
  }

  /**
   * Load issue templates from docs/best-practices/issues/templates/
   */
  loadTemplates() {
    const templatesPath = path.join(
      __dirname,
      '../../../docs/best-practices/issues/templates'
    );

    return {
      bug: fs.readFileSync(path.join(templatesPath, 'bug-report.md'), 'utf8'),
      feature: fs.readFileSync(path.join(templatesPath, 'feature-request.md'), 'utf8'),
      enhancement: fs.readFileSync(path.join(templatesPath, 'enhancement.md'), 'utf8'),
      docs: fs.readFileSync(path.join(templatesPath, 'documentation.md'), 'utf8'),
      integration: fs.readFileSync(path.join(templatesPath, 'integration-task.md'), 'utf8'),
      incident: fs.readFileSync(path.join(templatesPath, 'rollback-incident.md'), 'utf8'),
      swarm: fs.readFileSync(path.join(templatesPath, 'swarm-task.md'), 'utf8'),
    };
  }

  /**
   * Create issue with intelligent template selection
   */
  async createIssue(options) {
    const { type, title, details, swarmReady = false } = options;

    // Select appropriate template
    const template = this.templates[type];
    if (!template) {
      throw new Error(`Unknown issue type: ${type}`);
    }

    // Fill in template with provided details
    const issueBody = this.fillTemplate(template, details);

    // Suggest labels based on content
    const labels = this.suggestLabels(type, details, swarmReady);

    // Create issue via gh CLI
    const command = [
      'gh', 'issue', 'create',
      '--title', `"${title}"`,
      '--body', `"${issueBody}"`,
      '--label', labels.join(',')
    ];

    if (swarmReady) {
      // Initialize swarm coordination for complex issues
      await this.initializeSwarm(details);
    }

    return execSync(command.join(' '), { encoding: 'utf8' });
  }

  /**
   * Intelligent label suggestion based on content analysis
   */
  suggestLabels(type, details, swarmReady) {
    const labels = [type];

    // Analyze content for additional labels
    const content = JSON.stringify(details).toLowerCase();

    // Priority labels
    if (content.includes('critical') || content.includes('urgent')) {
      labels.push('critical');
    } else if (content.includes('high priority')) {
      labels.push('high-priority');
    }

    // Area labels
    const areaKeywords = {
      mcp: ['mcp', 'server', 'protocol'],
      neural: ['neural', 'pattern', 'training'],
      swarm: ['swarm', 'agent', 'coordination'],
      github: ['github', 'repository', 'pr', 'issue'],
    };

    Object.entries(areaKeywords).forEach(([area, keywords]) => {
      if (keywords.some(kw => content.includes(kw))) {
        labels.push(area);
      }
    });

    // Swarm labels
    if (swarmReady) {
      labels.push('swarm-ready');
    }

    return labels;
  }

  /**
   * Initialize swarm coordination for complex issues
   */
  async initializeSwarm(details) {
    const { topology = 'mesh', maxAgents = 4 } = details.swarmConfig || {};

    // Initialize swarm using MCP tools
    await this.mcp.call('swarm_init', {
      topology,
      maxAgents
    });

    // Store issue context in swarm memory
    await this.mcp.call('memory_usage', {
      action: 'store',
      key: `issue/${details.number}/context`,
      value: JSON.stringify(details)
    });
  }
}
```

#### Plugin Registration

**`.claude-plugin/src/index.js`**

```javascript
import { IssueCreator } from './tools/issue-create.js';
import { IssueTriage } from './tools/issue-triage.js';
import { IssueTracker } from './tools/issue-track.js';

export default {
  name: 'issue-management',
  version: '1.0.0',
  description: 'Comprehensive issue management with best practices',

  tools: [
    {
      name: 'issue_create',
      description: 'Create issue with best-practice templates',
      handler: IssueCreator,
    },
    {
      name: 'issue_triage',
      description: 'Intelligent issue triage and labeling',
      handler: IssueTriage,
    },
    {
      name: 'issue_track',
      description: 'Track issue progress with swarm coordination',
      handler: IssueTracker,
    },
  ],

  async initialize(context) {
    // Plugin initialization
    console.log('Issue Management Plugin initialized');
    return {
      templatesLoaded: true,
      swarmEnabled: true,
    };
  },
};
```

---

## Integration with Claude Code Commands

### What Are Claude Code Commands?

Commands are markdown files in `.claude/commands/` that provide reusable workflows.

### Issue Management Commands

#### Command: Create Issue

**Location**: `.claude/commands/github/create-issue.md`

```markdown
# create-issue

Create a comprehensive GitHub issue using best-practice templates.

## Usage

Create an issue by type:
- **bug**: Bug reports with root cause analysis
- **feature**: Feature requests with use cases
- **enhancement**: Improvements to existing features
- **docs**: Documentation updates
- **integration**: Complex integrations
- **incident**: Rollback incidents
- **swarm**: Multi-agent coordination tasks

## Process

1. **Gather Requirements**
   - Ask user for issue type
   - Request title and description
   - Determine if swarm coordination needed

2. **Select Template**
   - Load appropriate template from docs/best-practices/issues/templates/
   - Guide user through required sections

3. **Analyze Content**
   - Suggest labels based on content analysis
   - Recommend priority level
   - Identify related issues

4. **Create Issue**
   \`\`\`bash
   gh issue create \\
     --title "[TYPE] Clear Title" \\
     --body "$(cat filled-template.md)" \\
     --label "type,priority,area"
   \`\`\`

5. **Initialize Swarm** (if complex)
   \`\`\`bash
   npx claude-flow@alpha hooks pre-task \\
     --description "Issue #123: [title]" \\
     --session-id "issue-123"
   \`\`\`

6. **Return Issue URL**
   Provide user with issue URL and next steps

## Examples

### Example 1: Bug Report
\`\`\`
User: "Create a bug report for memory leak in SwarmCoordinator"