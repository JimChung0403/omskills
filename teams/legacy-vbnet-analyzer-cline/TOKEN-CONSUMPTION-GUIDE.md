# Token Consumption Reference Guide

## Overview

This guide provides detailed token consumption estimates for the VB.NET Legacy Analyzer workflow. These estimates help you plan conversation splits for projects of varying sizes.

**Context Window**: 128K tokens (~131,072 tokens)
**Recommended Buffer**: 20% (~26,214 tokens)
**Usable Capacity**: ~104,858 tokens per conversation
**Model**: Claude Haiku 4.5 (fast, cost-effective)

---

## Token Accounting Method

### Estimation Basis

- **English text**: 1 token ≈ 4 characters
- **Chinese Traditional (繁體)**: 1 token ≈ 1.5 characters (CJK is denser)
- **Tool overhead per call**: ~200 tokens (request serialization + response wrapper)
  - Glob operations: ~150 tokens
  - Grep operations: ~200 tokens per pattern
  - File read: ~100 tokens base + content
  - File write: ~100 tokens base + content

### Fixed Overhead (every conversation)

| Component | Tokens |
|-----------|--------|
| Claude system context | ~500 |
| Session initialization | ~150 |
| progress.md (initial load) | ~100 |
| **Subtotal** | **~750** |

### Per-Phase Skill Loading

| Skill File | Lines | Estimated Tokens |
|-----------|-------|------------------|
| project-scan/SKILL.md | 91 | ~80 |
| protocol-detection/SKILL.md | 99 | ~86 |
| form-inventory/SKILL.md | 98 | ~85 |
| form-analysis/SKILL.md | 216 | ~189 |
| report-generation/SKILL.md | 103 | ~90 |

---

## Phase 1a: Project-Scan

### Inputs
- `progress.md` (32 lines): ~100 tokens
- Skill: ~80 tokens
- Tool overhead: ~600 tokens

### File Operations

| Operation | Low | Typical | High |
|-----------|-----|---------|------|
| Glob scan (file list) | 50 files | 100 files | 300+ files |
| Glob result tokens | 150 | 300 | 600 |
| Read .vbproj (100-300 lines) | 150 | 200 | 400 |
| Read .sln (30-50 lines) | 75 | 100 | 150 |
| Grep classification (10-50 matches) | 200 | 300 | 600 |

### Output Generation

| Document | Lines (Low/Typ/High) | Tokens |
|----------|---------------------|--------|
| 01-project-structure.md | 50/75/100 | 1,000/1,500/1,800 |

### Phase 1a Totals

| Scenario | Small Project | Medium Project | Complex Project |
|----------|---------------|----------------|-----------------|
| **Low** | 2,000 | — | — |
| **Typical** | — | 3,500 | — |
| **High** | — | — | 6,000 |

---

## Phase 1b: Protocol-Detection

### Inputs
- `progress.md` + `01-project-structure.md` (reread): ~300 tokens
- Skill: ~86 tokens
- Tool overhead: ~1,000 tokens (20 grep patterns)

### File Operations

| Operation | Tokens |
|-----------|--------|
| Grep searches (20 patterns × 5 matches avg) | ~625 |
| Context reading (5 lines per match) | ~400 |
| Config file reads (App.config, etc.) | ~150 |

### Output Generation

| Document | Tokens |
|----------|--------|
| 02-protocols.md (50-100 lines + Mermaid) | 1,667 |

### Phase 1b Totals

| Scenario | Simple | Typical | Complex |
|----------|--------|---------|---------|
| Token consumption | 2,500 | 4,000 | 6,500 |
| **Example** | 1 DB | 3-5 protocols | EF + WCF + API |

---

## Phase 2: Form-Inventory

### Inputs
- `progress.md` + `01-project-structure.md` (reread): ~300 tokens
- Skill: ~85 tokens
- Tool overhead: ~800 tokens

### File Operations (varies by Form count)

**Per Form:**
- Read `.Designer.vb` (200 lines avg): ~175 tokens
- Grep form relationships: ~50 tokens
- Total per form: ~225 tokens

**All Forms:**
- Grep searches (New, Show, ShowDialog): ~125 tokens

### Output Generation

| Document | Tokens |
|----------|--------|
| 03-form-inventory.md (~150 lines + Mermaid) | ~1,200 |

### Phase 2 Totals (by Form count)

| Form Count | Token Consumption |
|-----------|-------------------|
| 5 | ~2,400 |
| 10 | ~3,100 |
| 20 | ~5,000 |
| 50 | ~10,250 |

**Formula**: ~1,500 base + (N_forms × 225)

---

## Phase 3: Form-Analysis (per Form)

### Inputs (reread for each form)
- `progress.md` + `01-proj` + `02-proto` + `03-inventory`: ~800 tokens
- Skill: ~189 tokens
- Tool overhead: ~1,500 tokens (5-15 file reads, 20+ grep searches)

### Per-Form File Reading

