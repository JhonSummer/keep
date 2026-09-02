# Service Scenario Scout

Source: pasted ChatGPT note, 2026-09-03
Exported: 2026-09-03 02:10 +07

---

Yes. And I think I can pinpoint what went wrong in the previous attempt.

Your high-level instruction was probably clear enough about the business goal. What Codex lacked was a reliable coverage procedure. So it fell back to an easy computational interpretation—SQL summaries, counts, obvious categories—while you and your boss ended up performing the hard part yourselves: reading the language, noticing distinctions, deciding which tickets were actually about services, and spotting scenarios that didn’t fit the obvious buckets.

That doesn’t mean the agentic workflow is doomed. It means “analyze this sheet and identify the scenarios” needs to be turned into a repeatable analyst skill rather than left as an open-ended prompt.

OpenAI’s current Codex design is actually well suited to this. Codex now has first-class Skills specifically for repeatable multi-step workflows, tool usage, domain procedures, and reliability constraints. Repository skills live under `.agents/skills`, can include scripts and references, and can be triggered explicitly with `$skill-name`. Codex also now supports specialized subagents for things like exploration and independent review, specifically to keep large exploratory work from polluting the main agent’s context.

## What I would make Codex do tomorrow

The fundamental workflow should be:

```
                    CLIENT WORKBOOK
                         │
                         ▼
              Structural inspection
             sheets / columns / nulls
                         │
                         ▼
             Build semantic row text
       title + description + story + AC + etc.
                         │
                         ▼
              SERVICE RELEVANCE PASS
              ┌────────┼─────────┐
              ▼        ▼         ▼
           in-scope  unclear   out-of-scope
              │        │
              ▼        │
       scenario discovery
              │        │
              ▼        │
        candidate taxonomy
              │        │
              ▼        │
        CLASSIFY EVERY ROW
              │        │
       ┌──────┼────────┘
       ▼      ▼
    covered   novel / ambiguous
       │             │
       │       taxonomy expansion
       │             │
       └──────┬──────┘
              ▼
       COVERAGE CRITIC
              │
              ▼
        Scenario catalogue
              │
              ▼
          SOP candidates
```

The important bit is **CLASSIFY EVERY ROW**.

Clustering or sampling discovers candidate scenarios. It does not establish coverage. After the taxonomy exists, Codex must go back through the entire workbook and account for every ticket/user story. That prevents “these are the eight obvious themes” from masquerading as “these are all the scenarios.”

And you should explicitly tell it that rare/outlier cases are valuable rather than noise. That’s particularly important if you add something like Sentence Transformers or BERTopic: those tools are very useful for semantic neighborhoods and topic discovery, but BERTopic/HDBSCAN deliberately leaves some records as outliers. In your application, those outliers could be exactly the unusual service scenarios you care about.

## The minimal stack I’d use

I wouldn’t install twelve “AI data analyst” projects.

I’d use Codex Skills + qsv + DuckDB, with Sentence Transformers as the one extra semantic tool.

qsv should be the boring structural layer: inspect sheets, columns, nulls, values, frequencies and schemas. Its MCP tooling supports Excel formats directly, and the project has explicitly developed profiling/ontology/reproducibility workflows around agent use.

DuckDB should do the exact accounting: “How many rows were classified?”, “which IDs weren’t assigned?”, “which scenarios overlap?”, “what percentage has missing description?”, etc. DuckDB can read `.xlsx` directly; for a ticket workbook I’d initially ingest it as text (`all_varchar=true`) rather than let Excel type inference accidentally reinterpret weird identifiers or textual values.

Sentence Transformers gives Codex a deterministic way to ask questions such as “which tickets mean similar things despite using different terminology?” It explicitly supports embeddings, semantic search, clustering/community detection and paraphrase mining.

I would make BERTopic optional. It’s useful for discovering candidate themes, but you don’t actually need it for tomorrow if Sentence Transformers + Codex is enough.

Your repo can stay extremely small:

```
service-analysis/
│
├── AGENTS.md
├── .agents/
│   └── skills/
│       └── service-scenario-scout/
│           └── SKILL.md
│
├── .codex/
│   └── agents/
│       └── coverage-critic.toml       # optional
│
├── data/
│   └── raw/
│       └── client_tickets.xlsx
│
├── scripts/
│   └── semantic_index.py             # optional
│
└── analysis/
    ├── data_inventory.md
    ├── scenario_catalog.csv
    ├── row_classification.csv
    ├── review_queue.csv
    ├── sop_candidates.md
    └── analysis_journal.md
```

## The skill I would give Codex

Put the following in `.agents/skills/service-scenario-scout/SKILL.md`. This is deliberately stricter than a normal prompt.

---

```
name: service-scenario-scout
description: Discover, validate, and exhaustively map service scenarios from ticket, backlog, incident, request, and user-story datasets. Use when analyzing support/service Excel sheets or issue queues to determine relevant cases, discover scenario taxonomy, identify rare cases, and develop evidence-backed SOP candidates.
```

