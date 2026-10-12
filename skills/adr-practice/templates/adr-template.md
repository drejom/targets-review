# ADR-XXXX: [Decision Title]

## Status
[Proposed | Accepted | Deprecated | Superseded by ADR-YYYY]

**Date**: [YYYY-MM-DD]
**Decision Confidence**: [High | Medium | Low]
**Impact Scope**: [Core Architecture | Module-specific | Infrastructure | Performance]

## Context
<!-- Describe the forces at play, including technical, organizational, and project-local
ones. These forces are probably in tension, and should be called out as such. The language
in this section is value-neutral — it describes facts, not conclusions. -->

### Problem Statement
[What problem are we trying to solve?]

### Requirements
[What are the functional and non-functional requirements?]

### Constraints
- **Language/platform constraints**: [e.g. CRAN compliance, Python version floor, browser support]
- **Technical constraints**: [external systems this decision must work with]
- **Performance constraints**: [data scale, latency budget, resource limits]

## Decision
<!-- This section describes the response to the forces above. Full sentences, active voice:
"We will ..." -->

[What is the change being proposed or agreed to?]

## Technical Approach

### Key Components
- **Component 1**: [Description and role]
- **Component 2**: [Description and role]

### Implementation Details
- **Structure**: [Which files/modules are involved]
- **Dependencies**: [New packages/libraries added]
- **API Design**: [Function signatures, exported interfaces]
- **Data Flow**: [How data moves through the system]

### Integration Points
<!-- How this integrates with existing architecture -->
- [System 1]: [impact]
- [System 2]: [impact]

## Consequences
<!-- All consequences, not just the positive ones. A decision may have positive, negative,
and neutral consequences, and all of them affect the project going forward. -->

### Positive
- [Benefits gained]
- [Problems solved]

### Negative
- [Complexity added]
- [Trade-offs accepted]

### Risks and Mitigation
- **Risk**: [Description]
  - **Likelihood**: [High | Medium | Low]
  - **Impact**: [High | Medium | Low]
  - **Mitigation**: [How this is addressed]

## Alternatives Considered

### Alternative 1: [Name]
- **Description**: [Brief description]
- **Pros**: [Benefits]
- **Cons**: [Drawbacks]
- **Reason for rejection**: [Why this wasn't chosen]

### Alternative 2: [Name]
- **Description**: [Brief description]
- **Pros**: [Benefits]
- **Cons**: [Drawbacks]
- **Reason for rejection**: [Why this wasn't chosen]

## Implementation Notes

### Testing Strategy
[Unit tests, integration tests, fixtures needed]

### Documentation
[Doc updates needed as a result of this decision]

### Deployment / execution environment considerations
[Any environment this decision behaves differently in — CI vs local, a cluster vs a laptop,
an installed package vs source tree. See `principle-environment.md` if more than one
environment is relevant.]

### Monitoring and Validation
- [How will we know if this decision is working?]
- [What would indicate it should be revisited?]

## Related Decisions
<!-- Links to related ADRs -->
- Related to ADR-YYYY: [Brief description]
- Builds on ADR-ZZZZ: [Brief description]

## References
<!-- Links to supporting issues, PRs, external documentation -->
- Issue: [link]
- Pull Request: [link]
- External documentation: [links]

---

**Decision Maker(s)**: [Who made this decision]
**Contributors**: [Who provided input]
**Next Review Date**: [When should this be revisited, or what event should trigger a revisit]
