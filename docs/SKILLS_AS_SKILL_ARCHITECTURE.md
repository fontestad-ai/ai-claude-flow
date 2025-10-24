# Skills-as-a-Skill Architecture
## Self-Hosting Meta-Skill for Claude Code

**Created:** 2025-10-23
**Version:** 1.0.0
**Purpose:** Implement the Skills System itself as a Claude Skill, eliminating claude-flow dependencies

---

## Executive Summary

This document provides a comprehensive technical architecture for implementing the **Skills Management System as a Claude Skill** - a meta-skill that creates, validates, deploys, and manages other skills, including itself. This eliminates all dependencies on claude-flow's skill deployment infrastructure.

**The Challenge:**
> "How do you use a skills system to manage skills when the skills system itself is a skill?"

This is a **bootstrap problem** requiring careful architectural design to achieve self-hosting while maintaining simplicity for initial deployment.

**Key Innovations:**
- ✅ **Self-Hosting**: Meta-skill manages itself after bootstrap
- ✅ **Zero Dependencies**: No claude-flow copier or deployment tools
- ✅ **Bootstrap Script**: Minimal one-time setup
- ✅ **Claude Code Native**: Pure integration with Claude's skill discovery
- ✅ **Template System**: Skill generation from templates
- ✅ **Validation Engine**: YAML frontmatter and structure checking

---

## Current Skills Implementation Analysis

### 1. How claude-flow Implements Skills

#### **Skill Discovery (Claude Code Native)**
```
Process:
1. Claude Code scans ~/.claude/skills/ and .claude/skills/
2. Reads YAML frontmatter from */SKILL.md files
3. Loads name + description into system prompt
4. When triggered, loads full SKILL.md body
5. Referenced files loaded on-demand

Location Hierarchy:
~/.claude/skills/          # Personal skills (global)
.claude/skills/            # Project skills (local)
```

#### **Skill Deployment (claude-flow Specific)**
```javascript
// src/cli/simple-commands/init/skills-copier.js
export async function copySkillFiles(targetDir, options = {}) {
  // Source: claude-flow package's .claude/skills/
  const packageSkillsDir = join(__dirname, '../../../../.claude/skills');

  // Target: User project .claude/skills/
  const targetSkillsDir = join(targetDir, '.claude/skills');

  // Copy all SKILL.md files recursively
  await copyRecursive(sourceSkillsDir, targetSkillsDir);
}
```

**Dependencies:**
- ❌ claude-flow package structure
- ❌ Node.js file copying logic
- ❌ Path resolution to find skill source
- ❌ Recursive directory traversal
- ❌ Validation after copying

#### **Current Skill Catalog**
25 skills totaling ~26,500 lines:
```
.claude/skills/
├── skill-builder/                  # Meta-skill (910 lines)
├── sparc-methodology/              # TDD methodology (1,230 lines)
├── pair-programming/               # Collaboration (1,180 lines)
├── agentdb-*/                      # Memory & learning (6 skills)
├── github-*/                       # GitHub integration (5 skills)
├── swarm-*/                        # Multi-agent coordination (3 skills)
├── flow-nexus-*/                   # Cloud platform (3 skills)
├── hooks-automation/               # Automation framework
├── verification-quality/           # Quality assurance
└── [+10 more skills]
```

### 2. Skill Structure Specification

#### **YAML Frontmatter (REQUIRED)**
```yaml
---
name: "Skill Name"                   # Max 64 chars
description: "What this skill does   # Max 1024 chars
and when Claude should use it."      # MUST include what + when
---
```

#### **Progressive Disclosure (3 Levels)**
```
Level 1: YAML frontmatter (~60 bytes)
  ↓ Loaded at Claude Code startup
  ↓ Enables skill matching

Level 2: SKILL.md body (~1-10KB)
  ↓ Loaded when skill is triggered
  ↓ Contains main instructions

Level 3: Referenced files (variable size)
  ↓ Loaded on-demand as Claude navigates
  ↓ Templates, examples, advanced docs
```

#### **Directory Structure**
```
skill-name/
├── SKILL.md                # REQUIRED: Main skill file
├── README.md               # Optional: Human-readable docs
├── scripts/                # Optional: Executable scripts
│   ├── create.js
│   ├── validate.js
│   └── deploy.sh
├── resources/              # Optional: Supporting files
│   ├── templates/
│   │   ├── basic-skill.md
│   │   └── advanced-skill.md
│   └── schemas/
│       └── yaml-schema.json
└── docs/                   # Optional: Additional documentation
    ├── ADVANCED.md
    └── API_REFERENCE.md
```

### 3. Dependencies to Eliminate

#### **A. skill-copier.js (216 lines)**
```javascript
// DEPENDENCIES:
- Package path resolution (join(__dirname, '../../../../.claude/skills'))
- Recursive file copying logic
- File system operations (fs.readdir, fs.readFile, fs.writeFile)
- Directory creation (fs.mkdir recursive)
- File existence checking (fs.access)
- Validation after copying

// FUNCTIONALITY TO REPLACE:
✅ copySkillFiles() → skill-deploy.js script
✅ createSkillDirectories() → skill-init.js script
✅ validateSkillSystem() → skill-validate.js script
```

#### **B. Skill Builder (Current Meta-Skill)**
```
Location: .claude/skills/skill-builder/SKILL.md
Size: 910 lines
Purpose: Provides instructions for creating skills

LIMITATIONS:
- Documentation only (no code execution)
- No automation
- No validation
- No template generation
- Requires manual file creation

FUNCTIONALITY TO ADD:
✅ Automated skill creation
✅ Template-based generation
✅ YAML validation
✅ Structure verification
✅ Script generation
```

#### **C. claude-flow init command**
```bash
# Current deployment:
npx claude-flow@alpha init --force

# Internally calls:
- skills-copier.copySkillFiles()
- Copies from package location
- Requires claude-flow to be installed

FUNCTIONALITY TO REPLACE:
✅ Standalone bootstrap script
✅ Self-contained deployment
✅ No package dependency
```

---

## Meta-Skill Architecture: "Skill Management"

### 1. Core Philosophy

