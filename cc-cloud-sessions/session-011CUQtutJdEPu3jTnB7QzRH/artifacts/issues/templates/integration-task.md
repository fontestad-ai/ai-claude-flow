---
name: 🔄 Integration Task
about: Complex integration between components or systems
title: '[INTEGRATION] Clear, Descriptive Title'
labels: ['integration']
assignees: ''
---

## 🔄 Integration Overview

<!-- High-level description of what's being integrated -->

[Describe the integration task and components involved]

---

## 🎯 Integration Objectives

- [ ] **Objective 1**: [Specific integration goal]
- [ ] **Objective 2**: [Specific integration goal]
- [ ] **Objective 3**: [Specific integration goal]

### Success Criteria
- **Criterion 1**: [How success will be measured]
- **Criterion 2**: [How success will be measured]
- **Criterion 3**: [How success will be measured]

---

## 🏗️ Components Involved

### Component A: [Name]
- **Repository**: [repository name/link]
- **Version**: vX.Y.Z
- **Role**: [Role in integration]
- **Owner**: [Team/person responsible]

### Component B: [Name]
- **Repository**: [repository name/link]
- **Version**: vX.Y.Z
- **Role**: [Role in integration]
- **Owner**: [Team/person responsible]

### Component C: [Name] (if applicable)
- **Repository**: [repository name/link]
- **Version**: vX.Y.Z
- **Role**: [Role in integration]
- **Owner**: [Team/person responsible]

---

## 📋 Integration Areas

### 1. Dependencies
- [ ] Update package.json in all repositories
- [ ] Verify version compatibility
- [ ] Resolve dependency conflicts
- [ ] Update import statements
- [ ] Configure monorepo/workspace (if applicable)

**Dependency Matrix**:
| Component | Requires | Version | Status |
|-----------|----------|---------|--------|
| Component A | Component B | ^X.Y.Z | ⏳ Pending |
| Component B | Shared Lib | ^X.Y.Z | ⏳ Pending |

---

### 2. Functionality Integration

