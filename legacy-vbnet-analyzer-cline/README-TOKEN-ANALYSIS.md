# Token Consumption Analysis Documentation
## Complete Guide to VB.NET Analyzer Context Window Planning

**Project**: VB.NET Legacy Analyzer
**Context Window**: 128K tokens
**Model**: Claude Haiku 4.5
**Analysis Date**: 2026-03-11
**Status**: Complete and Ready

---

## Quick Start: Choose Your Document

### If You Have 5 Minutes: Start Here
**→ Read**: `TOKEN-CONSUMPTION-NUMBERS.md`
- Quick reference tables
- Phase-by-phase token counts
- Conversation planning matrix
- Simple formulas

**Outcome**: Know how many conversations you need for your project

---

### If You Have 20 Minutes: Planning Phase
**→ Read**: `TOKEN-CONSUMPTION-GUIDE.md` (Section 1-4)
- Token accounting method
- Detailed phase breakdown
- Conversation capacity analysis
- Planning examples (5/20/50 form projects)

**Outcome**: Detailed plan for your specific project size

---

### If You Have 45 Minutes: Complete Understanding
**→ Read**: `TOKEN-CONSUMPTION-DETAILED-ANALYSIS.md`
- Complete technical breakdown
- Tool overhead accounting
- All formulas with derivations
- Adjustment guide for your specific context

**Outcome**: Understand how token consumption is calculated and how to adapt estimates

---

### If You Have 30 Minutes: Technical Deep Dive
**→ Read**: `TOOL-OVERHEAD-AND-CONTEXT-MATH.md`
- Token conversion factors (English vs. Chinese)
- Tool invocation overhead (glob, grep, read, write)
- Context window math and headroom calculations
- Optimization techniques with real numbers

**Outcome**: Understand the technical foundation of token accounting

---

## Document Map

### Summary & Overview Documents

1. **TOKEN-ANALYSIS-INDEX.md** ← Start here if overwhelmed
   - Document index and reading paths
   - Quick facts table
   - Common scenarios (5/20/50 forms)
   - When to reference which document

2. **README-TOKEN-ANALYSIS.md** ← This file
   - Which document to read based on your needs
   - Document relationships
   - Navigation guide

### Quick Reference Documents

3. **TOKEN-CONSUMPTION-NUMBERS.md** ⭐ BEST FOR DECISIONS
   - Phase-by-phase token counts (all scenarios)
   - Forms per conversation limits
   - Project examples (3/6/13 conversations)
   - Formulas for quick calculation
   - When to switch conversations checklist

4. **QUICK-REFERENCE.md**
   - Token budget summary table
   - Conversation planning formula
   - Quick decision tree
   - Phase breakdown checklist
   - Pre-start checklist

### Detailed Reference Documents

5. **TOKEN-CONSUMPTION-GUIDE.md** ⭐ BEST FOR PLANNING
   - Complete token accounting method
   - Phase 1a breakdown (2K-6K tokens)
   - Phase 1b breakdown (2.5K-6.5K tokens)
   - Phase 2 formula: 1.5K + (N × 225)
   - Phase 3 formula: N × 10K per form
   - Phase 4 formula: 3.9K + (N × 2.5K)
   - Single conversation capacity analysis
   - Step-by-step conversation planning
   - Cost optimization tips

6. **TOKEN-CONSUMPTION-DETAILED-ANALYSIS.md** ⭐ BEST FOR UNDERSTANDING
   - Comprehensive technical analysis
   - Token conversion framework
   - Tool invocation overhead breakdown
   - Actual project measurements
   - Phase-by-phase detailed calculations
   - Conversation capacity analysis
   - Complete project examples
   - Context window monitoring
   - Adjustment guide
   - 12,000+ words of technical detail

7. **TOOL-OVERHEAD-AND-CONTEXT-MATH.md** ⭐ BEST FOR DEEP TECHNICAL
   - Token conversion reference (English vs. Chinese)
   - Tool overhead accounting (glob, grep, read, write)
   - Context window math (128K allocation)
   - Headroom calculations with examples
   - Token cost breakdown per operation
   - Optimization opportunities with numbers
   - Monitoring techniques during conversation
   - Troubleshooting guide
   - Technical deep dive (6,000+ words)

### Existing Documents (Updated & Verified)

8. **TOKEN-CONSUMPTION-GUIDE.md** (original)
   - Still valid and useful
   - Verified against new analysis
   - Recommended for users during workflow

9. **ANALYSIS-SUMMARY.md** (original)
   - Executive summary with key findings
   - Three recommended conversation plans (A/B/C)
   - Implementation notes
   - Assumptions and limitations