| File Type | Low (lines) | Typical | High |
|-----------|-------------|---------|------|
| {FormName}.vb | 200 | 600 | 1500 |
| {FormName}.Designer.vb | 100 | 250 | 500 |
| Tokens for above | 75 | 250 | 750 |

### Cross-File Tracking

- Related files: 3-10 (average 5)
- Tokens per file: ~300 (avg 300 lines)
- Subtotal: **~1,500 tokens**

### Grep Operations

- Part A-E searches: ~30 patterns
- Context snippets: ~375 tokens

### Output Generation

| Form Complexity | Lines | Tokens |
|-----------------|-------|--------|
| Simple (few fields) | 200 | ~2,000 |
| Medium (20-40 fields) | 300-350 | ~2,500-2,800 |
| Complex (50+ fields) | 400-500 | ~3,000-3,500 |
| Very Complex (deep logic) | 500+ | ~3,500+ |

### Phase 3 Totals (per Form)

| Form Type | Token Consumption | Formula |
|-----------|-------------------|---------|
| Simple | ~6,000 | 800 + 189 + 1,500 + 2,511 + 2,000 |
| Medium | ~10,000 | base + complex form output |
| Complex | ~15,000 | base + complex operations |
| Very Complex | ~18,000 | deep call chains (10+ files) |

**Average per Form**: ~10,000 tokens (medium complexity)

---

## Phase 4: Report-Generation

### Inputs (load all memory-bank)
- `01-project-structure.md`: ~300 tokens
- `02-protocols.md`: ~300 tokens
- `03-form-inventory.md`: ~300 tokens
- Summarized form data: ~500 tokens per form
- Skill: ~90 tokens

### Output Generation (N = forms analyzed)

| Document | Tokens |
|----------|--------|
| 00-system-architecture.md (~200 lines + Mermaid) | ~1,500 |
| 01-form-{name}.md per form (~300-400 lines + Mermaid) | ~2,000 × N |
| 99-business-logic-index.md (~200 lines) | ~1,500 |

### Phase 4 Totals

| Forms Analyzed | Token Consumption |
|----------------|-------------------|
| 5 | ~18,500 |
| 10 | ~31,000 |
| 20 | ~56,000 |
| 30 | ~81,000 |
| 50 | ~131,000 ⚠️ LIMIT |

**Formula**: ~6,000 base + (N × 2,500)

---

## Single Conversation Capacity Analysis

### Available Capacity

```
Total window:           128K tokens = 131,072 tokens
Safety margin (20%):                   -26,214 tokens
Usable capacity:                      ~104,858 tokens
```

### Scenario: 20-Form Project

| Phase | Tokens |
|-------|--------|
| 1a | 3,500 |
| 1b | 4,000 |
| 2 | 5,000 |
| **Subtotal** | **12,500** ← Conv 1 |
| **Remaining** | **~92,358** |
| Phase 3 × 5 forms | 50,000 | ← Conv 2
| Phase 3 × 5 forms | 50,000 | ← Conv 3
| Phase 3 × 5 forms | 50,000 | ← Conv 4
| Phase 3 × 5 forms | 50,000 | ← Conv 5
| **Phase 4** | 56,000 | ← Conv 6 |

### Capacity by Project Size

#### Small Projects (< 10 Forms)

```
Conv 1:  Phase 1a + 1b + 2        = ~12,500 tokens ✅
         Remaining:                = ~92,000 tokens

Conv 2:  Phase 3 (all 10 forms)   = ~100,000 tokens ⚠️ RISKY
         → Split into Conv 2 + Conv 3

Recommendation: 2-3 conversations total
```

#### Medium Projects (10-30 Forms)

```
Conv 1:  Phase 1a + 1b + 2              = ~12,500 tokens ✅
Conv 2:  Phase 3 (forms 1-5)            = ~50,000 tokens ✅
Conv 3:  Phase 3 (forms 6-10)           = ~50,000 tokens ✅
Conv 4:  Phase 3 (forms 11-15)          = ~50,000 tokens ✅
Conv 5:  Phase 3 (forms 16-20)          = ~50,000 tokens ✅
...continue for 21-30...
Conv N:  Phase 4 (all forms)            = ~56,000-81,000 tokens ✅

Recommendation: Phase 1 in 1 conv, then 1 conv per 5 forms (Phase 3)
Total: 1 + ceil(N/5) + 1 conversations
```

#### Large Projects (30+ Forms)

```
Conv 1:  Phase 1a + 1b + 2              = ~12,500 tokens ✅
Conv 2+: Phase 3 in batches of 4-6 forms per conv ✅
Final:   Phase 4 (separate)              = 81,000+ tokens ✅

Recommendation: 1 + ceil(N/5) + 1 conversations
```

---

## Forms Per Conversation Limits

### Phase 3 Analysis Only (separate conversations)

