---
name: 🐝 Swarm Task
about: Create a task for AI swarm processing and multi-agent coordination
title: '[SWARM] Clear, Descriptive Title'
labels: ['swarm-task', 'swarm-ready']
assignees: ''
---

## 🐝 Swarm Task Overview

<!-- High-level description of the multi-agent task -->

[Describe the complex task that requires swarm coordination]

---

## 🎯 Objectives

### Primary Goals
- [ ] **Goal 1**: [Specific, measurable goal]
- [ ] **Goal 2**: [Specific, measurable goal]
- [ ] **Goal 3**: [Specific, measurable goal]

### Success Metrics
- **Metric 1**: [How success will be measured]
- **Metric 2**: [How success will be measured]
- **Target Completion**: [Specific target]

---

## 🏗️ Swarm Configuration

### Topology Selection
- **Selected Topology**: ⚪ Mesh | ⚪ Hierarchical | ⚪ Ring | ⚪ Star

**Reasoning**: [Why this topology was chosen for this task]

#### Topology Details

<details>
<summary><strong>Mesh Topology</strong> (Click to expand)</summary>

**Best For**: Parallel, independent tasks
- Peer-to-peer coordination
- No single point of failure
- Flexible agent communication
- Ideal for: Feature development, parallel testing, distributed analysis

</details>

<details>
<summary><strong>Hierarchical Topology</strong> (Click to expand)</summary>

**Best For**: Coordinated workflows with dependencies
- Queen-led coordination
- Worker specialization
- Clear delegation and reporting
- Ideal for: Release management, complex builds, stage-gate processes

</details>

<details>
<summary><strong>Ring Topology</strong> (Click to expand)</summary>

**Best For**: Sequential processing with handoffs
- Sequential task flow
- State passing between agents
- Pipeline-style processing
- Ideal for: CI/CD pipelines, data transformation, review workflows

</details>

<details>
<summary><strong>Star Topology</strong> (Click to expand)</summary>

**Best For**: Centralized coordination with distributed workers
- Central hub coordination
- Specialized worker agents
- Task routing and aggregation
- Ideal for: Issue triage, load distribution, orchestration

</details>

### Swarm Parameters
- **Max Agents**: [number] (recommended: 3-8 for most tasks)
- **Coordination Mode**: Synchronous | Asynchronous | Hybrid
- **Memory Sharing**: Enabled | Disabled
- **Progress Tracking**: Real-time | Periodic

---

## 👥 Agent Assignment

### Agent Types & Roles

<details>
<summary><strong>Coordinator/Queen</strong> (Primary orchestrator)</summary>

**Responsibilities**:
- Overall task orchestration
- Progress tracking and reporting
- Resource allocation
- Conflict resolution
- Final validation

**Skills Required**: Project management, system architecture
**Priority**: Critical

</details>

<details>
<summary><strong>Analyst</strong> (Research and analysis)</summary>

**Responsibilities**:
- Requirement analysis
- Code analysis
- Performance profiling
- Risk assessment
- Technical recommendations

**Skills Required**: Analysis, pattern recognition
**Priority**: High

</details>

<details>
<summary><strong>Coder/Developer</strong> (Implementation)</summary>

**Responsibilities**:
- Code implementation
- Feature development
- Bug fixing
- Code refactoring
- API development

**Skills Required**: Programming, problem-solving
**Priority**: High

</details>

<details>
<summary><strong>Tester/QA</strong> (Quality assurance)</summary>

**Responsibilities**:
- Test creation
- Test execution
- Bug verification
- Quality validation
- Coverage analysis

**Skills Required**: Testing, quality assurance
**Priority**: High

</details>

<details>
<summary><strong>Architect</strong> (System design)</summary>

**Responsibilities**:
- System architecture
- Design patterns
- Technology selection
- Integration design
- Scalability planning

**Skills Required**: Architecture, system design
**Priority**: Medium-High

</details>

<details>
<summary><strong>Security Specialist</strong> (Security review)</summary>

**Responsibilities**:
- Security audit
- Vulnerability scanning
- Access control review
- Compliance checking
- Security recommendations

**Skills Required**: Security, compliance
**Priority**: Medium-High

</details>

<details>
<summary><strong>Performance Engineer</strong> (Optimization)</summary>

**Responsibilities**:
- Performance profiling
- Optimization recommendations
- Load testing
- Bottleneck analysis
- Resource optimization

**Skills Required**: Performance tuning
**Priority**: Medium

</details>

<details>
<summary><strong>Documenter</strong> (Documentation)</summary>

**Responsibilities**:
- Documentation creation
- API documentation
- User guides
- Examples and tutorials
- Knowledge transfer

**Skills Required**: Technical writing
**Priority**: Medium

</details>

---

## 📋 Task Decomposition