10. **usage-guide.md** (Section 4 updated)
    - Token consumption estimate table
    - Single conversation capacity analysis
    - Recommended conversation plans
    - When to open new conversation table

---

## Navigation by Use Case

### I'm Starting a New Analysis

**Step 1**: Count your forms
```bash
grep -r "Inherits.*Form" --include="*.vb" | wc -l
```

**Step 2**: Read → `TOKEN-CONSUMPTION-NUMBERS.md`
- Find your form count in the planning matrix
- Note number of conversations needed

**Step 3**: Execute according to plan
- Keep `TOKEN-CONSUMPTION-NUMBERS.md` open
- Reference the "red flags" section during Phase 3

---

### I'm Planning an Analysis Before Starting

**Step 1**: Read → `TOKEN-CONSUMPTION-GUIDE.md` (Sections 1-4)
- Understand the 5-phase workflow
- See token consumption per phase
- Review conversation capacity

**Step 2**: Estimate your project
- Is your project small (< 10 forms)?
- Medium (10-30 forms)?
- Large (30+ forms)?

**Step 3**: Reference the example plan
- Small: See "Step 3: Example Plans" → "10 forms"
- Medium: See the "20-Form Project" scenario
- Large: See the "50+ Forms" scenario

**Step 4**: Create your conversation schedule
- Use the formula: `1 + ceil(N/5) + 1`
- Plan form batches: 5-6 forms per Phase 3 conversation

---

### I'm Mid-Analysis and Need to Monitor

**During Execution**: Keep these open:
1. `TOKEN-CONSUMPTION-NUMBERS.md` → "When to Start New Conversation"
2. `TOKEN-CONSUMPTION-GUIDE.md` → "Monitoring Token Usage"

**Watch for**:
- Response times (should be 5-10 seconds)
- Complete output (all 6 sections present)
- Forms analyzed so far (switch after 5-6 in Phase 3)

**If in doubt**: Start a new conversation
- Saves more than it costs
- Progress saved in memory-bank/progress.md

---

### I Need to Understand Token Accounting

**Deep Technical**: Start with `TOOL-OVERHEAD-AND-CONTEXT-MATH.md`
- Learn token conversion factors
- Understand tool overhead costs
- See context window math in detail

**Then read**: `TOKEN-CONSUMPTION-DETAILED-ANALYSIS.md`
- See how all components combine
- Review adjustment guide for your context
- Understand optimization techniques

**Finally**: `TOKEN-CONSUMPTION-GUIDE.md` (Sections 8-10)
- Cost optimization tips
- Adjustment guidelines
- When/how to apply each optimization

---

### I'm Optimizing for Cost or Speed

**Reference**: `TOKEN-CONSUMPTION-GUIDE.md` Section 8
- Token efficiency ranking
- Skip Mermaid diagrams: -10-15% tokens
- Use summary mode: -15-20% tokens
- Cache file reloading: -15% tokens (dev-side)
- Archive old forms: -5% tokens

**Read**: `TOOL-OVERHEAD-AND-CONTEXT-MATH.md` Section 5
- Specific optimization techniques
- Combined optimization example
- Expected time savings

---

### I Need to Explain This to My Team

**Executive Summary**: `ANALYSIS-SUMMARY.md`
- Key findings and recommendations
- Conversation plans A/B/C
- Why 128K token limit matters

**For Stakeholders**: `TOKEN-CONSUMPTION-NUMBERS.md`
- Simple tables and formulas
- Real project examples (5/20/50 forms)
- Conversation count estimates

**For Developers**: `TOKEN-CONSUMPTION-DETAILED-ANALYSIS.md`
- Complete technical breakdown
- Adjustment guide
- Implementation notes

---

## Document Relationships

```
README (this file)
├─ Entry point for all users
│
├─ QUICK DECISIONS BRANCH
│  ├─ TOKEN-CONSUMPTION-NUMBERS.md
│  └─ QUICK-REFERENCE.md
│
├─ PLANNING BRANCH
│  ├─ TOKEN-CONSUMPTION-GUIDE.md
│  └─ ANALYSIS-SUMMARY.md
│
├─ TECHNICAL BRANCH
│  ├─ TOKEN-CONSUMPTION-DETAILED-ANALYSIS.md
│  └─ TOOL-OVERHEAD-AND-CONTEXT-MATH.md
│
└─ RUNTIME REFERENCE BRANCH
   ├─ usage-guide.md (Section 4)
   └─ memory-bank/progress.md (track during execution)
```

---

## Key Numbers (Copy & Paste Reference)

### Phase Tokens (Typical Scenarios)

