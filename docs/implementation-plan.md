# Docker replay reviewer to ChatGPT Project

## Goal

Build a small local web tool for post-game reviews across heroes and roles.
There is no required interaction during play. The user enters a match ID,
uploads a replay if retrieval fails, selects their player, and exports one
Markdown report to upload manually to a ChatGPT Project.

The review should help evaluate skill progression, item order, target selection,
ally coordination, good decisions, and concrete improvement opportunities.
The initial release uses no paid AI API. Local code parses and packages evidence;
ChatGPT provides conversational coaching after the user uploads the report.

## Docker architecture

Use Docker Compose with named volumes for PostgreSQL and replay/report artifacts:

| Service | Responsibility |
| --- | --- |
| web | Next.js/TypeScript UI, imports, player selection, report download |
| worker | Retrieval, parse jobs, feature extraction, rules, report generation |
| parser | Internal-only Java replay parsing service |
| db | PostgreSQL, including persistent job state |
| migrate | One-shot database setup |

Target startup: `docker compose up --build`, without host language runtimes.
Publish only the web port; keep the database and parser on the internal network.
Add health checks and ensure jobs and data survive container restarts.

Use a pinned [OpenDota parser](https://github.com/odota/parser) as the baseline.
It accepts `.dem` input and emits JSON event logs. Extend extraction with
[Clarity](https://github.com/skadistats/clarity) where supported state measurements
are needed. Record parser versions with every report.

## Import pipeline

Match ID or file -> replay artifact -> parser output -> normalized evidence ->
review rules -> downloadable report.

- Obtain match metadata and replay-location information through OpenDota where
  available. If retrieval fails, offer `.dem`/`.dem.bz2` upload.
- Select the reviewed player; preselect a saved account ID when available.
- Summaries may supplement evidence but never substitute silently for a replay.
- Validate provider download hosts and enforce upload, decompression, parser
  memory, and execution-time limits.
- Keep jobs retryable and reuse existing artifacts for duplicate imports.

## Evidence and analysis

Normal code extracts facts and identifies review candidates. Strategic judgments
remain explicitly separate from measurements.

| Area | Evidence and review focus |
| --- | --- |
| Skills | Upgrade order, legality, patch-specific build deviations |
| Items | Purchases, components, progression, timing, lineup alternatives |
| Lane | Available economy/experience measurements, deaths, recovery |
| Fights | Casts, targets, damage, disables, deaths, allied involvement |
| Coordination | Available positioning and action sequences around key events |
| Objectives | Observable activity following fights and review opportunities |

Reviews emphasize initiation/follow-up, ability impact, target selection,
coordination, and farming/fighting decisions where supported by evidence.

Maintain a telemetry capability map for each parser version. Verify sample
replays before enabling a finding. Missing visibility, cooldown, position, or
cast-target data disables any dependent finding.

Do not use hidden enemy information to criticize a decision. A won fight does
not prove a good decision, and a lost fight does not prove a mistake. Missing
events must not be treated as proof that an action never occurred.

## Single-file report

Generate `match-<id>-review.md` with:

- Match, hero, player, role, patch, result, and data completeness.
- Actual skill progression and purchase timeline.
- Supported positive patterns and improvement candidates.
- Timestamped moments and nearby context, each with stable evidence IDs.
- Observations separated from interpretations and evidence strength.
- Uncertainties and questions for the player.
- Up to three suggested practice priorities.
- A compact evidence appendix for ChatGPT, not the entire raw event stream.

Do not force a fixed number of compliments or mistakes without supporting data.

## Storage and knowledge

Store match/player records, replay artifacts, parse runs, normalized events,
selected state samples, derived measurements, findings, reports, jobs, and
settings. Keep large raw parser outputs in the artifact volume and queryable
events/features in PostgreSQL. Record checksums, versions, and completeness.

Use immutable patch-specific mechanics and reviewed hero rules. Valve references
and versioned OpenDota constants ground mechanics. Curated Dotabuff and
Dota2ProTracker references must carry dates. Statistical comparisons require
cohort and sample information; professional timings are not mandatory targets.

Use the match's patch. If historical knowledge is missing, export observed
facts while suppressing patch-dependent build judgments.

## ChatGPT handoff

Provide a companion Project-instructions document:

> Review my Dota 2 replay report. Separate observations from
> interpretations. Cite timestamps and evidence IDs. Explain good decisions as
> well as improvement opportunities. Assess skill progression, purchase order,
> target selection and ally coordination using only supported context. Do not
> infer that missing events never happened or judge decisions using hidden enemy
> information. Finish with three actionable priorities for my next match.

The user uploads one generated report after the match. Automatic insertion into
a particular ChatGPT Project is not a verified integration and is outside v1.

## Delivery order

1. Compose setup and end-to-end parsing of representative replays.
2. Verify extracted fields and publish the capability map.
3. Match-ID retrieval, upload fallback, player selection, persistent jobs.
4. Evidence normalization, patch-aware rules, and single-file export.
5. Human validation before expanding coverage.

## Acceptance criteria

- Fresh Compose startup succeeds; state survives restarts.
- Failed downloads lead to a usable upload fallback.
- Both compressed and uncompressed replays work.
- Corrupt input, unsupported replay versions, and parser crashes fail clearly.
- Player timelines match manually checked replay moments.
- Every finding references evidence; missing telemetry produces limitations.
- Historical matches never silently use current mechanics.
- Identical input and versions produce equivalent factual output.
- The exported report supports coaching without another screenshot or replay.

## Risks and boundaries

Replay availability, parser compatibility, telemetry coverage, and strategic
judgment are the main uncertainties. Prioritize reliable evidence over claiming
to identify every mistake. Live coaching, screenshot planning, automated Project
uploads are outside this initial implementation.
