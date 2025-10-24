---
name: 🔄 Rollback Incident Report
about: Report an incident that requires or resulted from a rollback
title: '🔄 Rollback Incident: [Brief Description]'
labels: ['rollback', 'incident', 'high-priority']
assignees: []
---

## 🔄 Rollback Incident Details

### Incident Summary
- **Incident Type**: Manual Rollback | Automated Rollback | Rollback Failure | Prevented Incident
- **Severity**: 🔴 Critical | 🟡 High | 🟢 Medium | ⚪ Low
- **Status**: 🔴 Active | 🟡 Investigating | 🟢 Resolved | ⚪ Monitoring
- **Detected At**: YYYY-MM-DD HH:MM UTC
- **Resolved At**: YYYY-MM-DD HH:MM UTC (if resolved)

---

## 📋 Rollback Information

### Version Details
- **Source Version**: vX.Y.Z (problematic version)
- **Target Version**: vX.Y.Z (rollback target)
- **Rollback Session ID**: [From workflow logs or git commit]
- **Rollback Method**: Automated | Manual | Partial

### Trigger Information
- **Triggered By**: [Person, System, or Alert]
- **Trigger Reason**: [Brief description of why rollback was initiated]
- **Decision Made At**: YYYY-MM-DD HH:MM UTC
- **Approval From**: [Person or role]

---

## 💥 Impact Assessment

### Production Impact
- [ ] Production services affected
- [ ] User-facing functionality impacted
- [ ] Data integrity concerns
- [ ] Performance degradation
- [ ] Security implications
- [ ] Financial impact

### Affected Components
- Component 1: [Name] - [Impact description]
- Component 2: [Name] - [Impact description]
- Component 3: [Name] - [Impact description]

### User Impact
- **Users Affected**: [Number or percentage]
- **Impact Duration**: [How long users were affected]
- **Impact Severity**: Complete Outage | Degraded Service | Minor Issues
- **Geographic Scope**: Global | Regional ([regions])
- **User Segments Affected**: [Which types of users]

### Business Impact
- **Revenue Impact**: [$amount or %]
- **SLA Violations**: ✅ Yes | ❌ No
- **Customer Complaints**: [Number]
- **Reputation Impact**: High | Medium | Low

---

## ⏱️ Timeline

### Detection
- **YYYY-MM-DD HH:MM UTC**: [First indication of issue]
- **YYYY-MM-DD HH:MM UTC**: [Issue confirmed]
- **YYYY-MM-DD HH:MM UTC**: [Escalation to on-call]

### Response
- **YYYY-MM-DD HH:MM UTC**: [Incident response initiated]
- **YYYY-MM-DD HH:MM UTC**: [Rollback decision made]
- **YYYY-MM-DD HH:MM UTC**: [Rollback initiated]

### Rollback Execution
- **YYYY-MM-DD HH:MM UTC**: [Rollback started]
- **YYYY-MM-DD HH:MM UTC**: [Code reverted]
- **YYYY-MM-DD HH:MM UTC**: [Services restarted]
- **YYYY-MM-DD HH:MM UTC**: [Rollback completed]

### Resolution
- **YYYY-MM-DD HH:MM UTC**: [Service restored]
- **YYYY-MM-DD HH:MM UTC**: [Verification completed]
- **YYYY-MM-DD HH:MM UTC**: [Incident closed]

### Total Duration
- **Detection to Rollback**: [duration]
- **Rollback Execution**: [duration]
- **Total Incident Duration**: [duration]

---

## 🔍 Root Cause Analysis

### Primary Cause
[Technical explanation of what went wrong]

### Contributing Factors

#### Technical Factors
- **Factor 1**: [Description and how it contributed]
- **Factor 2**: [Description and how it contributed]
- **Factor 3**: [Description and how it contributed]

#### Process Factors
- **Factor 1**: [Process gap or failure]
- **Factor 2**: [Process gap or failure]