| Phase | Scenario | Tokens |
|-------|----------|--------|
| 1a | Project-scan (100 files) | 3,500 |
| 1b | Protocol-detect (3-5 protocols) | 4,000 |
| 2 | Form-inventory (20 forms) | 5,000 |
| **1+2** | **Setup (20 forms)** | **12,500** |
| 3 | Per form (medium) | 10,000 |
| 4 | 20 forms reported | 56,000 |

### Forms Per Conversation (Phase 3)

| Strategy | Forms | Buffer | Risk |
|----------|-------|--------|------|
| Conservative | 5 | 50% | Low ✅ |
| Balanced | 6 | 40% | Low ✅ |
| Aggressive | 7 | 31% | Medium ⚠️ |

### Total Conversations Needed

```
5 forms:   1 + ceil(5/5) + 1 = 3 conversations
10 forms:  1 + ceil(10/5) + 1 = 4 conversations
20 forms:  1 + ceil(20/5) + 1 = 6 conversations
50 forms:  1 + ceil(50/5) + 1 = 12 conversations
```

### Context Window (128K)

- Total: 128,000 tokens
- Safety margin: 20% (26,214 tokens)
- Usable: ~104,858 tokens per conversation

---

## What Each Document Covers

### TOKEN-CONSUMPTION-NUMBERS.md
```
✅ Phase-by-phase tokens (all scenarios)
✅ Forms per conversation limits
✅ Project examples with conversation breakdown
✅ Quick decision tree
✅ When to switch conversations checklist
✅ Formulas for quick calculation
❌ Technical details (see other documents)
❌ Tool overhead breakdown (see TOOL-OVERHEAD)
```

### TOKEN-CONSUMPTION-GUIDE.md
```
✅ Complete token accounting method
✅ All 5 phases with detailed breakdowns
✅ Conversation capacity analysis
✅ Example plans for 10/25/50 form projects
✅ Cost optimization tips
✅ Token efficiency ranking
❌ Does NOT include adjustment factors (see DETAILED-ANALYSIS)
❌ Does NOT include tool overhead math (see TOOL-OVERHEAD)
```

### TOKEN-CONSUMPTION-DETAILED-ANALYSIS.md
```
✅ Complete technical breakdown (12,000+ words)
✅ Tool overhead accounting
✅ Real project measurements
✅ Adjustment guide (forms larger/smaller, deep chains, etc.)
✅ Optimization techniques with examples
✅ Practical workflow checklist
✅ Troubleshooting guide
❌ Not meant for quick reference (use NUMBERS for that)
```

### TOOL-OVERHEAD-AND-CONTEXT-MATH.md
```
✅ Token conversion factors (English vs. Chinese)
✅ Tool invocation overhead (glob, grep, read, write)
✅ Context window math and headroom
✅ Practical headroom examples
✅ Token cost breakdown per operation
✅ Monitoring techniques
✅ Troubleshooting token issues
❌ Not a planning document (use GUIDE for planning)
```

---

## Document Sizes & Reading Times

| Document | Type | Lines | Read Time | Best For |
|----------|------|-------|-----------|----------|
| TOKEN-CONSUMPTION-NUMBERS.md | Quick Ref | 400 | 5-10 min | Decisions |
| QUICK-REFERENCE.md | Quick Ref | 250 | 5-10 min | Decisions |
| TOKEN-CONSUMPTION-GUIDE.md | Reference | 445 | 20-30 min | Planning |
| ANALYSIS-SUMMARY.md | Summary | 500 | 30-45 min | Understanding |
| TOKEN-CONSUMPTION-DETAILED-ANALYSIS.md | Technical | 800+ | 45-60 min | Deep dive |
| TOOL-OVERHEAD-AND-CONTEXT-MATH.md | Technical | 600+ | 30-45 min | Technical |
| usage-guide.md (Section 4) | Reference | 100 | 5 min | Runtime |
| This file | Index | 400 | 10 min | Navigation |

---

## Recommended Reading Order by Role

### User (Just Want to Analyze)
1. `TOKEN-CONSUMPTION-NUMBERS.md` (5 min)
2. Keep it open during analysis
3. Reference "red flags" section

**Total prep**: 5 minutes

---

### Project Manager (Planning the Analysis)
1. `ANALYSIS-SUMMARY.md` (30 min) - Executive overview
2. Count forms and choose Plan A/B/C
3. Schedule conversations based on plan
4. Reference `TOKEN-CONSUMPTION-NUMBERS.md` during execution

**Total prep**: 35 minutes

---