**Self-Hosting Principle:**
> The skill-management skill should be able to create, update, and maintain itself after initial bootstrap.

**Bootstrap Minimalism:**
> The initial deployment should require the absolute minimum - a single bootstrap script.

**Claude Code Native:**
> All operations should use Claude Code's native skill discovery and activation mechanisms.

### 2. File Structure

```
.claude/skills/skill-management/
├── SKILL.md                            # Main skill file (YAML + docs)
├── README.md                           # Human-readable documentation
├── bootstrap.sh                        # ONE-TIME bootstrap script
├── scripts/
│   ├── skill-create.js                 # Create new skill from template
│   ├── skill-validate.js               # Validate YAML frontmatter
│   ├── skill-deploy.js                 # Deploy skill to .claude/skills/
│   ├── skill-list.js                   # List installed skills
│   ├── skill-search.js                 # Search skill catalog
│   ├── skill-update.js                 # Update existing skill
│   ├── skill-delete.js                 # Remove skill
│   ├── skill-test.js                   # Test skill structure
│   └── lib/
│       ├── yaml-parser.js              # YAML frontmatter parsing
│       ├── template-engine.js          # Template processing
│       ├── validator.js                # Structure validation
│       └── utils.js                    # Utility functions
├── resources/
│   ├── templates/
│   │   ├── minimal-skill.md            # Minimal skill template
│   │   ├── basic-skill.md              # Basic skill with scripts
│   │   ├── advanced-skill.md           # Full-featured skill
│   │   └── meta-skill.md               # Template for meta-skills
│   ├── schemas/
│   │   ├── yaml-frontmatter-schema.json
│   │   └── skill-structure-schema.json
│   └── examples/
│       ├── simple-skill-example.md
│       └── complex-skill-example.md
└── docs/
    ├── BOOTSTRAP.md                    # Bootstrap instructions
    ├── TEMPLATES.md                    # Template documentation
    ├── VALIDATION.md                   # Validation rules
    └── SELF_HOSTING.md                 # Self-hosting guide

Total Implementation: ~3,200 lines
```

### 3. YAML Frontmatter

```yaml
---
name: "Skill Management"
description: "Create, validate, deploy, and manage Claude Code Skills with template generation, YAML validation, and self-hosting capabilities. Use when building new skills, managing skill catalog, or deploying skills to projects. Eliminates dependencies on claude-flow skill deployment infrastructure."
---
```

### 4. Bootstrap Solution (Chicken-and-Egg Problem)

#### **The Problem:**
```
Q: How do you install the skill-management skill?
A: Use the skill-management skill to install it.

Q: But how do you get the skill-management skill in the first place?
A: Ah, the bootstrap problem!
```

#### **The Solution: Minimal Bootstrap Script**

```bash
#!/bin/bash
# bootstrap.sh - One-time bootstrap script

set -e

echo "🚀 Bootstrapping Skill Management Skill..."

# Detect installation location
if [ -d "$HOME/.claude/skills" ]; then
  SKILLS_DIR="$HOME/.claude/skills"
  SCOPE="global"
else
  SKILLS_DIR=".claude/skills"
  SCOPE="project"
fi

echo "📂 Installing to: $SKILLS_DIR ($SCOPE)"

# Create directory
mkdir -p "$SKILLS_DIR/skill-management"

# Self-extracting archive embedded in this script
# The skill-management skill files are embedded below this line
# They will be extracted to $SKILLS_DIR/skill-management/

# Extract marker
ARCHIVE_MARKER="__SKILL_MANAGEMENT_ARCHIVE_BEGIN__"

# Find the archive start line
ARCHIVE_LINE=$(awk "/$ARCHIVE_MARKER/{print NR + 1; exit 0; }" "$0")

# Extract archive using tail + tar
tail -n +"$ARCHIVE_LINE" "$0" | tar xzf - -C "$SKILLS_DIR"

echo "✅ Skill Management skill installed!"
echo ""
echo "Next steps:"
echo "  1. Restart Claude Code (or refresh Claude.ai)"
echo "  2. Ask: 'Create a new skill for API documentation'"
echo "  3. The skill-management skill will activate automatically"
echo ""
echo "Or use directly:"
echo "  node $SKILLS_DIR/skill-management/scripts/skill-create.js --name 'My Skill'"
echo ""
echo "📚 Documentation: $SKILLS_DIR/skill-management/README.md"

exit 0

# Archive begins here
__SKILL_MANAGEMENT_ARCHIVE_BEGIN__
# <compressed tar archive of skill-management/ directory>
```

**Bootstrap Process:**
1. User downloads `bootstrap.sh`
2. Runs `bash bootstrap.sh`
3. Script extracts embedded skill-management skill
4. Skill installed to `.claude/skills/skill-management/`
5. Claude Code discovers skill on next startup
6. User can now use skill-management to manage all skills (including updating itself)

**Alternative: Git Clone Bootstrap**
```bash
# Even simpler bootstrap
git clone https://github.com/your-org/claude-skills-standalone.git
cd claude-skills-standalone
./install.sh  # Copies skill-management to .claude/skills/
```

---

## Implementation Specifications

### 1. skill-create.js - Create New Skill

