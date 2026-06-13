# **CONTRIBUTION LOG AI 301**

# Contribution 1: [onert] Enhance python API - inference

**Contribution Number:** 1 

**Student:** Srithan Andra

**Issue:** [GitHub issue link](https://github.com/valhalla/valhalla/issues/3412)

**Status:** Phase 1 Complete

---

## Why I Chose This Issue

I chose this issue because it was one of the more basic ones that I could understand. I also wanted to start off with a topic that I have some experience with. I have built APIs before, especially in Python and C++, which are the languages that this project uses. The goal of double checking the edge IDs and making sure that the lat/lon coordinates are properly used in the valhalla loki namespace is a good start for me as I haven't done much open-source work before. In summary, I wanted to start off with a topic that I have a good understanding of and built up from there.

---

## Understanding the Issue

### Problem Description

Basically, when a developer is using loki, which is the coordinate to graph conversion stage, they are forced to pass in raw lat/lon coordinate data. This is just unnecessary computation and wastes performance. Also, the issue pointed out that a missing capability to easily find edge IDs from a trajectory without starting a new map-making request is also needed.

### Expected Behavior

Users should be able to provide a list of edge IDs with the standard coordinates. The code should skip the normal lookup and instead use the coordinates with the edge geometry to calculate the distances. Lastly, the code should fall back to a normal lookup if the given IDs are invalid.

### Current Behavior

None of the above idea is implemented so far.

### Affected Components

Most of the affected components are in the src/loki directory under the search.cc file. I think there should be changes in the src/worker.cc file as well.

---

## Reproduction Process

### Environment Setup

[Notes on setting up your local development environment - challenges you faced, how you solved them]

### Steps to Reproduce

1. [Step 1]
2. [Step 2]
3. [Observed result]

### Reproduction Evidence

- **Commit showing reproduction:** [Link to commit in your fork]
- **Screenshots/logs:** [If applicable]
- **My findings:** [What you discovered during reproduction]

---

## Solution Approach

### Analysis

[Your analysis of the root cause - what's causing the issue?]

### Proposed Solution

[High-level description of your fix approach]

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** [Restate the problem]

**Match:** [What similar patterns/solutions exist in the codebase?]

**Plan:** [Step-by-step implementation plan]
1. [Modify file X to do Y]
2. [Add function Z]
3. [Update tests]

**Implement:** [Link to your branch/commits as you work]

**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]

**Evaluate:** [How will you verify it works?]

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
