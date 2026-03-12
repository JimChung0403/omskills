# VB.NET Analyzer: Quick Reference Card

## Token Budget Summary

**Context Window**: 128K tokens | **Usable**: ~105K tokens | **Safety Margin**: 20%

### Per-Phase Token Consumption

| Phase | Typical | Range | Notes |
|-------|---------|-------|-------|
| **1a: Project-Scan** | 3.5K | 2K-6K | Once per project |
| **1b: Protocol-Detection** | 4K | 2.5K-6.5K | Once per project |
| **2: Form-Inventory** | 5K (20F) | 2.4K-10K | Once per project |
| **3: Form-Analysis** | 10K/form | 6K-18K | Per form (medium) |
| **4: Report-Generation** | 2.5K/form | Base 6K | Per form included |

### Conversation Planning Formula

```
Total conversations = 1 + ceil(N_forms / 5) + 1
                    = Setup + Analysis batches + Reporting
```

**Examples**:
- 5 forms → 1 + 1 + 1 = **3 conversations**
- 10 forms → 1 + 2 + 1 = **4 conversations**
- 20 forms → 1 + 4 + 1 = **6 conversations** ← typical
- 30 forms → 1 + 6 + 1 = **8 conversations**
- 50 forms → 1 + 10 + 1 = **12 conversations**

### Forms Per Conversation Limits (Phase 3)

| Strategy | Forms/Conv | Safety | Recommendation |
|----------|-----------|--------|-----------------|
| **Conservative** | 5 | 50% | ✅ **USE THIS** |
| Balanced | 6-7 | 30-40% | ✅ If monitoring |
| Aggressive | 8 | 20% | ⚠️ Risky |
| Unsafe | 9+ | <10% | ❌ Don't do this |

---

## Conversation Plans by Project Size

### Small (< 10 Forms)

```
Conv 1: Phase 1a + 1b + 2           = 9.9K tokens  ✅
Conv 2: Phase 3 (forms 1-5)         = 50K tokens   ✅
Conv 3: Phase 3 (forms 6-10)        = 50K tokens   ✅
Conv 4: Phase 4 (reporting)         = 31K tokens   ✅
Total: 4 conversations
```

### Medium (10-30 Forms) ← Most Common

```
Conv 1: Phase 1a + 1b + 2           = 12.5K tokens ✅
Conv 2: Phase 3 (forms 1-5)         = 50K tokens   ✅
Conv 3: Phase 3 (forms 6-10)        = 50K tokens   ✅
Conv 4: Phase 3 (forms 11-15)       = 50K tokens   ✅
Conv 5: Phase 3 (forms 16-20)       = 50K tokens   ✅
Conv 6: Phase 4 (all forms)         = 56K tokens   ✅
Total: 6 conversations (for 20 forms)
```

### Large (30+ Forms)

```
Conv 1: Phase 1a + 1b + 2           = 12.5K ✅
Conv 2-N: Phase 3 (5 forms each)    = 50K ✅ (repeat N times)
Conv Final: Phase 4 (all forms)     = ~81K for 30F ✅
Total: 1 + ceil(N/5) + 1 conversations
```

---

## Quick Decision Tree

```
Question 1: How many forms in your VB.NET project?
├─ Count all {FormName}.vb files that "Inherits Form"

Question 2: Pick your project size
├─ < 10 forms      → Use Plan A (3-4 conversations)
├─ 10-30 forms     → Use Plan B (6-8 conversations) ← MOST COMMON
├─ 30-50 forms     → Use Plan C (10-12 conversations)
└─ > 50 forms      → Use Plan C, consider subset

Question 3: Plan your conversation batches
├─ Conv 1: Always Phase 1a + 1b + 2 together
├─ Conv 2-N: Phase 3 with 5 forms per conversation
└─ Conv Final: Phase 4 with all analyzed forms
```

---

## When to Switch Conversations

### Red Flags ❌ (Immediate Action)
- Cline says "output truncated" or "message incomplete"
- Missing form analysis sections (skipped fields, buttons)
- API responds with "context_length_exceeded"
- Response time > 30 seconds consistently
- You've analyzed 5+ forms in Phase 3 already

### Safe to Continue ✅ (Monitor)
- Normal response time (5-10 seconds)
- Complete form analysis output (6 sections + diagrams)
- Fewer than 5 forms analyzed in current Phase 3 batch
- No warnings or error messages

---

## Phase Breakdown Checklist

### Phase 1a: Project-Scan
- [ ] Glob scan for .vb, .vbproj, .sln, .config, .resx, .xsd files
- [ ] Classify files (Form, Module, Class, DataSet)
- [ ] Identify entry point (Sub Main, Application.Run)
- [ ] Extract references and dependencies
- [ ] Output: `memory-bank/01-project-structure.md`
- [ ] Tokens: ~3.5K