### Service Scenario Scout

#### Objective

Determine which records are relevant to the service domain, discover the distinct service scenarios represented in the source material, account for every source record, identify scenarios that may have been missed, and produce evidence-backed inputs for SOP design.

The goal is coverage, not merely summarization.

Do not assume that the most frequent themes are the complete scenario set.

#### Evidence hierarchy

Maintain four epistemic states throughout the analysis:

**OBSERVED** — directly present in source data or deterministically calculated from it.

**VALIDATED** — established through an explicit query, join, count, or verification procedure.

**INFERRED** — a semantic interpretation supported by evidence but not explicitly stated.

**UNKNOWN** — cannot be established from the available material.

Never present INFERRED information as OBSERVED.

#### Source preservation

Never modify the original workbook.

Inventory all sheets and columns before analysis.

Preserve a stable source-row identifier so every conclusion can be traced back to the original row.

Initially treat ambiguous Excel fields as text unless there is a concrete reason to coerce their type.

Record missing and unusable text explicitly rather than silently dropping rows.

#### Semantic evidence

Do not classify records from task title alone when richer fields exist.

Construct the semantic evidence for each record using all useful available fields, including where present:

task or ticket title,
description,
user story,
acceptance criteria,
service/component,
resolution information,
comments or notes,
request type,
status,
labels.

Keep the original fields separately so conclusions remain auditable.

Metadata such as status, assignee, sprint, date, priority, or component may inform interpretation but must not substitute for reading the semantic content.

#### Service relevance

Classify every row into:

IN_SCOPE,
OUT_OF_SCOPE,
or UNCERTAIN.

For each decision retain:

source row identifier,
relevance decision,
confidence,
fields supporting the decision,
and a concise evidence-based rationale.

Never discard UNCERTAIN rows.

Out-of-scope decisions with weak evidence belong in the review queue.

#### Scenario discovery

Discover candidate scenarios from semantic content rather than relying on SQL GROUP BY operations over existing labels.

Use a combination of:

representative examples,
semantic similarity,
diverse samples,
rare records,
unusual terminology,
existing service/component fields,
and semantic outliers.

Existing ticket categories may be evidence but are not assumed to be the correct scenario taxonomy.

When embeddings or clustering are available, use them to propose candidate groupings.

Clusters are hypotheses, not truth.

Never discard an outlier merely because it does not belong to a large semantic cluster. Inspect outliers specifically for rare or previously undiscovered scenarios.

A scenario should describe a distinct service situation, trigger, user intent, failure/request condition, or operational pathway — not merely a repeated word or product name.

Avoid categories so broad that materially different operational responses are collapsed together.

Avoid categories so narrow that trivial wording differences become separate scenarios.

#### Scenario catalogue

For every candidate scenario maintain:

scenario_id,
scenario_name,
definition,
inclusion criteria,
exclusion criteria,
representative source records,
edge cases,
related scenarios,
service/component if known,
confidence,
and unresolved questions.

Scenario names should describe what is happening, not simply repeat ticket terminology.

#### Exhaustive classification

After creating an initial taxonomy, perform a second pass over EVERY source record.

Every row must receive:

source_row_id,
service_relevance,
scenario_id or scenario_ids,
classification confidence,
supporting evidence,
classification status,
and review reason where applicable.

Multi-scenario assignment is allowed when a record genuinely contains multiple operational situations.

Classification status must be one of:

COVERED,
MULTI_SCENARIO,
NEW_SCENARIO_CANDIDATE,
UNCERTAIN,
or OUT_OF_SCOPE.

Do not stop after finding representative examples.

Do not report scenario coverage until every source row has an explicit status.

#### Novelty loop

Collect all NEW_SCENARIO_CANDIDATE and UNCERTAIN rows, along with semantic outliers and low-confidence classifications.

Analyze these separately.

If multiple records reveal a coherent scenario not represented in the catalogue, add the scenario and reclassify affected records.

If a single record describes a materially distinct operational case, preserve it as a rare scenario candidate rather than forcing it into the nearest large category.

Repeat until another novelty pass produces no materially new scenario supported by the data.

#### Coverage verification

Before finalizing, verify deterministically that:

every source row appears in the classification output,

no source rows were silently lost during ingestion,

duplicate identifiers are accounted for,

null-text records are accounted for,

every IN_SCOPE row is either assigned to a scenario or explicitly marked UNCERTAIN/NEW_SCENARIO_CANDIDATE,

and every scenario has traceable source evidence.

Use exact queries or scripts for these checks.

Do not rely on visual inspection.

#### Adversarial review

After the first complete taxonomy, perform an independent critic pass.

The critic’s task is not to summarize the analysis.

The critic must actively attempt to find:

relevant records incorrectly excluded,

rows forced into an inappropriate scenario,

distinct scenarios that were incorrectly merged,

near-duplicate scenarios that should be merged,

rare scenarios hidden among outliers,

unsupported semantic claims,