### Phase 1: Planning & Design
**Duration**: [estimated time]
**Agents**: Coordinator, Architect, Analyst

- [ ] **Task 1.1**: [Specific task]
  - **Owner**: [Agent type]
  - **Priority**: High | Medium | Low
  - **Dependencies**: [None or list dependencies]
  - **Estimated Effort**: [hours/days]

- [ ] **Task 1.2**: [Specific task]
  - **Owner**: [Agent type]
  - **Priority**: High | Medium | Low
  - **Dependencies**: [None or list dependencies]
  - **Estimated Effort**: [hours/days]

---

### Phase 2: Implementation
**Duration**: [estimated time]
**Agents**: Coder, Database Architect, Frontend Developer

- [ ] **Task 2.1**: [Specific task]
  - **Owner**: [Agent type]
  - **Priority**: High | Medium | Low
  - **Dependencies**: [Task 1.1, etc.]
  - **Estimated Effort**: [hours/days]

- [ ] **Task 2.2**: [Specific task]
  - **Owner**: [Agent type]
  - **Priority**: High | Medium | Low
  - **Dependencies**: [Task 2.1, etc.]
  - **Estimated Effort**: [hours/days]

---

### Phase 3: Testing & Validation
**Duration**: [estimated time]
**Agents**: Tester, Security Specialist, Performance Engineer

- [ ] **Task 3.1**: [Specific task]
  - **Owner**: [Agent type]
  - **Priority**: High | Medium | Low
  - **Dependencies**: [Task 2.x, etc.]
  - **Estimated Effort**: [hours/days]

- [ ] **Task 3.2**: [Specific task]
  - **Owner**: [Agent type]
  - **Priority**: High | Medium | Low
  - **Dependencies**: [Task 3.1, etc.]
  - **Estimated Effort**: [hours/days]

---

### Phase 4: Documentation & Finalization
**Duration**: [estimated time]
**Agents**: Documenter, Coordinator

- [ ] **Task 4.1**: [Specific task]
  - **Owner**: [Agent type]
  - **Priority**: High | Medium | Low
  - **Dependencies**: [Task 3.x, etc.]
  - **Estimated Effort**: [hours/days]

- [ ] **Task 4.2**: [Specific task]
  - **Owner**: [Agent type]
  - **Priority**: High | Medium | Low
  - **Dependencies**: [Task 4.1, etc.]
  - **Estimated Effort**: [hours/days]

---

## 🔄 Coordination Protocol

### Pre-Task Hooks

```bash
# Initialize swarm environment
npx claude-flow@alpha hooks pre-task \
  --description "Swarm Task: [task title]" \
  --session-id "swarm-[task-slug]-[session-id]"

# Restore any prior context
npx claude-flow@alpha hooks session-restore \
  --session-id "swarm-[task-slug]-[session-id]"

# Initialize swarm topology
npx claude-flow@alpha mcp swarm_init \
  --topology "[selected-topology]" \
  --max-agents [number]
```

---

### During Task Execution

#### Progress Tracking
```bash
# Each agent reports progress
npx claude-flow@alpha hooks notify \
  --message "[Agent type]: Completed [specific milestone]" \
  --memory-key "swarm/[task-slug]/progress/[agent-id]"

# Store intermediate results
npx claude-flow@alpha hooks post-edit \
  --file "[path/to/file]" \
  --memory-key "swarm/[task-slug]/artifacts/[component]"

# Share data between agents
npx claude-flow@alpha mcp memory_usage \
  --action "store" \
  --key "swarm/[task-slug]/shared/[data-key]" \
  --value "[data-value]"
```

#### Agent Communication
```bash
# Coordinator broadcasts to all agents
npx claude-flow@alpha hooks notify \
  --message "Coordinator: Phase 2 ready to start" \
  --broadcast true

# Agent requests help from another agent
npx claude-flow@alpha hooks notify \
  --message "Coder → Tester: Ready for integration testing" \
  --target "tester"
```

---

### Post-Task Hooks

```bash
# Finalize and export results
npx claude-flow@alpha hooks post-task \
  --task-id "swarm-[task-slug]" \
  --export-metrics true

# Store completion state
npx claude-flow@alpha hooks session-end \
  --session-id "swarm-[task-slug]-[session-id]" \
  --export-path "./swarm-reports/[task-slug]"

# Generate swarm report
npx claude-flow@alpha mcp swarm_status \
  --format "detailed" > "./swarm-reports/[task-slug]-status.json"
```

---

## 📊 Progress Tracking

### Automated Updates
This section will be automatically updated by swarm agents during execution.

#### Completion Status
- **Overall Progress**: 0% (auto-updated)
- **Phase 1 (Planning)**: ⏳ Not Started
- **Phase 2 (Implementation)**: ⏳ Not Started
- **Phase 3 (Testing)**: ⏳ Not Started
- **Phase 4 (Documentation)**: ⏳ Not Started