```javascript
#!/usr/bin/env node
// scripts/skill-create.js

import fs from 'fs/promises';
import path from 'path';
import { parseArgs } from 'node:util';
import { TemplateEngine } from './lib/template-engine.js';
import { Validator } from './lib/validator.js';

const { values } = parseArgs({
  options: {
    name: { type: 'string' },
    description: { type: 'string' },
    template: { type: 'string', default: 'basic' },
    category: { type: 'string', default: 'general' },
    'output-dir': { type: 'string', default: '.claude/skills' },
    scripts: { type: 'boolean', default: false },
    resources: { type: 'boolean', default: false },
    docs: { type: 'boolean', default: false },
    interactive: { type: 'boolean', default: false }
  }
});

async function createSkill(options) {
  const {
    name,
    description,
    template,
    category,
    outputDir,
    scripts,
    resources,
    docs,
    interactive
  } = options;

  // Validate name
  if (!name || name.length > 64) {
    throw new Error('Skill name required (max 64 chars)');
  }

  // Validate description
  if (!description || description.length > 1024) {
    throw new Error('Description required (max 1024 chars)');
  }

  // Create skill directory name (kebab-case)
  const skillDirName = name
    .toLowerCase()
    .replace(/[^a-z0-9]+/g, '-')
    .replace(/^-+|-+$/g, '');

  const skillDir = path.join(outputDir, skillDirName);

  // Check if skill already exists
  try {
    await fs.access(skillDir);
    throw new Error(`Skill already exists: ${skillDirName}`);
  } catch (error) {
    if (error.code !== 'ENOENT') throw error;
  }

  console.log(`📦 Creating skill: ${name}`);
  console.log(`📂 Directory: ${skillDir}`);

  // Create directory structure
  await fs.mkdir(skillDir, { recursive: true });

  // Load template
  const templateEngine = new TemplateEngine(
    path.join(__dirname, '../resources/templates')
  );

  const templateFile = `${template}-skill.md`;
  const skillContent = await templateEngine.render(templateFile, {
    name,
    description,
    category,
    timestamp: new Date().toISOString(),
    year: new Date().getFullYear()
  });

  // Write SKILL.md
  await fs.writeFile(
    path.join(skillDir, 'SKILL.md'),
    skillContent,
    'utf8'
  );

  console.log('  ✅ Created SKILL.md');

  // Create README.md
  const readmeContent = await templateEngine.render('README.md', {
    name,
    description,
    skillDirName
  });

  await fs.writeFile(
    path.join(skillDir, 'README.md'),
    readmeContent,
    'utf8'
  );

  console.log('  ✅ Created README.md');

  // Optional: Create scripts directory
  if (scripts) {
    const scriptsDir = path.join(skillDir, 'scripts');
    await fs.mkdir(scriptsDir, { recursive: true });
    await fs.mkdir(path.join(scriptsDir, 'lib'), { recursive: true });

    // Create example script
    const exampleScript = await templateEngine.render('example-script.js', {
      skillName: name
    });

    await fs.writeFile(
      path.join(scriptsDir, 'example.js'),
      exampleScript,
      'utf8'
    );

    console.log('  ✅ Created scripts/ directory');
  }

  // Optional: Create resources directory
  if (resources) {
    const resourcesDir = path.join(skillDir, 'resources');
    await fs.mkdir(path.join(resourcesDir, 'templates'), { recursive: true });
    await fs.mkdir(path.join(resourcesDir, 'examples'), { recursive: true });
    await fs.mkdir(path.join(resourcesDir, 'schemas'), { recursive: true });

    console.log('  ✅ Created resources/ directory');
  }

  // Optional: Create docs directory
  if (docs) {
    const docsDir = path.join(skillDir, 'docs');
    await fs.mkdir(docsDir, { recursive: true });

    // Create basic documentation files
    await fs.writeFile(
      path.join(docsDir, 'ADVANCED.md'),
      `# ${name} - Advanced Usage\n\n(Add advanced documentation here)\n`,
      'utf8'
    );

    console.log('  ✅ Created docs/ directory');
  }

  // Validate created skill
  const validator = new Validator();
  const validation = await validator.validateSkill(skillDir);

  if (!validation.valid) {
    console.error('⚠️  Validation warnings:');
    validation.warnings.forEach(warning => {
      console.error(`   - ${warning}`);
    });
  }

  console.log('');
  console.log('✅ Skill created successfully!');
  console.log('');
  console.log('Next steps:');
  console.log(`  1. Edit ${skillDirName}/SKILL.md to add instructions`);
  console.log(`  2. Restart Claude Code to discover the skill`);
  console.log(`  3. Test with: "Use the ${name} skill to..."`);
  console.log('');

  return {
    success: true,
    skillDir,
    skillDirName,
    validation
  };
}

// Run if called directly
if (import.meta.url === `file://${process.argv[1]}`) {
  createSkill(values)
    .then(result => {
      process.exit(0);
    })
    .catch(error => {
      console.error('❌ Error:', error.message);
      process.exit(1);
    });
}

export { createSkill };
```

### 2. skill-validate.js - Validate Skill Structure

```javascript
#!/usr/bin/env node
// scripts/skill-validate.js

import fs from 'fs/promises';
import path from 'path';
import yaml from 'yaml';