#### Core Features
- [ ] Feature A integration
  - **Description**: [What's being integrated]
  - **Approach**: [How it will be integrated]
  - **Testing**: [How to verify]

- [ ] Feature B integration
  - **Description**: [What's being integrated]
  - **Approach**: [How it will be integrated]
  - **Testing**: [How to verify]

#### API Compatibility
- [ ] Verify API contracts match
- [ ] Update API endpoints
- [ ] Add backward compatibility layer (if needed)
- [ ] Document API changes

```javascript
// API integration example
// Component A calls Component B
const componentB = require('component-b');

async function integratedFeature() {
  const result = await componentB.method({
    parameter: value
  });
  return processResult(result);
}
```

---

### 3. Data Flow & Communication

#### Data Exchange
```javascript
// Data flow diagram in code
/**
 * Component A → Transform → Component B → Process → Output
 *
 * A.getData()
 *   ↓
 * transform(data)
 *   ↓
 * B.processData(transformed)
 *   ↓
 * result
 */
```

#### Communication Protocol
- **Method**: REST API | GraphQL | Message Queue | Event Bus | Direct Call
- **Format**: JSON | Protocol Buffers | XML
- **Authentication**: API Key | OAuth | JWT | mTLS
- **Error Handling**: Retry Strategy | Circuit Breaker | Fallback

---

### 4. Configuration Management

#### Environment Variables
```bash
# Component A
COMPONENT_B_URL=https://api.component-b.example.com
COMPONENT_B_API_KEY=secret_key
INTEGRATION_MODE=production

# Component B
COMPONENT_A_WEBHOOK=https://api.component-a.example.com/webhook
SHARED_SECRET=shared_secret
```

#### Configuration Files
```yaml
# integration-config.yml
integration:
  componentA:
    enabled: true
    endpoint: ${COMPONENT_A_URL}
    timeout: 5000

  componentB:
    enabled: true
    endpoint: ${COMPONENT_B_URL}
    retries: 3

  shared:
    logLevel: info
    environment: production
```

---

### 5. Testing Strategy

#### Unit Tests
- [ ] Component A unit tests (maintain 90%+ coverage)
- [ ] Component B unit tests (maintain 90%+ coverage)
- [ ] Integration helper unit tests

```javascript
// Example unit test for integration
describe('Component Integration', () => {
  it('should call Component B correctly', async () => {
    const mockB = mockComponentB();
    const result = await componentA.useComponentB(mockB);
    expect(result).toBeDefined();
    expect(mockB.method).toHaveBeenCalledWith(expectedParams);
  });
});
```

#### Integration Tests
- [ ] End-to-end data flow test
- [ ] Error handling and recovery test
- [ ] Performance and load test
- [ ] Concurrent operation test

```javascript
// Example integration test
describe('Full Integration Flow', () => {
  it('should complete full workflow', async () => {
    // Setup both components
    const componentA = await setupComponentA();
    const componentB = await setupComponentB();

    // Execute integrated workflow
    const input = createTestInput();
    const result = await integratedWorkflow(input);

    // Verify results
    expect(result.status).toBe('success');
    expect(result.data).toMatchExpectedOutput();
  });
});
```

#### Compatibility Tests
- [ ] Backward compatibility test
- [ ] Version compatibility matrix test
- [ ] Breaking change detection

---

## 🔗 Integration Challenges

### Known Challenges

#### Challenge 1: [Title]
- **Description**: [What's the challenge]
- **Impact**: High | Medium | Low
- **Mitigation**: [How to address it]
- **Workaround**: [Alternative if mitigation fails]

#### Challenge 2: [Title]
- **Description**: [What's the challenge]
- **Impact**: High | Medium | Low
- **Mitigation**: [How to address it]
- **Workaround**: [Alternative if mitigation fails]

### Risk Assessment

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| Version incompatibility | Medium | High | Lock versions, test thoroughly |
| Breaking API changes | Low | High | API versioning, contracts |
| Performance degradation | Medium | Medium | Load testing, optimization |
| Data inconsistency | Low | High | Transaction management, validation |

---

## 🛠️ Migration Path

### From Current State
**Current Architecture**:
```
[Diagram of current state]
Component A (standalone)
Component B (standalone)
```

**Target Architecture**:
```
[Diagram of target state]
Component A ←→ Integration Layer ←→ Component B
```

### Migration Steps

#### Phase 1: Preparation
- [ ] Analyze current implementations
- [ ] Identify breaking changes
- [ ] Create integration design
- [ ] Set up test environments

#### Phase 2: Implementation
- [ ] Implement integration layer
- [ ] Update Component A
- [ ] Update Component B
- [ ] Create configuration management

#### Phase 3: Testing & Validation
- [ ] Run unit tests
- [ ] Run integration tests
- [ ] Performance testing
- [ ] Security testing

#### Phase 4: Deployment
- [ ] Deploy to development
- [ ] Deploy to staging
- [ ] Validation in staging
- [ ] Deploy to production

### Rollback Plan
```bash
# Quick rollback procedure
git revert <integration-commit>
npm install component-a@<previous-version>
npm install component-b@<previous-version>
restart-services
```

---

## 📊 Performance Considerations

### Expected Performance Impact

#### Before Integration
- **Component A**: [metrics]
- **Component B**: [metrics]
- **Total Latency**: [ms]

#### After Integration
- **Component A**: [metrics]
- **Component B**: [metrics]
- **Integration Overhead**: [ms]
- **Total Latency**: [ms]

### Optimization Strategies
- [ ] Cache frequently accessed data
- [ ] Implement connection pooling
- [ ] Use async/await for non-blocking operations
- [ ] Batch requests where possible
- [ ] Implement circuit breakers

---

## 🔒 Security Considerations

### Authentication & Authorization
- [ ] Implement mutual authentication
- [ ] Use encrypted connections (TLS)
- [ ] Rotate credentials regularly
- [ ] Implement least-privilege access

### Data Security
- [ ] Encrypt data in transit
- [ ] Encrypt sensitive data at rest
- [ ] Validate all inputs
- [ ] Sanitize all outputs

### Audit & Compliance
- [ ] Log all integration operations
- [ ] Implement request tracing
- [ ] Set up security monitoring
- [ ] Document compliance requirements

---

## 🐝 Swarm Coordination

**Recommended**: ✅ Yes

### Swarm Configuration
- **Topology**: mesh (parallel development across components)
- **Max Agents**: 6-8

### Agent Assignment

#### Architecture & Design
- **Architect**: Overall integration design
- **Analyst**: Requirements analysis and compatibility check

#### Development
- **Component A Developer**: Updates to Component A
- **Component B Developer**: Updates to Component B
- **Integration Engineer**: Integration layer implementation
- **Database Architect**: Data flow and schema integration

#### Quality Assurance
- **Integration Tester**: End-to-end integration testing
- **Security Specialist**: Security review and testing
- **Performance Engineer**: Load testing and optimization

#### Documentation
- **Documenter**: Integration guides and API documentation

### Task Decomposition

#### Phase 1: Analysis & Design (Week 1)
- [ ] Analyze current state (Analyst)
- [ ] Design integration architecture (Architect)
- [ ] Identify compatibility issues (Analyst)
- [ ] Design data flow (Database Architect)
- [ ] Security planning (Security Specialist)

#### Phase 2: Implementation (Week 2-3)
- [ ] Implement Component A changes (Component A Developer)
- [ ] Implement Component B changes (Component B Developer)
- [ ] Build integration layer (Integration Engineer)
- [ ] Update data schemas (Database Architect)
- [ ] Implement security controls (Security Specialist)

#### Phase 3: Testing & Optimization (Week 4)
- [ ] Integration testing (Integration Tester)
- [ ] Security testing (Security Specialist)
- [ ] Performance testing (Performance Engineer)
- [ ] Bug fixes (All developers)

#### Phase 4: Documentation & Deployment (Week 5)
- [ ] Write integration docs (Documenter)
- [ ] Create migration guide (Documenter)
- [ ] Deploy to staging (Integration Engineer)
- [ ] Final validation (All team)
- [ ] Production deployment (Integration Engineer)

### Coordination Protocol

```bash
# Pre-task: Initialize integration swarm
npx claude-flow@alpha hooks pre-task \
  --description "Integration: ComponentA + ComponentB" \
  --session-id "swarm-integration-a-b"

# During: Coordinate across agents
npx claude-flow@alpha hooks notify \
  --message "Component A integration complete" \
  --memory-key "swarm/integration/componentA/status"

npx claude-flow@alpha hooks notify \
  --message "Component B integration complete" \
  --memory-key "swarm/integration/componentB/status"

# Post-task: Finalize integration
npx claude-flow@alpha hooks post-task \
  --task-id "integration-a-b" \
  --export-metrics true
```

---

## 📚 Documentation Requirements

### Integration Guide
- [ ] Architecture overview
- [ ] Setup instructions
- [ ] Configuration guide
- [ ] API documentation
- [ ] Troubleshooting guide

### API Documentation
- [ ] Integration endpoints
- [ ] Request/response formats
- [ ] Error codes
- [ ] Rate limits
- [ ] Authentication

### Migration Guide
- [ ] Pre-migration checklist
- [ ] Step-by-step migration
- [ ] Verification steps
- [ ] Rollback procedure
- [ ] FAQ

---

## ✅ Completion Checklist

### Planning
- [ ] Integration scope defined
- [ ] Components identified
- [ ] Architecture designed
- [ ] Risks assessed
- [ ] Timeline created

### Implementation
- [ ] Dependencies updated
- [ ] Core integration implemented
- [ ] Configuration complete
- [ ] Security implemented
- [ ] Error handling comprehensive

### Testing
- [ ] Unit tests (90%+ coverage)
- [ ] Integration tests passing
- [ ] Performance tests passing
- [ ] Security tests passing
- [ ] Compatibility verified

### Documentation
- [ ] Integration guide complete
- [ ] API docs updated
- [ ] Migration guide written
- [ ] Troubleshooting guide created
- [ ] Changelog updated

### Deployment
- [ ] Deployed to development
- [ ] Deployed to staging
- [ ] Validated in staging
- [ ] Deployed to production
- [ ] Monitoring configured

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

### Architecture Diagrams
<!-- Include integration architecture diagrams -->

### Compatibility Matrix
| Component A | Component B | Status | Notes |
|-------------|-------------|--------|-------|
| v1.x | v2.x | ❌ Incompatible | Breaking changes |
| v2.x | v2.x | ✅ Compatible | Recommended |
| v2.x | v3.x | ⚠️ Partial | Some features missing |

---

**Metadata**
- **Target Version**: vX.Y.Z
- **Priority**: High | Medium | Low
- **Complexity**: High | Medium | Low
- **Estimated Effort**: [days/weeks]
- **Integration Scope**: [Components involved]
- **Breaking Changes**: ✅ Yes | ❌ No