#### Agent Status
| Agent | Status | Current Task | Progress | Last Update |
|-------|--------|--------------|----------|-------------|
| Coordinator | ⏳ Idle | - | - | - |
| Analyst | ⏳ Idle | - | - | - |
| Coder | ⏳ Idle | - | - | - |
| Tester | ⏳ Idle | - | - | - |

#### Metrics
- **Tasks Completed**: 0 / [total]
- **Test Coverage**: 0%
- **Code Quality Score**: -
- **Performance Benchmarks**: -

---

## ⚠️ Risks & Blockers

### Identified Risks

#### Risk 1: [Title]
- **Probability**: High | Medium | Low
- **Impact**: High | Medium | Low
- **Mitigation**: [Strategy to mitigate]
- **Owner**: [Agent responsible]

#### Risk 2: [Title]
- **Probability**: High | Medium | Low
- **Impact**: High | Medium | Low
- **Mitigation**: [Strategy to mitigate]
- **Owner**: [Agent responsible]

### Active Blockers
<!-- Updated in real-time as blockers arise -->

- [ ] **Blocker 1**: [Description]
  - **Blocking**: [What it's blocking]
  - **Owner**: [Who's resolving]
  - **ETA**: [Expected resolution]

---

## 🔗 Dependencies

### External Dependencies
- **Dependency 1**: [Description and why needed]
- **Dependency 2**: [Description and why needed]

### Internal Dependencies
- **Component A**: [What's needed from it]
- **Component B**: [What's needed from it]

### Blocking Issues
- Blocked by #[issue number]: [Description]
- Blocks #[issue number]: [Description]

---

## ✅ Acceptance Criteria

### Functional Requirements
- [ ] Requirement 1: [Specific, testable requirement]
- [ ] Requirement 2: [Specific, testable requirement]
- [ ] Requirement 3: [Specific, testable requirement]

### Quality Requirements
- [ ] Test Coverage ≥ 90%
- [ ] All tests passing
- [ ] Performance benchmarks met
- [ ] Security audit passed
- [ ] Documentation complete
- [ ] Code review approved

### Swarm Coordination Requirements
- [ ] All agents completed assigned tasks
- [ ] Cross-agent integration verified
- [ ] Coordination overhead documented
- [ ] Lessons learned captured

---

## 📈 Success Metrics

### Performance Metrics
- **Task Completion Time**: [target vs actual]
- **Swarm Efficiency**: [coordination overhead]
- **Agent Utilization**: [% time agents active]

### Quality Metrics
- **Code Quality Score**: [target score]
- **Test Coverage**: [target %]
- **Bug Density**: [bugs per KLOC]

### Coordination Metrics
- **Agent Collaboration**: [cross-agent interactions]
- **Communication Efficiency**: [signal-to-noise ratio]
- **Synchronization Overhead**: [% time coordinating]

---

## 📚 Documentation & Artifacts

### Required Documentation
- [ ] Architecture documentation
- [ ] API documentation
- [ ] User guides
- [ ] Developer guides
- [ ] Swarm coordination report

### Generated Artifacts
- [ ] Code implementations
- [ ] Test suites
- [ ] Performance benchmarks
- [ ] Swarm metrics report
- [ ] Lessons learned document

---

## ✅ Completion Checklist

### Planning
- [ ] Task scope defined
- [ ] Swarm topology selected
- [ ] Agents assigned
- [ ] Tasks decomposed
- [ ] Timeline estimated

### Execution
- [ ] All phases completed
- [ ] All tasks finished
- [ ] Cross-agent integration verified
- [ ] Quality criteria met

### Validation
- [ ] Acceptance criteria met
- [ ] Tests passing (90%+ coverage)
- [ ] Performance targets achieved
- [ ] Security validated
- [ ] Documentation complete

### Finalization
- [ ] Code merged
- [ ] Swarm metrics exported
- [ ] Lessons learned documented
- [ ] Follow-up tasks created
- [ ] Issue closed

---

## 🔗 Related Issues/PRs

<!-- Link related work -->

- Relates to #
- Depends on #
- Blocks #
- Part of epic #

---

## 📝 Additional Notes

<!-- Any additional context, diagrams, research -->

### Swarm Topology Diagram
```
[Visual representation of chosen topology with agent connections]
```

### Coordination Insights
<!-- Filled in post-completion -->
- [Key insight about swarm coordination]
- [Challenge encountered and resolution]
- [Recommendation for future swarm tasks]

---

**Metadata**
- **Swarm Session ID**: swarm-[task-slug]-[session-id]
- **Topology**: mesh | hierarchical | ring | star
- **Agent Count**: [number]
- **Estimated Duration**: [hours/days/weeks]
- **Priority**: Critical | High | Medium | Low
- **Complexity**: High | Medium | Low