export class Validator {
  async validateSkill(skillDir) {
    const warnings = [];
    const errors = [];

    // Check SKILL.md exists
    const skillMdPath = path.join(skillDir, 'SKILL.md');
    try {
      await fs.access(skillMdPath);
    } catch {
      errors.push('SKILL.md not found (REQUIRED)');
      return { valid: false, errors, warnings };
    }

    // Read SKILL.md
    const content = await fs.readFile(skillMdPath, 'utf8');

    // Extract YAML frontmatter
    const yamlMatch = content.match(/^---\n([\s\S]*?)\n---/);
    if (!yamlMatch) {
      errors.push('YAML frontmatter not found (REQUIRED)');
      return { valid: false, errors, warnings };
    }

    const frontmatter = yaml.parse(yamlMatch[1]);

    // Validate required fields
    if (!frontmatter.name) {
      errors.push('name field missing in YAML frontmatter');
    } else if (frontmatter.name.length > 64) {
      errors.push('name exceeds 64 characters');
    }

    if (!frontmatter.description) {
      errors.push('description field missing in YAML frontmatter');
    } else {
      if (frontmatter.description.length > 1024) {
        errors.push('description exceeds 1024 characters');
      }

      // Check for "when" clause in description
      if (!frontmatter.description.match(/\b(use when|when|use for|activate)\b/i)) {
        warnings.push('Description should include trigger conditions (when to use)');
      }
    }

    // Check for extra fields (informational only)
    const allowedFields = ['name', 'description'];
    const extraFields = Object.keys(frontmatter).filter(
      key => !allowedFields.includes(key)
    );

    if (extraFields.length > 0) {
      warnings.push(`Extra YAML fields will be ignored: ${extraFields.join(', ')}`);
    }

    // Check directory structure
    const entries = await fs.readdir(skillDir, { withFileTypes: true });

    // Recommended files/directories
    const hasReadme = entries.some(e => e.name === 'README.md');
    const hasScripts = entries.some(e => e.isDirectory() && e.name === 'scripts');
    const hasResources = entries.some(e => e.isDirectory() && e.name === 'resources');
    const hasDocs = entries.some(e => e.isDirectory() && e.name === 'docs');

    if (!hasReadme) {
      warnings.push('README.md not found (recommended for documentation)');
    }

    // Check SKILL.md content structure
    const hasLevel1Header = content.match(/^# /m);
    if (!hasLevel1Header) {
      warnings.push('SKILL.md should have a Level 1 header (# Title)');
    }

    const hasQuickStart = content.match(/## Quick Start/i);
    if (!hasQuickStart) {
      warnings.push('SKILL.md should have a "Quick Start" section');
    }

    return {
      valid: errors.length === 0,
      errors,
      warnings,
      frontmatter,
      structure: {
        hasReadme,
        hasScripts,
        hasResources,
        hasDocs
      }
    };
  }

  async validateAllSkills(skillsDir) {
    const entries = await fs.readdir(skillsDir, { withFileTypes: true });
    const skillDirs = entries.filter(e => e.isDirectory());

    const results = [];

    for (const skillDir of skillDirs) {
      const skillPath = path.join(skillsDir, skillDir.name);
      const validation = await this.validateSkill(skillPath);

      results.push({
        name: skillDir.name,
        ...validation
      });
    }

    return results;
  }
}

// CLI usage
if (import.meta.url === `file://${process.argv[1]}`) {
  const skillPath = process.argv[2] || '.claude/skills';

  const validator = new Validator();

  (async () => {
    try {
      const stats = await fs.stat(skillPath);

      if (stats.isDirectory()) {
        // Check if this is a single skill or skills directory
        const hasSkillMd = await fs.access(path.join(skillPath, 'SKILL.md'))
          .then(() => true)
          .catch(() => false);

        if (hasSkillMd) {
          // Validate single skill
          const result = await validator.validateSkill(skillPath);
          console.log(JSON.stringify(result, null, 2));
        } else {
          // Validate all skills in directory
          const results = await validator.validateAllSkills(skillPath);
          console.log(JSON.stringify(results, null, 2));
        }
      }
    } catch (error) {
      console.error('Error:', error.message);
      process.exit(1);
    }
  })();
}
```

### 3. Template Engine (lib/template-engine.js)

```javascript
// scripts/lib/template-engine.js

import fs from 'fs/promises';
import path from 'path';

export class TemplateEngine {
  constructor(templatesDir) {
    this.templatesDir = templatesDir;
  }

  /**
   * Render a template with variables
   */
  async render(templateName, variables = {}) {
    const templatePath = path.join(this.templatesDir, templateName);
    let template = await fs.readFile(templatePath, 'utf8');

    // Simple variable substitution: {{variableName}}
    for (const [key, value] of Object.entries(variables)) {
      const regex = new RegExp(`\\{\\{${key}\\}\\}`, 'g');
      template = template.replace(regex, value);
    }

    return template;
  }

  /**
   * List available templates
   */
  async listTemplates() {
    const files = await fs.readdir(this.templatesDir);
    return files.filter(f => f.endsWith('.md'));
  }
}
```

### 4. Basic Skill Template

```markdown
---
name: "{{name}}"
description: "{{description}}"
---

# {{name}}

## What This Skill Does

[Describe what this skill does in 2-3 sentences]

## Prerequisites

- Prerequisite 1
- Prerequisite 2

## Quick Start

### Basic Usage

\`\`\`bash
# Example command
command --option value
\`\`\`

### Common Scenarios

1. **Scenario 1**: How to...
2. **Scenario 2**: How to...

---

## Detailed Instructions

### Step 1: Setup

[Setup instructions]

### Step 2: Configuration

[Configuration instructions]

### Step 3: Execution

[Execution instructions]

---

## Troubleshooting

### Issue: Common Problem

**Solution**: Steps to resolve...

---

## Learn More

- Documentation: [Link]
- Examples: resources/examples/
- Advanced: docs/ADVANCED.md

---

*Generated on {{timestamp}} by Skill Management*
```

### 5. skill-deploy.js - Deploy Skill

```javascript
#!/usr/bin/env node
// scripts/skill-deploy.js

import fs from 'fs/promises';
import path from 'path';
import { parseArgs } from 'node:util';
import { Validator } from './lib/validator.js';

const { values } = parseArgs({
  options: {
    source: { type: 'string' },
    target: { type: 'string', default: '.claude/skills' },
    force: { type: 'boolean', default: false },
    'dry-run': { type: 'boolean', default: false },
    validate: { type: 'boolean', default: true }
  }
});

async function deploySkill(options) {
  const { source, target, force, dryRun, validate } = options;

  if (!source) {
    throw new Error('Source skill directory required (--source)');
  }

  // Validate source skill
  if (validate) {
    console.log('🔍 Validating skill...');
    const validator = new Validator();
    const validation = await validator.validateSkill(source);

    if (!validation.valid) {
      console.error('❌ Validation failed:');
      validation.errors.forEach(error => {
        console.error(`   - ${error}`);
      });
      throw new Error('Skill validation failed');
    }

    if (validation.warnings.length > 0) {
      console.log('⚠️  Warnings:');
      validation.warnings.forEach(warning => {
        console.log(`   - ${warning}`);
      });
    }
  }

  // Get skill name from directory
  const skillName = path.basename(source);
  const targetPath = path.join(target, skillName);

  console.log(`📦 Deploying skill: ${skillName}`);
  console.log(`📂 Source: ${source}`);
  console.log(`📂 Target: ${targetPath}`);

  // Check if skill already exists
  try {
    await fs.access(targetPath);
    if (!force) {
      throw new Error(`Skill already exists at ${targetPath}. Use --force to overwrite.`);
    }
    console.log('⚠️  Overwriting existing skill (--force)');
  } catch (error) {
    if (error.code !== 'ENOENT' && !error.message.includes('--force')) {
      throw error;
    }
  }

  if (dryRun) {
    console.log('[DRY RUN] Would deploy skill to:', targetPath);
    return { success: true, dryRun: true, targetPath };
  }

  // Copy skill directory recursively
  await copyRecursive(source, targetPath);

  console.log('✅ Skill deployed successfully!');
  console.log('');
  console.log('Next steps:');
  console.log('  1. Restart Claude Code to discover the skill');
  console.log(`  2. Test with: "Use the ${skillName} skill to..."`);
  console.log('');

  return { success: true, targetPath };
}

async function copyRecursive(src, dest) {
  const stats = await fs.stat(src);

  if (stats.isDirectory()) {
    await fs.mkdir(dest, { recursive: true });
    const entries = await fs.readdir(src, { withFileTypes: true });

    for (const entry of entries) {
      const srcPath = path.join(src, entry.name);
      const destPath = path.join(dest, entry.name);

      if (entry.isDirectory()) {
        await copyRecursive(srcPath, destPath);
      } else {
        await fs.copyFile(srcPath, destPath);
      }
    }
  } else {
    await fs.copyFile(src, dest);
  }
}

// CLI usage
if (import.meta.url === `file://${process.argv[1]}`) {
  deploySkill(values)
    .then(() => process.exit(0))
    .catch(error => {
      console.error('❌ Error:', error.message);
      process.exit(1);
    });
}

export { deploySkill };
```

### 6. skill-list.js - List Skills

```javascript
#!/usr/bin/env node
// scripts/skill-list.js

import fs from 'fs/promises';
import path from 'path';
import yaml from 'yaml';
import { parseArgs } from 'node:util';

const { values } = parseArgs({
  options: {
    'skills-dir': { type: 'string', default: '.claude/skills' },
    format: { type: 'string', default: 'table' }, // 'table', 'json', 'yaml'
    'show-description': { type: 'boolean', default: false },
    category: { type: 'string' },
    search: { type: 'string' }
  }
});

async function listSkills(options) {
  const { skillsDir, format, showDescription, category, search } = options;

  const entries = await fs.readdir(skillsDir, { withFileTypes: true });
  const skillDirs = entries.filter(e => e.isDirectory());

  const skills = [];

  for (const skillDir of skillDirs) {
    const skillPath = path.join(skillsDir, skillDir.name);
    const skillMdPath = path.join(skillPath, 'SKILL.md');

    try {
      const content = await fs.readFile(skillMdPath, 'utf8');
      const yamlMatch = content.match(/^---\n([\s\S]*?)\n---/);

      if (yamlMatch) {
        const frontmatter = yaml.parse(yamlMatch[1]);

        // Apply filters
        if (category && frontmatter.category !== category) {
          continue;
        }

        if (search) {
          const searchLower = search.toLowerCase();
          const nameMatch = frontmatter.name.toLowerCase().includes(searchLower);
          const descMatch = frontmatter.description.toLowerCase().includes(searchLower);

          if (!nameMatch && !descMatch) {
            continue;
          }
        }

        skills.push({
          directory: skillDir.name,
          name: frontmatter.name,
          description: frontmatter.description,
          category: frontmatter.category || 'uncategorized',
          path: skillPath
        });
      }
    } catch (error) {
      // Skip invalid skills
    }
  }

  // Output
  if (format === 'json') {
    console.log(JSON.stringify(skills, null, 2));
  } else if (format === 'yaml') {
    console.log(yaml.stringify(skills));
  } else {
    // Table format
    console.log(`Found ${skills.length} skills:\n`);

    for (const skill of skills) {
      console.log(`📦 ${skill.name}`);
      console.log(`   Directory: ${skill.directory}`);
      if (showDescription) {
        const desc = skill.description.substring(0, 80);
        console.log(`   ${desc}${skill.description.length > 80 ? '...' : ''}`);
      }
      console.log('');
    }
  }

  return skills;
}

// CLI usage
if (import.meta.url === `file://${process.argv[1]}`) {
  listSkills(values)
    .then(() => process.exit(0))
    .catch(error => {
      console.error('❌ Error:', error.message);
      process.exit(1);
    });
}

export { listSkills };
```

---

## Complete SKILL.md for Meta-Skill

```markdown
---
name: "Skill Management"
description: "Create, validate, deploy, and manage Claude Code Skills with template generation, YAML validation, and self-hosting capabilities. Use when building new skills, managing skill catalog, deploying skills to projects, or updating existing skills. Eliminates dependencies on claude-flow skill deployment infrastructure."
---

# Skill Management

## What This Skill Does

Provides a complete toolkit for creating, validating, deploying, and managing Claude Code Skills. This meta-skill can create other skills (including updating itself) and handles all aspects of skill lifecycle management without requiring claude-flow dependencies.

**Key Capabilities:**
1. Create new skills from templates
2. Validate YAML frontmatter and structure
3. Deploy skills to .claude/skills/
4. List and search installed skills
5. Update existing skills
6. Self-hosting (can manage itself)

## Prerequisites

- Node.js 18+
- Claude Code 2.0+ or Claude.ai with Skills support

## Quick Start

### Bootstrap Installation (First Time Only)

```bash
# Download bootstrap script
curl -o bootstrap.sh https://raw.githubusercontent.com/your-org/claude-skills/main/bootstrap.sh

# Run bootstrap
bash bootstrap.sh

# Bootstrap extracts skill-management to .claude/skills/
# Restart Claude Code to discover the skill
```

### Create Your First Skill

```bash
# Interactive mode (Claude activates skill automatically)
"Create a new skill for API documentation"

# Or use CLI directly
node .claude/skills/skill-management/scripts/skill-create.js \
  --name "API Documentation Generator" \
  --description "Generate OpenAPI 3.0 docs from Express routes. Use when documenting APIs." \
  --template advanced \
  --scripts \
  --resources
```

### Validate Skills

```bash
# Validate single skill
node .claude/skills/skill-management/scripts/skill-validate.js \
  .claude/skills/my-skill

# Validate all skills
node .claude/skills/skill-management/scripts/skill-validate.js \
  .claude/skills
```

### Deploy Skills

```bash
# Deploy to project
node .claude/skills/skill-management/scripts/skill-deploy.js \
  --source ~/my-skills/api-docs \
  --target .claude/skills

# Deploy to global skills
node .claude/skills/skill-management/scripts/skill-deploy.js \
  --source ~/my-skills/api-docs \
  --target ~/.claude/skills
```

### List Skills

```bash
# List all skills
node .claude/skills/skill-management/scripts/skill-list.js

# Search skills
node .claude/skills/skill-management/scripts/skill-list.js \
  --search "api"

# Filter by category
node .claude/skills/skill-management/scripts/skill-list.js \
  --category "development"

# JSON output
node .claude/skills/skill-management/scripts/skill-list.js \
  --format json
```

---

## Detailed Instructions

### Creating Skills

#### Using Templates

Available templates:
- `minimal` - Just SKILL.md and README
- `basic` - Includes scripts/ directory
- `advanced` - Full structure with scripts/, resources/, docs/
- `meta` - Template for meta-skills (skills that create skills)

```bash
# Create with specific template
node .claude/skills/skill-management/scripts/skill-create.js \
  --name "Database Schema Designer" \
  --description "Design PostgreSQL schemas with constraints. Use when creating databases." \
  --template advanced \
  --scripts \
  --resources \
  --docs \
  --category "database"
```

#### Interactive Mode

```bash
# Let Claude guide you through skill creation
"I want to create a skill for generating React components with TypeScript and tests"

# Claude will:
# 1. Activate skill-management skill
# 2. Ask clarifying questions
# 3. Generate skill with appropriate template
# 4. Validate structure
# 5. Deploy to .claude/skills/
```

### Validation Rules

#### YAML Frontmatter

```yaml
---
name: "Skill Name"              # REQUIRED: Max 64 chars
description: "What it does      # REQUIRED: Max 1024 chars, must include "when"
and when to use it."
---
```

**Validation checks:**
- ✅ YAML frontmatter exists
- ✅ `name` field present and ≤64 chars
- ✅ `description` field present and ≤1024 chars
- ✅ Description includes trigger words ("when", "use for", etc.)
- ⚠️ Warns about extra fields (ignored by Claude)

#### Structure Validation

```
skill-name/
├── SKILL.md           # REQUIRED
├── README.md          # RECOMMENDED
├── scripts/           # Optional but recommended
├── resources/         # Optional
└── docs/              # Optional
```

**Validation checks:**
- ✅ SKILL.md exists
- ✅ Has Level 1 header (#)
- ⚠️ Recommends README.md
- ⚠️ Recommends "Quick Start" section

### Deployment Workflows

#### Local Development

```bash
# 1. Create skill in workspace
cd ~/workspace/my-skill

# 2. Validate
node .claude/skills/skill-management/scripts/skill-validate.js .

# 3. Test locally
cp -r . .claude/skills/my-skill
# Restart Claude Code, test skill

# 4. Deploy when ready
node .claude/skills/skill-management/scripts/skill-deploy.js \
  --source . \
  --target ~/.claude/skills \
  --validate
```

#### Team Distribution

```bash
# Create skill repository
git init my-team-skills
cd my-team-skills

# Create skills
node .claude/skills/skill-management/scripts/skill-create.js \
  --name "Team Coding Standards" \
  --description "Enforce team coding standards and patterns." \
  --output-dir ./skills

# Commit and push
git add .
git commit -m "Add team coding standards skill"
git push

# Team members deploy
git clone <repo>
node .claude/skills/skill-management/scripts/skill-deploy.js \
  --source ./skills/team-coding-standards \
  --target .claude/skills
```

---

## Advanced Features

### Self-Hosting: Updating the Skill-Management Skill

The skill-management skill can update itself:

```bash
# 1. Create updated version
cd ~/workspace
git clone https://github.com/your-org/skill-management.git
cd skill-management

# Make changes to skill-management/
# ...

# 2. Validate updated skill
node ./scripts/skill-validate.js .

# 3. Deploy update (replaces current version)
node ./scripts/skill-deploy.js \
  --source . \
  --target ~/.claude/skills \
  --force

# 4. Restart Claude Code
# Updated skill-management skill is now active
```

### Creating Meta-Skills

Meta-skills are skills that create or manage other skills:

```bash
# Use meta template
node .claude/skills/skill-management/scripts/skill-create.js \
  --name "Code Generator Skill Builder" \
  --description "Create skills that generate code from templates." \
  --template meta \
  --scripts \
  --resources

# This creates a skill that generates code-generation skills!
```

### Skill Templates Best Practices

#### Progressive Disclosure Structure

```markdown
# Skill Name

## Level 1: Overview (Always Read)
2-3 sentence summary

## Prerequisites
- List requirements

## Quick Start (Level 2: Fast Onboarding)
### Basic Usage
[Simple examples]

---

## Detailed Instructions (Level 3: Deep Work)
### Step-by-Step Guide
[Comprehensive instructions]

---

## Advanced Features (Level 4: Power Users)
### Complex Scenarios
[Advanced use cases]

## Learn More
- See: docs/ADVANCED.md
- Examples: resources/examples/
```

#### YAML Description Tips

```yaml
# ✅ GOOD: Includes what + when + specific triggers
description: "Generate OpenAPI 3.0 documentation from Express.js routes. Use when creating API docs, documenting endpoints, or building API specifications for teams."

# ✅ GOOD: Front-loads key trigger words
description: "Create React functional components with TypeScript, hooks, and tests. Use when scaffolding new components, converting class components, or building UI libraries."

# ❌ BAD: No trigger conditions
description: "A comprehensive guide to API documentation with examples and best practices."

# ❌ BAD: Too vague
description: "Documentation tool"
```

---

## Troubleshooting

### Issue: Skill not discovered by Claude Code

**Solutions:**
1. Verify SKILL.md has valid YAML frontmatter
2. Restart Claude Code completely
3. Check skill location: `~/.claude/skills/` or `.claude/skills/`
4. Run validation: `node .claude/skills/skill-management/scripts/skill-validate.js <skill-dir>`

### Issue: YAML parsing error

**Solutions:**
1. Ensure `---` delimiters are on their own lines
2. Quote strings with special characters
3. Check indentation (use spaces, not tabs)
4. Use validation script to identify issues

### Issue: Skill creates skill but Claude doesn't activate it

**Solutions:**
1. Check description includes trigger words ("when", "use for")
2. Make description more specific and detailed
3. Add relevant keywords to description
4. Restart Claude Code after creating skill

### Issue: Bootstrap script fails

**Solutions:**
1. Check bash version: `bash --version` (need 3.2+)
2. Verify write permissions: `ls -la ~/.claude/`
3. Download script directly: `curl -O <url>`
4. Extract manually if needed: See docs/BOOTSTRAP.md

---

## Migration from claude-flow

### Replacing claude-flow init

**Before (claude-flow):**
```bash
npx claude-flow@alpha init --force
# Copies skills from claude-flow package
```

**After (standalone):**
```bash
bash bootstrap.sh
# Installs skill-management skill only
# Use skill-management to create/deploy other skills
```

### Replacing skill-copier.js

**Before:** skill-copier.js copies all 25 skills from package

**After:** skill-create.js generates skills on-demand from templates

**Advantages:**
- Create only the skills you need
- Customize skills during creation
- No dependency on claude-flow package structure
- Easy to version control custom skills

### Migration Strategy

**Phase 1: Install skill-management**
```bash
# Bootstrap skill-management skill
bash bootstrap.sh
```

**Phase 2: Recreate needed skills**
```bash
# List claude-flow skills currently in use
ls .claude/skills/

# For each skill, either:
# A. Keep claude-flow version (no action needed)
# B. Recreate with skill-management:

node .claude/skills/skill-management/scripts/skill-create.js \
  --name "Pair Programming" \
  --description "..." \
  --template advanced
```

**Phase 3: Remove claude-flow dependency**
```bash
# Optional: Remove claude-flow if only used for skills
npm uninstall claude-flow

# skill-management skill is now standalone
```

---

## Learn More

- **Bootstrap Guide**: docs/BOOTSTRAP.md
- **Template Documentation**: docs/TEMPLATES.md
- **Validation Rules**: docs/VALIDATION.md
- **Self-Hosting Guide**: docs/SELF_HOSTING.md
- **Claude Skills Spec**: https://docs.claude.com/claude-code/skills

---

## API Reference

### skill-create.js

```
Usage: node skill-create.js [options]

Options:
  --name <name>               Skill name (required, max 64 chars)
  --description <desc>        Skill description (required, max 1024 chars)
  --template <type>           Template type: minimal|basic|advanced|meta
  --category <category>       Skill category
  --output-dir <dir>          Output directory (default: .claude/skills)
  --scripts                   Create scripts/ directory
  --resources                 Create resources/ directory
  --docs                      Create docs/ directory
  --interactive               Interactive mode with prompts
```

### skill-validate.js

```
Usage: node skill-validate.js [skill-dir]

Arguments:
  skill-dir                   Directory to validate (default: .claude/skills)

Output:
  JSON object with validation results
```

### skill-deploy.js

```
Usage: node skill-deploy.js [options]

Options:
  --source <dir>              Source skill directory (required)
  --target <dir>              Target skills directory
  --force                     Overwrite existing skill
  --dry-run                   Preview changes without deploying
  --validate                  Validate before deploying (default: true)
```

### skill-list.js

```
Usage: node skill-list.js [options]

Options:
  --skills-dir <dir>          Skills directory (default: .claude/skills)
  --format <type>             Output format: table|json|yaml
  --show-description          Show full descriptions
  --category <cat>            Filter by category
  --search <query>            Search skill names and descriptions
```

---

*This skill is self-hosting - it can manage and update itself!*
```

---

## Migration Strategy

### Phase 1: Bootstrap (One-Time Setup)

**Step 1: Download Bootstrap Script**
```bash
curl -o bootstrap.sh https://raw.githubusercontent.com/your-org/claude-skills/main/bootstrap.sh
```

**Step 2: Run Bootstrap**
```bash
bash bootstrap.sh
# Extracts skill-management to .claude/skills/skill-management/
```

**Step 3: Verify Installation**
```bash
ls .claude/skills/skill-management/
# Should see: SKILL.md, README.md, scripts/, resources/, docs/
```

**Step 4: Restart Claude Code**
```bash
# Restart Claude Code to discover skill
# Or refresh Claude.ai
```

### Phase 2: Skill Migration (Optional)

If you have existing claude-flow skills you want to recreate:

**Step 1: Inventory Current Skills**
```bash
ls .claude/skills/
# List: skill-builder, sparc-methodology, pair-programming, etc.
```

**Step 2: Recreate Critical Skills**
```bash
# For each skill you actively use, recreate with skill-management:

node .claude/skills/skill-management/scripts/skill-create.js \
  --name "Pair Programming" \
  --description "Driver/navigator pair programming with real-time verification. Use when coding collaboratively." \
  --template advanced \
  --scripts \
  --resources \
  --docs

# Copy content from old skill
cp .claude/skills/pair-programming/SKILL.md \
   .claude/skills/pair-programming-new/SKILL.md

# Validate
node .claude/skills/skill-management/scripts/skill-validate.js \
  .claude/skills/pair-programming-new

# Deploy
mv .claude/skills/pair-programming-new .claude/skills/pair-programming
```

**Step 3: Remove claude-flow (Optional)**
```bash
# If claude-flow was only used for skill deployment:
npm uninstall claude-flow

# skill-management is now standalone
```

### Phase 3: Team Distribution

**Option A: Git Repository**
```bash
# 1. Create skill repository
mkdir my-team-skills
cd my-team-skills

# 2. Add skill-management as submodule
git submodule add https://github.com/your-org/skill-management.git .claude/skills/skill-management

# 3. Team members clone
git clone --recursive https://github.com/your-org/my-team-skills.git
```

**Option B: Bootstrap Distribution**
```bash
# 1. Host bootstrap script
# Upload bootstrap.sh to: https://your-company.com/claude-skills/bootstrap.sh

# 2. Team members run
curl -o bootstrap.sh https://your-company.com/claude-skills/bootstrap.sh
bash bootstrap.sh
```

**Option C: NPM Package**
```bash
# 1. Publish skill-management as npm package
npm publish @your-org/claude-skill-management

# 2. Team members install
npx @your-org/claude-skill-management bootstrap

# 3. Creates skill-management in .claude/skills/
```

---

## Comparison Matrix

| Feature | claude-flow Skills | Standalone Skill-Management |
|---------|-------------------|----------------------------|
| **Installation** | `npx claude-flow init` | `bash bootstrap.sh` |
| **Dependencies** | claude-flow package | None (standalone) |
| **Skill Deployment** | Copy from package | Generate from templates |
| **Number of Skills** | 25 pre-built | Create on-demand |
| **Customization** | Edit after copying | Customize during creation |
| **Version Control** | Commit all 25 skills | Commit only what you need |
| **Updates** | npm update | Self-hosting (update itself) |
| **Team Distribution** | NPM dependency | Git/bootstrap script |
| **Offline Support** | After initial install | Fully offline |
| **File Size** | ~26,500 lines (all skills) | ~3,200 lines (meta-skill only) |
| **Learning Curve** | Low (pre-built) | Medium (template-based) |
| **Flexibility** | Fixed 25 skills | Unlimited custom skills |

### Recommendation:

- **Use claude-flow** if you want all 25 pre-built skills instantly with zero configuration
- **Use skill-management** if you want:
  - Zero dependencies
  - Create custom skills
  - Minimal deployment (only what you need)
  - Self-hosting capabilities
  - Team-specific skill templates

---

## FAQ

### Q: How does the bootstrap script work?

A: The bootstrap script is a self-extracting archive. It contains:
1. Installation logic (bash script at the top)
2. Compressed tar archive of skill-management/ (embedded at bottom)
3. Extraction marker separating the two

When run, it:
1. Detects target location (~/.claude/skills/ or .claude/skills/)
2. Finds the embedded archive using the marker
3. Extracts the archive using `tail + tar`
4. skill-management/ is now installed

### Q: Can I use both claude-flow skills and skill-management?

A: Yes! They can coexist:
- claude-flow provides 25 pre-built skills
- skill-management creates custom skills
- Both deploy to .claude/skills/
- Claude Code discovers all skills equally

### Q: How do I update the skill-management skill itself?

A: Use self-hosting:
```bash
# 1. Get updated version
git clone https://github.com/your-org/skill-management.git
cd skill-management

# 2. Deploy update (overwrites current version)
node ./scripts/skill-deploy.js \
  --source . \
  --target ~/.claude/skills \
  --force

# 3. Restart Claude Code
```

The skill-management skill updates itself!

### Q: What if I want to share skills with my team?

A: Three options:

**Option 1: Git Repository**
```bash
# Create skill repo
git init my-team-skills
cp -r .claude/skills/* my-team-skills/
git add . && git commit -m "Team skills"
git push

# Team members clone and deploy
git clone <repo>
node .claude/skills/skill-management/scripts/skill-deploy.js \
  --source my-team-skills/custom-skill \
  --target .claude/skills
```

**Option 2: Skill Packages**
Create NPM packages for individual skills:
```bash
# Publish skill
cd .claude/skills/my-skill
npm init
npm publish @myteam/claude-skill-my-skill

# Team installs
npx @myteam/claude-skill-my-skill
```

**Option 3: Shared Bootstrap**
Host a custom bootstrap with your team's skills pre-installed.

### Q: Performance impact of many skills?

A: Minimal! Thanks to progressive disclosure:
- **100 skills** = ~6KB context (just YAML frontmatter)
- Only active skill body (~1-10KB) loaded when triggered
- Referenced files loaded on-demand

### Q: Can I convert claude-flow skills to standalone?

A: Yes, two approaches:

**Approach 1: Copy and Deploy**
```bash
# Copy existing skill
cp -r .claude/skills/sparc-methodology ~/my-skills/

# Deploy with skill-management
node .claude/skills/skill-management/scripts/skill-deploy.js \
  --source ~/my-skills/sparc-methodology \
  --target ~/.claude/skills
```

**Approach 2: Recreate from Template**
```bash
# Create new skill with same content
node .claude/skills/skill-management/scripts/skill-create.js \
  --name "SPARC Methodology" \
  --description "..." \
  --template advanced

# Copy content from original
cp .claude/skills/sparc-methodology/SKILL.md \
   .claude/skills/sparc-methodology-new/SKILL.md
```

### Q: What's the minimum viable skill?

A: Just SKILL.md with YAML frontmatter:
```markdown
---
name: "Hello World"
description: "Greets the world. Use when testing skills."
---

# Hello World

Say hello to the world!
```

That's it! Claude Code will discover and activate it.

---

## Next Steps

1. **Review this architecture document**
2. **Approve implementation approach**
3. **Create bootstrap script**
   - Self-extracting archive
   - Installation logic
   - Verification
4. **Implement core scripts** (~3,200 lines):
   - skill-create.js
   - skill-validate.js
   - skill-deploy.js
   - skill-list.js
   - Template engine
   - Validator
5. **Create templates**:
   - minimal-skill.md
   - basic-skill.md
   - advanced-skill.md
   - meta-skill.md
6. **Write documentation**:
   - BOOTSTRAP.md
   - TEMPLATES.md
   - VALIDATION.md
   - SELF_HOSTING.md
7. **Test self-hosting**:
   - Bootstrap installation
   - Skill creation
   - Validation
   - Deployment
   - Self-update
8. **Distribution**:
   - GitHub repository
   - Bootstrap script hosting
   - Documentation site

---

## References

- **Claude Code Skills Documentation**: https://docs.claude.com/claude-code/skills
- **Claude Skills Research**: docs/CLAUDE_SKILLS_RESEARCH.md (1,693 lines)
- **Commands to Skills Migration**: docs/COMMANDS_TO_SKILLS_MIGRATION.md
- **YAML Specification**: https://yaml.org/spec/
- **Progressive Disclosure Pattern**: Anthropic Skills API docs