rows whose classification lacks sufficient textual evidence,

and scenarios whose boundaries cannot be consistently applied.

Where Codex subagents are available, delegate this pass to a fresh read-only agent that has not authored the taxonomy.

Resolve material critic findings before finalizing.

#### SOP analysis

Only begin SOP extrapolation after scenario coverage has stabilized.

For each scenario identify, where supported:

trigger or initiating condition,
user/customer intent,
inputs or prerequisites,
service/component involved,
observed actions or resolution steps,
dependencies,
decision points,
escalations,
exceptions,
outputs,
and evidence records.

Distinguish rigorously between:

**OBSERVED PROCEDURE** — supported by source tickets, user stories, acceptance criteria, or resolution information.

**PROPOSED PROCEDURE** — a reasonable SOP step inferred by the analyst but not directly demonstrated in source data.

**UNKNOWN PROCEDURE** — required information is not present in the source material.

Never invent a complete SOP merely because the scenario itself is identifiable.

#### Required outputs

Create `analysis/scenario_catalog.csv` containing the scenario taxonomy and its evidence.

Create `analysis/row_classification.csv` containing one auditable classification record for every source row.

Create `analysis/review_queue.csv` containing only records that require meaningful human judgment: ambiguous relevance, uncertain classification, novel scenario candidates, taxonomy conflicts, or materially unusual cases.

Create `analysis/sop_candidates.md` summarizing SOP opportunities and clearly separating observed procedure from proposed procedure.

Create `analysis/data_inventory.md` documenting sheets, useful fields, missingness, data-quality limitations, and the semantic evidence used.

Create `analysis/analysis_journal.md` documenting material transformations, queries, assumptions, taxonomy changes, and verification checks.

#### Human-review objective

The objective is not to eliminate human judgment.

The objective is to concentrate human judgment on the small set of decisions where it adds value.

Do not ask the user to read the entire workbook to verify the analysis.

Present the taxonomy, evidence, coverage statistics, and review queue so a human can challenge the uncertain and consequential cases directly.

That’s the actual layer I think you were missing.

The `AGENTS.md` can then stay tiny—things like “never modify `data/raw`, use the service-scenario skill for ticket analysis, preserve source IDs, and make analyses reproducible.” OpenAI specifically recommends keeping Skills focused and giving them explicit inputs/outputs rather than making one giant omnibus instruction file.

## There’s one reliability trick I would absolutely add

Have a second Codex agent attack the first agent’s taxonomy.

Not “review the writing.” Give it a hostile objective:

```
You are the coverage critic.
Assume the proposed taxonomy is incomplete or misleading.
Search the entire classified dataset for:
- relevant records incorrectly excluded
- apparently similar tickets requiring different operational responses
- rare cases hidden inside common categories
- rows assigned with weak evidence
- scenarios whose inclusion/exclusion boundaries are inconsistent
- clusters whose labels hide multiple distinct service situations
Do not propose SOPs.
Return only concrete counterexamples with source row IDs.
```

Codex’s current subagent architecture is intended for exactly this kind of narrow exploration/review role, and custom project agents can live under `.codex/agents/*.toml`.

That changes the meeting dynamic quite a lot.

Instead of you saying to your boss:

“Codex says there are 11 scenarios; I manually skimmed them and it seems okay.”

you can say:

“The first pass found 11 scenarios. Every in-scope record was mapped against the taxonomy. A separate coverage pass challenged the assignments and found three edge cases; two produced a new scenario and one was reclassified. These are the seven records still requiring human judgment.”

That’s a much more defensible agentic workflow.

## And you still shouldn’t aim for literally zero review

The human work should move from reading hundreds/thousands of rows to reviewing three things: the proposed scenario boundaries, the genuinely ambiguous/novel records, and any operational assumptions that become SOP steps.

In fact, if this is something you’re going to reuse for the client, I’d eventually make a tiny challenge set: a handful of obvious, subtle, rare and disputed tickets that you and your boss agree on. Every future change to the skill has to classify those correctly. That’s far more valuable than repeatedly eyeballing the entire workbook.

OpenAI itself now presents “analyze issue queues/feedback, group repeated problems into clear themes, preserve supporting evidence, and surface follow-ups” as a normal supported workflow. Your requirement is simply stricter because you need scenario completeness + SOP extrapolation, so we add the exhaustive-classification and critic loops.

And crucially: don’t judge your previous Codex workflow by the fact that it needed course correction. The weak point was that “find all scenarios” was treated as a single reasoning request. What we’re doing here is turning it into a testable procedure with coverage accounting and exceptions.

Tomorrow, your first Codex instruction can genuinely be almost this short:

```
$service-scenario-scout
Analyze data/raw/client_tickets.xlsx.
Our domain of interest is service cases. Discover the service-scenario
taxonomy, account for every source row, run the adversarial coverage
review, and produce the required analysis outputs.
Do not begin SOP extrapolation until scenario coverage is verified.
```

That’s the level of ergonomics I think you were originally looking for.