```
Available for analysis:  ~104,858 tokens
Overhead per phase:      ~2,500 tokens (reload files + skill)
Available for forms:     ~102,358 tokens
Per form:               ~10,000 tokens

Capacity = floor(102,358 / 10,000) = 10 forms THEORETICAL

BUT with safety margin:
Conservative (30% buffer): 7 forms per conversation ✅
Recommended (50% buffer):  5 forms per conversation ✅✅
Minimum (20% buffer):      8 forms per conversation ⚠️
```

### Token Headroom by Strategy

| Strategy | Forms/Conv | Safety | Headroom | Risk Level |
|----------|-----------|--------|----------|-----------|
| Conservative | 5 | 50% | 52K tokens | Very Low ✅ |
| Balanced | 6-7 | 30-40% | 31-52K tokens | Low ✅ |
| Aggressive | 8 | 20% | 20K tokens | Medium ⚠️ |
| Risky | 10+ | <10% | <10K tokens | High ❌ |

---

## Conversation Planning Guide

### Step 1: Calculate Total Forms in Your Project
```
Total forms = count all {FormName}.vb files matching "Inherits Form"
```

### Step 2: Plan Conversation Sequence

**Formula for Phase 3 conversations**:
```
N_phase3_convs = ceil(total_forms / 5)  // conservative
N_phase3_convs = ceil(total_forms / 6)  // balanced
```

**Total conversations**:
```
total_convs = 1 (Phase 1a+1b+2) + N_phase3_convs + 1 (Phase 4)
```

### Step 3: Example Plans

**10 forms**:
```
Conv 1: Phase 1a + 1b + 2
Conv 2: Phase 3 (forms 1-5)
Conv 3: Phase 3 (forms 6-10)
Conv 4: Phase 4 (reporting)
Total: 4 conversations
```

**25 forms**:
```
Conv 1: Phase 1a + 1b + 2
Conv 2: Phase 3 (forms 1-5)
Conv 3: Phase 3 (forms 6-10)
Conv 4: Phase 3 (forms 11-15)
Conv 5: Phase 3 (forms 16-20)
Conv 6: Phase 3 (forms 21-25)
Conv 7: Phase 4 (reporting)
Total: 7 conversations
```

**50 forms**:
```
Conv 1: Phase 1a + 1b + 2
Conv 2-11: Phase 3 (5 forms each)
Conv 12: Phase 4 (reporting)
Total: 12 conversations
```

---

## Monitoring Token Usage in Practice

### Red Flags (Switch to New Conversation)

1. **Output truncation**: Cline says "output was truncated" or "message incomplete"
2. **Missing content**: Expected analysis sections are skipped or abbreviated
3. **Slow response**: API responses take > 30 seconds consistently
4. **Form count reached**: Phase 3 has analyzed 5-7 forms already
5. **Error messages**: "context_length_exceeded" or similar warnings

### Green Flags (Safe to Continue)

1. ✅ Cline responding normally (5-10 second response times)
2. ✅ Full form analysis output (6 sections + Mermaid diagrams)
3. ✅ Fewer than 5 forms analyzed so far in Phase 3
4. ✅ No warnings or truncation messages

---

## Cost Optimization Tips

### Token Efficiency Ranking

| Technique | Token Savings | Difficulty |
|-----------|---------------|-----------|
| Batch Phase 3 by 5-6 forms | 20-30% | Easy |
| Skip detailed Mermaid diagrams | 10% | Medium |
| Use summary mode (skip line-by-line traces) | 15-20% | Hard |
| Combine Phase 4 with last Phase 3 | 10% | Easy |

### Recommended Optimizations

**For all projects:**
- Plan form count before starting
- Use recommended conversation splits (5 forms/conv Phase 3)
- Batch Phase 4 reporting into 1 conversation

**For large projects (50+ forms):**
- Consider analyzing only critical forms (Phase 3)
- Generate summarized reports instead of full analysis
- Use external merge tool to combine final reports

---

## Quick Reference Table

| Metric | Value | Notes |
|--------|-------|-------|
| Context window | 128K | Safe buffer: 20% |
| Usable capacity | ~105K | After margin |
| Phase 1a+1b+2 | ~12.5K | Bundle in 1 conv |
| Phase 3 per form | ~10K | Medium complexity |
| Phase 3 per conv | 5-7 forms | Recommended batch |
| Phase 4 base | ~6K | +2.5K per form |
| Forms in Phase 4 | 20 | ~56K tokens |
| Forms in Phase 4 | 30 | ~81K tokens |
| Conversation count (20 forms) | 6 | Conv 1 + 4×Phase3 + Conv6 |
| Total project capacity | 50+ forms | With proper splits |

---

## Updates and Adjustments

These estimates are based on:
- Haiku 4.5 model tokenization
- Average VB.NET project structure
- Typical form complexity distribution

**Adjust estimates if**:
- Your forms are significantly larger (2000+ lines): +50% tokens
- Your forms are minimal (< 200 lines): -30% tokens
- Heavy Mermaid usage: +10-15% tokens
- Minimal diagrams: -10-15% tokens
