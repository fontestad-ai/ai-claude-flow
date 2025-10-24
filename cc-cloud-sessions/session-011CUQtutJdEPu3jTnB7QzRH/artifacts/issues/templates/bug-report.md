---
name: 🐛 Bug Report
about: Report a defect, error, or unexpected behavior
title: '[BUG] Clear, Descriptive Title'
labels: ['bug']
assignees: ''
---

## 🐛 Bug Description

<!-- Provide a clear and concise description of the bug -->

[Describe what's not working as expected]

---

## 📊 Impact Assessment

- **Severity**: 🔴 Critical | 🟡 High | 🟢 Medium | ⚪ Low
- **Affected Users**: [number, percentage, or description]
- **Data Loss Risk**: ✅ Yes | ❌ No
- **Workaround Available**: ✅ Yes | ❌ No
- **Discovery Method**: [How was this bug found]

### Impact Details

- **Production Impact**: [Description of production effects]
- **User-Facing**: ✅ Yes | ❌ No
- **Performance Degradation**: [% or description]
- **Security Implications**: [Any security concerns]

---

## 🔍 Root Cause Analysis

<!-- Technical explanation of why the bug occurs -->

### Primary Cause
[Main technical reason for the bug]

### Contributing Factors
- Factor 1: [Description]
- Factor 2: [Description]
- Factor 3: [Description]

### Code Location
**File**: `path/to/file.js` (lines X-Y)

```javascript
// Problematic code snippet
const example = "code here";
```

---

## 🔄 Reproduction Steps

<!-- Detailed steps to reproduce the behavior -->

1. [Step 1]
2. [Step 2]
3. [Step 3]
4. [Step 4]

### Expected Behavior
[What should happen]

### Actual Behavior
[What actually happens]

### Minimal Reproducible Example

```javascript
// Minimal code to demonstrate the bug
const reproduce = () => {
  // code here
};
```

---

## 🌍 Environment

- **Package Version**: vX.Y.Z
- **Node.js Version**: vX.Y.Z
- **npm/yarn Version**: vX.Y.Z
- **Operating System**: [OS and version]
- **Platform**: [Additional platform details]

### Additional Context

- **Related Configuration**: [Any relevant config]
- **Dependencies**: [Relevant dependency versions]
- **Environment Variables**: [Any relevant env vars]

---

## ✅ Solution Implemented

<!-- Detailed explanation of the fix (fill after implementation) -->

### Approach
[High-level approach to fixing the bug]

### Implementation Details

#### Changes Made

**File**: `path/to/file.js`
```javascript
// Fixed code
const fixed = "corrected code";
```

**Reasoning**: [Why this approach was chosen]

### Backward Compatibility
- ✅ Fully backward compatible
- ⚠️ Minor breaking changes (with migration path)
- 🔴 Breaking changes (major version bump)

---

## 🧪 Testing

### Unit Tests
- **Test File**: `tests/path/to/test.js`
- **Coverage**: X% (target: 90%+)
- **New Tests Added**: X

```javascript
// Example test case
describe('Bug Fix: Description', () => {
  it('should work correctly', () => {
    // test code
  });
});
```

### Integration Tests
- [ ] Integration test 1
- [ ] Integration test 2

### Manual Testing
- [ ] Tested scenario 1
- [ ] Tested scenario 2
- [ ] Verified edge cases

---

## ✔️ Verification Steps

<!-- Steps to verify the fix works -->

1. [Verification step 1]
2. [Verification step 2]
3. [Verification step 3]

### Test Commands

```bash
# Run specific test
npm test -- --grep "bug fix"

# Verify build
npm run build

# Manual verification
npx claude-flow [command to verify]
```

---

## 📈 Performance Impact

### Before Fix
- **Metric 1**: [value]
- **Metric 2**: [value]

### After Fix
- **Metric 1**: [value]
- **Metric 2**: [value]
- **Improvement**: [% or description]

---

## 🐝 Swarm Coordination (if applicable)

**Required**: ✅ Yes | ❌ No

### Swarm Configuration
- **Topology**: mesh | hierarchical | ring | star
- **Max Agents**: [number]

### Agent Assignment
- **Debugger**: Root cause investigation
- **Coder**: Fix implementation
- **Tester**: Test creation and validation
- **Reviewer**: Code review and verification

### Coordination Protocol

```bash
# Pre-task
npx claude-flow@alpha hooks pre-task --description "Bug fix: [title]"

# During task
npx claude-flow@alpha hooks notify --message "Fix implemented"

# Post-task
npx claude-flow@alpha hooks post-task --task-id "bug-[number]"
```

---

## ✅ Completion Checklist

### Implementation
- [ ] Root cause identified and documented
- [ ] Fix implemented and tested
- [ ] Code reviewed and approved
- [ ] Documentation updated

### Testing
- [ ] Unit tests written (90%+ coverage)
- [ ] Integration tests passing
- [ ] Manual testing completed
- [ ] Edge cases verified
- [ ] Performance impact assessed

### Release
- [ ] Changelog updated
- [ ] Version bumped appropriately
- [ ] Migration guide created (if needed)
- [ ] PR created and linked
- [ ] Deployed to staging
- [ ] Verified in production

---

## 🔗 Related Issues/PRs

<!-- Link related work -->

- Relates to #
- Depends on #
- Blocks #
- Duplicate of #

---

## 📝 Additional Notes

<!-- Any additional context, screenshots, logs, etc. -->

### Logs/Stack Traces

```
[Paste relevant logs or stack traces]
```

### Screenshots

<!-- If applicable, add screenshots to help explain the problem -->

---

**Metadata**
- **Fix Version**: vX.Y.Z (proposed)
- **Priority**: Critical | High | Medium | Low
- **Type**: Bug Fix
- **Module**: [Component/Module name]
- **Regression**: ✅ Yes (introduced in vX.Y.Z) | ❌ No