### Technical Lead (Implementing/Optimizing)
1. `TOKEN-CONSUMPTION-DETAILED-ANALYSIS.md` (45 min)
2. `TOOL-OVERHEAD-AND-CONTEXT-MATH.md` (40 min)
3. Review optimization opportunities
4. Plan caching/batching improvements

**Total prep**: 90 minutes

---

### Developer (Modifying the Analyzer Tool)
1. `TOOL-OVERHEAD-AND-CONTEXT-MATH.md` (40 min) - Understand costs
2. `TOKEN-CONSUMPTION-DETAILED-ANALYSIS.md` (45 min) - See bottlenecks
3. `TOKEN-CONSUMPTION-GUIDE.md` Section 8 (10 min) - Review optimizations
4. Design improvements based on findings

**Total prep**: 100 minutes

---

## Integration with existing Documentation

These token analysis documents complement:
- `usage-guide.md` — Updated Section 4 with token budget
- `memory-bank/progress.md` — Tracks execution state
- `.clinerules` — Defines skills and memory bank structure
- `skills/*/SKILL.md` — Execution procedures

---

## FAQ: Which Document Should I Read?

**Q: I just want to know how many conversations I need**
→ `TOKEN-CONSUMPTION-NUMBERS.md` (5 min)

**Q: I want to plan before starting**
→ `TOKEN-CONSUMPTION-GUIDE.md` (20-30 min)

**Q: I need to understand WHY the numbers are what they are**
→ `TOKEN-CONSUMPTION-DETAILED-ANALYSIS.md` (45-60 min)

**Q: I want to optimize the tool itself**
→ `TOOL-OVERHEAD-AND-CONTEXT-MATH.md` (40 min)

**Q: I need to explain this to my boss**
→ `ANALYSIS-SUMMARY.md` (30-45 min)

**Q: I'm mid-analysis and things seem slow**
→ `TOKEN-CONSUMPTION-NUMBERS.md` Section "When to Start New Conversation" (2 min)

**Q: I want to adjust estimates for my forms**
→ `TOKEN-CONSUMPTION-DETAILED-ANALYSIS.md` Section "Adjustment Guide" (10 min)

---

## Quick Validation Checklist

Before starting your analysis:

```
[ ] Read TOKEN-CONSUMPTION-NUMBERS.md (5 minutes)
[ ] Counted total forms in project
[ ] Calculated conversations = 1 + ceil(forms/5) + 1
[ ] Decided on forms per conversation (5, 6, or 7)
[ ] Noted token limits for my scenario
[ ] Bookmarked "red flags" section
[ ] Ready to start Phase 1a
```

---

## Support & Questions

**For token estimate questions**: See `TOKEN-CONSUMPTION-DETAILED-ANALYSIS.md` → "Adjustment Guide"

**For conversation planning questions**: See `TOKEN-CONSUMPTION-GUIDE.md` → "Conversation Planning Guide"

**For runtime questions**: See `TOKEN-CONSUMPTION-NUMBERS.md` → "When to Start New Conversation"

**For technical questions**: See `TOOL-OVERHEAD-AND-CONTEXT-MATH.md`

---

## Version & Status

| Document | Version | Status | Last Updated |
|----------|---------|--------|--------------|
| README-TOKEN-ANALYSIS.md | 1.0 | Complete | 2026-03-11 |
| TOKEN-CONSUMPTION-NUMBERS.md | 1.0 | Complete | 2026-03-11 |
| TOKEN-CONSUMPTION-DETAILED-ANALYSIS.md | 1.0 | Complete | 2026-03-11 |
| TOOL-OVERHEAD-AND-CONTEXT-MATH.md | 1.0 | Complete | 2026-03-11 |
| TOKEN-CONSUMPTION-GUIDE.md | 1.0 | Complete | 2026-03-11 |
| ANALYSIS-SUMMARY.md | 1.0 | Complete | 2026-03-11 |
| QUICK-REFERENCE.md | 1.0 | Complete | 2026-03-11 |
| usage-guide.md | 2.1 | Updated | 2026-03-11 |

---

## Summary

You now have **8 documents** covering token consumption from every angle:

- **2 Quick Reference**: 5-10 minute reads for decisions
- **3 Detailed Guides**: 20-45 minute reads for planning
- **2 Technical References**: 30-60 minute reads for deep understanding
- **1 Navigation Guide**: This file

Pick the document that matches your need, read it, and you'll know exactly how many conversations you need for your project.

**Recommended**: Start with `TOKEN-CONSUMPTION-NUMBERS.md`. If you want more detail, use this index to find the right document.

---

**Document Created**: 2026-03-11
**Purpose**: Navigation guide for token consumption analysis
**Status**: Complete and Ready to Use
