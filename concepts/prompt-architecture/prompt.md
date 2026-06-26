# Prompt architecture audit — agent procedure

**Role:** Architecture / quality reviewer (human or tool).  
**Use:** Before adding a new LLM feature, after observing hallucinations, or when standardizing prompt patterns across a codebase.  
**Evidence policy:** Every structural claim must cite a file path. Every "risk" must be tagged `observed` | `inferred` | `potential`.

## Inputs (required)

- All files that construct system prompts or call LLM APIs.
- The environment/domain configuration for prompt behaviour differences.
- Test commands for the LLM routes (or `unknown`).

## Procedure

1. **Inventory prompt construction sites** — list every file that builds or sends a system prompt. Count inline vs composed prompts.
2. **Score prompt structure** — for each site:
   - `inline` — hardcoded in the controller/route
   - `layered` — uses some separation (persona vs instructions vs context)
   - `registered` — uses a registry/composer pattern with priority ordering
3. **Check rule isolation** — are safety rules (anti-hallucination, confidentiality) in the same block as persona instructions? If yes, flag as risk.
4. **Check recency bias** — do the most important rules appear at the end of the prompt?
5. **Check stage separation** — does a single LLM call handle both fact extraction and generation? If yes and hallucinations are observed, recommend multi-stage pipeline.
6. **Check post-generation validation** — does any code verify output claims against source context?
7. **Check environment scoping** — can different environments (legal, medical, general) activate different rule sets?
8. **Prompt architecture score** — assign `none` | `basic` | `structured` | `comprehensive`:
   - `none`: all prompts are inline
   - `basic`: some shared prompt utilities exist
   - `structured`: registry + composition with priorities
   - `comprehensive`: multi-stage pipeline + validation + environment scoping

## Output (required sections)

```markdown
## Prompt architecture audit
- Prompt construction sites: <n> — <file paths>
- Structure score: none | basic | structured | comprehensive
- Rule isolation: yes | partial | no — <evidence>
- Recency bias utilized: yes | no
- Stage separation: single | multi
- Post-generation validation: yes | no

## Recommendations
- <ordered list of structural improvements with file paths>
- <risk items tagged observed/inferred/potential>

## Priority
- Critical: inline prompts with safety rules are <list>
- High: no rule registry
- Medium: no environment scoping
- Low: no post-generation validation
```

## Stop / escalate when

- Any LLM feature has **no** prompt structure review and handles sensitive data — **do not approve** for production.
- Prompt changes touch safety/confidentiality rules — require explicit human review.
