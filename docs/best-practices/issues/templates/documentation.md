---
name: 📚 Documentation
about: Documentation updates, additions, or corrections
title: '[DOCS] Clear, Descriptive Title'
labels: ['documentation']
assignees: ''
---

## 📚 Documentation Overview

<!-- Clear description of documentation need -->

[Describe what documentation needs to be updated, added, or corrected]

---

## 🎯 Documentation Type

- [ ] **API Documentation** - Interface, methods, parameters
- [ ] **User Guide** - How-to guides and tutorials
- [ ] **Developer Guide** - Architecture, contributing, development
- [ ] **Configuration Guide** - Setup and configuration
- [ ] **Troubleshooting Guide** - Common issues and solutions
- [ ] **Migration Guide** - Version upgrade instructions
- [ ] **Release Notes** - Changelog and release information
- [ ] **Examples** - Code examples and samples
- [ ] **README** - Project overview and quick start
- [ ] **Other**: [Specify type]

---

## 📋 Current State

### What Exists
[Describe current documentation, if any]

### Issues with Current Documentation
- [ ] **Outdated**: Information is no longer accurate
- [ ] **Incomplete**: Missing important details
- [ ] **Incorrect**: Contains errors or inaccuracies
- [ ] **Unclear**: Confusing or poorly explained
- [ ] **Missing**: Documentation doesn't exist
- [ ] **Poorly Organized**: Hard to find information
- [ ] **Lacks Examples**: Needs more code samples
- [ ] **Other**: [Specify issue]

---

## ✨ Proposed Documentation

### Objectives
- [ ] Objective 1: [Specific documentation goal]
- [ ] Objective 2: [Specific documentation goal]
- [ ] Objective 3: [Specific documentation goal]

### Scope

#### What Will Be Documented
- Item 1: [Description]
- Item 2: [Description]
- Item 3: [Description]

#### What Will NOT Be Documented
- Item 1: [Explanation of why]
- Item 2: [Explanation of why]

---

## 📝 Documentation Structure

### Proposed Outline

```markdown
# Main Title

## Section 1: Overview
- Subsection 1.1
- Subsection 1.2

## Section 2: Getting Started
- Subsection 2.1: Installation
- Subsection 2.2: Quick Start
- Subsection 2.3: Basic Configuration

## Section 3: Core Concepts
- Subsection 3.1
- Subsection 3.2

## Section 4: API Reference
- Subsection 4.1: Methods
- Subsection 4.2: Parameters
- Subsection 4.3: Return Values

## Section 5: Advanced Topics
- Subsection 5.1
- Subsection 5.2

## Section 6: Examples
- Example 1
- Example 2

## Section 7: Troubleshooting
- Common Issues
- FAQ

## Section 8: Additional Resources
```

---

## 💡 Content Requirements

### Information to Include

#### For API Documentation
- [ ] Function/method signatures
- [ ] Parameter descriptions and types
- [ ] Return value descriptions and types
- [ ] Error codes and exceptions
- [ ] Code examples for each method
- [ ] Best practices
- [ ] Common pitfalls

#### For User Guides
- [ ] Step-by-step instructions
- [ ] Screenshots/diagrams (if applicable)
- [ ] Prerequisites
- [ ] Expected outcomes
- [ ] Common issues and solutions
- [ ] Next steps
- [ ] Related guides

#### For Developer Guides
- [ ] Architecture overview
- [ ] Development setup
- [ ] Build instructions
- [ ] Test instructions
- [ ] Contribution guidelines
- [ ] Code style guide
- [ ] Release process

#### For Configuration Guides
- [ ] All configuration options
- [ ] Default values
- [ ] Valid value ranges
- [ ] Examples for common scenarios
- [ ] Environment variables
- [ ] Configuration file formats

---

## 📊 Content Strategy

### Target Audience
- [ ] End Users
- [ ] Developers
- [ ] Contributors
- [ ] System Administrators
- [ ] Other: [Specify]

### Skill Level
- [ ] Beginner
- [ ] Intermediate
- [ ] Advanced
- [ ] Mixed (progressive disclosure)

### Tone & Style
- [ ] Formal/Technical
- [ ] Conversational/Accessible
- [ ] Tutorial-based
- [ ] Reference-style

---

## 🎨 Formatting Guidelines

### Structure
- Use clear, hierarchical headings (H1, H2, H3)
- Include table of contents for long documents
- Use progressive disclosure (collapsible sections)
- Provide navigation links

### Code Examples
```markdown
\`\`\`javascript
// Always include:
// 1. Language hint for syntax highlighting
// 2. Inline comments explaining key parts
// 3. Complete, runnable examples

const example = "well-documented code";
// Explanation of what this does
\`\`\`

\`\`\`bash
# Command-line examples with expected output
$ npm install package
# Output:
# added 1 package, and audited 2 packages in 1s
\`\`\`
```