#### Human Factors
- **Factor 1**: [Human error or decision]
- **Factor 2**: [Human error or decision]

### Failure Points

#### Why Didn't Safeguards Catch This?
- **Testing**: [What testing missed this issue]
- **Monitoring**: [Why monitoring didn't detect it earlier]
- **Deployment**: [What deployment safeguards failed]
- **Code Review**: [What code review missed]

### Code/Configuration at Fault

**File**: `path/to/problematic/file.js` (lines X-Y)

```javascript
// Problematic code that caused the incident
const problematic = () => {
  // code that caused issue
};
```

**Why It Failed**: [Technical explanation]

---

## ✅ Resolution Actions

### Immediate Actions Taken
- [ ] Automated rollback executed successfully
- [ ] Manual rollback executed
- [ ] Manual intervention required
- [ ] Database rollback performed
- [ ] Configuration restored
- [ ] Cache cleared
- [ ] Monitoring alerts configured
- [ ] User notification sent

### Rollback Procedure

```bash
# Commands executed for rollback
git log --oneline -5

# Revert to previous version
git revert <commit-sha>

# Or hard reset (if necessary)
git reset --hard <target-commit>

# Rebuild and deploy
npm run build
npm run deploy

# Verify rollback
npm run verify
```

### Verification Steps
1. [Verification step 1]
2. [Verification step 2]
3. [Verification step 3]

### Service Restoration
- **Services Restarted**: [List of services]
- **Databases Restored**: [List of databases]
- **Caches Cleared**: [List of caches]
- **Configuration Reverted**: [List of configs]

---

## 🛡️ Prevention Measures

### Immediate Actions (Next 24-48 Hours)
- [ ] Hot fix deployed (if applicable)
- [ ] Monitoring enhanced
- [ ] Alerting rules updated
- [ ] Documentation updated
- [ ] Team briefed

### Short-Term Improvements (Next 1-2 Weeks)
- [ ] Add specific tests for this scenario
- [ ] Improve deployment safeguards
- [ ] Enhance monitoring coverage
- [ ] Update runbooks
- [ ] Review similar code for same issue

### Long-Term Improvements (Next 1-3 Months)
- [ ] Process improvement: [Specific improvement]
- [ ] System improvement: [Specific improvement]
- [ ] Training: [What training is needed]
- [ ] Tool improvement: [What tools needed]
- [ ] Architecture change: [If architectural change needed]

---

## 📚 Lessons Learned

### What Went Well
1. [Positive aspect 1]
2. [Positive aspect 2]
3. [Positive aspect 3]

### What Went Wrong
1. [Problem 1 and why it happened]
2. [Problem 2 and why it happened]
3. [Problem 3 and why it happened]

### What We Learned
1. **Lesson 1**: [Key takeaway]
   - **Action**: [Specific action to take]

2. **Lesson 2**: [Key takeaway]
   - **Action**: [Specific action to take]

3. **Lesson 3**: [Key takeaway]
   - **Action**: [Specific action to take]

### Surprises
- [Anything unexpected that happened]
- [Anything that worked differently than expected]

---

## 🔄 Follow-up Actions

### Technical Follow-ups
- [ ] Create ticket for permanent fix
- [ ] Update rollback procedures
- [ ] Improve monitoring/alerting
- [ ] Enhance testing procedures
- [ ] Review deployment pipeline
- [ ] Update documentation

**Permanent Fix Ticket**: #[issue number]

### Process Follow-ups
- [ ] Update incident response runbook
- [ ] Review and update deployment checklist
- [ ] Conduct team retrospective
- [ ] Update team training
- [ ] Review similar systems for same issue

### Communication Follow-ups
- [ ] Team notified
- [ ] Management informed
- [ ] Users communicated (if applicable)
- [ ] Post-mortem scheduled
- [ ] Post-mortem completed
- [ ] Findings shared

---

## 👥 Stakeholder Communication

### Internal Communication

#### Team Notification
- **Channel**: [Slack, Email, etc.]
- **Time**: YYYY-MM-DD HH:MM UTC
- **Message**: [Summary of what was communicated]

#### Management Notification
- **Stakeholders Notified**: [Names/roles]
- **Time**: YYYY-MM-DD HH:MM UTC
- **Format**: Email | Meeting | Report

### External Communication

#### User Communication
- [ ] Status page updated
- [ ] Email notification sent
- [ ] In-app notification displayed
- [ ] Social media update posted

**Message Template Used**: [Link or summary]

#### Post-Mortem
- **Scheduled For**: YYYY-MM-DD HH:MM UTC
- **Attendees**: [List of attendees]
- **Format**: Virtual | In-Person
- **Facilitator**: [Name]

---

## 📊 Metrics & Indicators

### Incident Metrics
- **Time to Detect**: [duration from incident to detection]
- **Time to Respond**: [duration from detection to response]
- **Time to Rollback**: [duration from decision to rollback complete]
- **Time to Resolve**: [total incident duration]
- **MTTR (Mean Time to Recovery)**: [duration]

### Impact Metrics
- **Affected Users**: [number or percentage]
- **Failed Requests**: [number]
- **Error Rate**: [percentage during incident]
- **Performance Degradation**: [percentage]
- **Downtime**: [duration]

### Rollback Metrics
- **Rollback Success**: ✅ Success | ⚠️ Partial | ❌ Failed
- **Rollback Duration**: [duration]
- **Data Loss**: ✅ Yes | ❌ No
- **Required Manual Intervention**: ✅ Yes | ❌ No

---

## 🔗 Related Issues/PRs

<!-- Link related work -->

- **Problematic PR**: #[PR that caused issue]
- **Rollback PR**: #[PR that performed rollback]
- **Permanent Fix**: #[PR with permanent fix]
- **Related Incidents**: #[Other related incidents]

---

## 📎 Rollback Artifacts

### Workflow Runs
- **Rollback Workflow**: [Link to workflow run]
- **Deployment Workflow**: [Link to original deployment]

### Logs & Monitoring
- **Error Logs**: [Link to logs]
- **Monitoring Dashboard**: [Link to dashboard]
- **Alert History**: [Link to alerts]

### Communication Records
- **Incident Channel**: [Link to Slack thread or email thread]
- **Status Page**: [Link to status page updates]

### Documents
- **Runbook Used**: [Link to runbook]
- **Post-Mortem**: [Link to post-mortem doc]

---

## ✅ Incident Closure Checklist

### Resolution Verification
- [ ] All services restored to normal
- [ ] All users can access system
- [ ] No residual errors or degradation
- [ ] Monitoring shows healthy state
- [ ] SLAs met post-recovery

### Documentation
- [ ] Incident timeline documented
- [ ] Root cause analysis complete
- [ ] Lessons learned documented
- [ ] Runbooks updated
- [ ] Knowledge base updated

### Follow-up
- [ ] All follow-up tickets created
- [ ] Responsible owners assigned
- [ ] Due dates set
- [ ] Post-mortem completed
- [ ] Findings shared with team

### Prevention
- [ ] Immediate safeguards implemented
- [ ] Monitoring enhanced
- [ ] Alerts configured
- [ ] Testing improved
- [ ] Deployment process updated

---

## 📝 Additional Notes

<!-- Any additional context, screenshots, or information -->

### Screenshots
<!-- Include relevant screenshots of errors, monitoring dashboards, etc. -->

### External References
- [Link to monitoring dashboard]
- [Link to error tracking]
- [Link to deployment system]

---

**Metadata**
- **Incident ID**: INC-[number]
- **Severity**: Critical | High | Medium | Low
- **Category**: Code Issue | Configuration Issue | Infrastructure Issue | External Dependency
- **Detection Method**: Monitoring | User Report | Manual Discovery | Automated Test
- **Rollback Success**: ✅ Complete | ⚠️ Partial | ❌ Failed