### Phase 1b: Protocol-Detection
- [ ] Search for DB connection types (ADO.NET, Entity Framework, etc.)
- [ ] Identify external communication (HTTP, WCF, Web Service, Socket)
- [ ] Find connection strings (config files, hardcoded)
- [ ] Detect serialization formats (XML, JSON, binary)
- [ ] Output: `memory-bank/02-protocols.md` + Mermaid system diagram
- [ ] Tokens: ~4K

### Phase 2: Form-Inventory
- [ ] Read all Form Designer.vb files
- [ ] Classify controls (input, display, action, container, other)
- [ ] Analyze Form call relationships (New, Show, ShowDialog)
- [ ] Identify shared UserControls
- [ ] Output: `memory-bank/03-form-inventory.md` + Mermaid call graph
- [ ] Tokens: ~5K (20 forms)

### Phase 3: Form-Analysis (Per Form)
- [ ] Read Form.vb and Form.Designer.vb
- [ ] Trace field data sources (A1-A3)
- [ ] Analyze button logic (B1-B4)
- [ ] Map event flow (C1-C3)
- [ ] Trace cross-file calls (D1-D4)
- [ ] Generate Mermaid diagrams (E1-E2)
- [ ] Output: `memory-bank/forms/{FormName}.md`
- [ ] Tokens: ~10K per form (medium)
- [ ] **Batch**: 5 forms per conversation

### Phase 4: Report-Generation
- [ ] Consolidate system architecture
- [ ] Format form reports (01-form-{name}.md)
- [ ] Create business logic index (99-business-logic-index.md)
- [ ] Output: `analysis-reports/`
- [ ] Tokens: ~2.5K per form + 6K base

---

## Token Accounting Quick Math

### How to estimate your project

```
Step 1: Count your forms
  N_forms = number of {FormName}.vb with "Inherits Form"

Step 2: Estimate Phase 3 tokens
  Phase_3_tokens = N_forms × 10,000
  (Use 6K for simple, 15K for complex)

Step 3: Estimate Phase 4 tokens
  Phase_4_tokens = 6,000 + (N_forms × 2,500)

Step 4: Calculate conversations needed
  conversations_phase3 = ceil(Phase_3_tokens / 100,000)
  conversations_total = 1 + conversations_phase3 + 1

Example (20 forms):
  Phase_3 = 20 × 10K = 200K tokens
  Phase_3_convs = ceil(200K / 100K) = 2 convs → NO! wrong calculation

  Correct: Use 5 forms per conv
  Phase_3_convs = ceil(20 / 5) = 4 convs
  Total = 1 + 4 + 1 = 6 conversations
```

### Form Complexity Estimation

| Type | Lines | Tokens | Example |
|------|-------|--------|---------|
| **Simple** | 200-300 | 6,000 | Lookup form, read-only display |
| **Medium** | 600-1000 | 10,000 | Data entry form with 5-10 buttons |
| **Complex** | 1500-2000 | 15,000 | Multi-tab form with business logic |
| **Very Complex** | 2000+ | 18,000+ | Form with 10+ related files |

---

## Important Notes

### Before You Start
- [ ] Install tool files to project root
- [ ] Load `.clinerules` in Cline
- [ ] Initialize `memory-bank/progress.md`
- [ ] Count total forms in project
- [ ] Choose conversation plan (A/B/C)

### During Phase 1a+1b+2
- All 3 phases run in **1 conversation** (12.5K tokens)
- No need to split
- Creates foundation for later phases

### During Phase 3 (Per-Form Analysis)
- **Open new conversation after 5 forms**
- Say "繼續" (continue) in new conversation
- Tool auto-reads `progress.md` and resumes
- Keep safety margin (30% buffer)

### Before Phase 4
- Ensure all forms in Phase 3 are analyzed
- Phase 4 reads all form files
- Can run in **1 conversation** if ≤ 20 forms
- Larger projects may need Phase 4 split

---

## Files Reference

| Document | Purpose | Audience |
|----------|---------|----------|
| **usage-guide.md** | Quick start + decision table | End users |
| **TOKEN-CONSUMPTION-GUIDE.md** | Detailed reference | Planners, developers |
| **ANALYSIS-SUMMARY.md** | Full technical analysis | Project managers |
| **QUICK-REFERENCE.md** | This card | Everyone |

---

## Support Resources

### If output is truncated:
→ Stop immediately, save progress (already saved to progress.md)
→ Open new conversation
→ Say "繼續" (continue)

### If uncertain about form count:
→ Use: `find . -name "*.vb" -exec grep -l "Inherits.*Form" {} \;`

### If analysis seems stuck:
→ Check `memory-bank/progress.md` status
→ Manually update status if needed
→ Use "重新分析 {FormName}" to redo a form

---

**Last Updated**: 2026-03-11
**Model**: Claude Haiku 4.5
**Context Window**: 128K tokens