### Visual Elements
- [ ] **Diagrams**: Architecture, flow charts, etc.
- [ ] **Screenshots**: UI elements, configurations
- [ ] **Tables**: Comparison tables, parameter lists
- [ ] **Callouts**: Notes, warnings, tips

```markdown
> **Note**: Important information
> **Warning**: Critical warnings
> **Tip**: Helpful suggestions
```

### Links & References
- Link to related documentation
- Link to source code when relevant
- Include external resources
- Provide "See Also" sections

---

## ✅ Examples to Include

### Example 1: [Title]
**Scenario**: [When to use this]

```javascript
// Complete, runnable code example
const example1 = () => {
  // Implementation
};
```

**Expected Output**:
```
Output here
```

### Example 2: [Title]
**Scenario**: [When to use this]

```javascript
// Complete, runnable code example
const example2 = () => {
  // Implementation
};
```

**Expected Output**:
```
Output here
```

### Example 3: [Title]
**Scenario**: [When to use this]

```javascript
// Complete, runnable code example
const example3 = () => {
  // Implementation
};
```

---

## 🔍 Accuracy & Review

### Technical Accuracy
- [ ] All code examples tested and verified
- [ ] All commands tested
- [ ] All configuration tested
- [ ] Version compatibility verified
- [ ] Links verified (not broken)

### Review Process
- [ ] Self-review for clarity and completeness
- [ ] Technical review by subject matter expert
- [ ] User review for comprehension
- [ ] Copy editing for grammar and style

### Version Information
- **Applies to Version**: vX.Y.Z
- **Last Verified**: YYYY-MM-DD
- **Next Review Date**: YYYY-MM-DD

---

## 🐝 Swarm Coordination

**Recommended**: ✅ Yes (for large documentation projects) | ❌ No

### Swarm Configuration
- **Topology**: star (coordinator with specialist writers)
- **Max Agents**: 3-5

### Agent Assignment
- **Coordinator**: Overall structure and consistency
- **Technical Writer**: API and developer docs
- **Tutorial Writer**: User guides and examples
- **Editor**: Copy editing and review

### Task Breakdown
- [ ] **Coordinator**: Create outline and structure
- [ ] **Technical Writer**: Write API documentation
- [ ] **Tutorial Writer**: Write user guides and examples
- [ ] **Editor**: Review for clarity and consistency
- [ ] **All**: Test all examples and commands

---

## 🔗 Related Documentation

<!-- Link to related documentation -->

- Related docs: [Link to related documentation]
- Supersedes: [Link to old documentation being replaced]
- Part of: [Link to parent documentation]

---

## 📂 File Locations

### Files to Create/Update

```
docs/
├── api/
│   └── [new-api-doc].md
├── guides/
│   ├── user/
│   │   └── [new-user-guide].md
│   └── developer/
│       └── [new-dev-guide].md
├── examples/
│   └── [new-example].md
└── README.md (update)
```

---

## ✅ Completion Checklist

### Writing
- [ ] Outline created and approved
- [ ] All sections written
- [ ] All code examples written and tested
- [ ] All images/diagrams created
- [ ] All links added and verified

### Review
- [ ] Technical accuracy verified
- [ ] Code examples tested
- [ ] Grammar and style checked
- [ ] Reviewed by subject matter expert
- [ ] User-tested for comprehension

### Publishing
- [ ] Files organized in correct locations
- [ ] Table of contents updated
- [ ] Index updated (if applicable)
- [ ] Changelog updated
- [ ] PR created
- [ ] Documentation deployed

### Post-Publication
- [ ] Announced to users (if major update)
- [ ] Feedback collected
- [ ] Review schedule set

---

## 📈 Success Metrics

### Documentation Quality
- [ ] All examples are runnable
- [ ] All links work
- [ ] Clear and easy to understand
- [ ] Comprehensive coverage

### User Impact
- **Reduced Support Questions**: [Target %]
- **User Satisfaction**: [Target score]
- **Documentation Usage**: [Target views/month]

---

## 🔗 Related Issues/PRs

<!-- Link related work -->

- Relates to #
- Documents #
- Part of #

---

## 📝 Additional Notes

<!-- Any additional context, research, references -->

### References
- [External documentation reference 1]
- [External documentation reference 2]

### Screenshots/Mockups
<!-- Include mockups if planning significant structural changes -->

---

**Metadata**
- **Target Version**: vX.Y.Z
- **Priority**: High | Medium | Low
- **Complexity**: High | Medium | Low
- **Estimated Effort**: [hours/days]
- **Documentation Type**: API | User Guide | Developer Guide | Other
