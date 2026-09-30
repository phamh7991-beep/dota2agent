# dota2agent

A planned local Dota 2 replay analysis tool for players of any hero or role.

**The primary output is Markdown content you can give to an AI chat for gameplay
advice.** After a match, provide its ID or upload a replay. The tool will parse
the replay and organize match context, timelines, and timestamped evidence into
a readable report. Paste the Markdown or upload the report to ChatGPT or another
AI chat to discuss what you did well, what to improve, and how to approach your
next game.

No screenshots or interaction are required while playing. The initial version
will generate reports locally without a paid AI API; you choose the AI chat
where you want to get advice.

## Status

Planning and repository setup only. The application, Docker Compose services,
replay parser integration, and analysis rules are not implemented yet.

See [the implementation plan](docs/implementation-plan.md) for architecture,
delivery milestones, and acceptance criteria. Analysis depth will depend on
available replay data and validated rules.

## Intended workflow

1. Start the application with Docker Compose.
2. Enter a match ID; upload `.dem` or `.dem.bz2` if download is unavailable.
3. Select your player.
4. Copy the generated Markdown or download the `.md` report.
5. Paste or upload it to your preferred AI chat and ask for gameplay advice.

## Markdown for AI coaching

Reports are intended to give an AI chat concrete evidence for reviewing:

- Skill builds and leveling decisions.
- Item purchases, their order, and timing.
- Target selection, fights, and coordination with allies.
- Positive decisions and opportunities to improve.

Each report will include match context, timestamps, and evidence references,
distinguish observed facts from interpretations, and identify missing data.
The AI chat can use that context to explain tradeoffs and suggest practice
priorities. Reports will not claim to establish mistakes that the replay data
cannot support.

Example prompt to send with a report:

> Review this Dota 2 match report. Explain what I did well and what I could
> improve, citing timestamps and evidence. Evaluate my skill build, item order,
> target selection, and team coordination where the data supports it. Finish
> with three actionable priorities for my next game, and flag uncertainty.

## Local data

Credentials, environment files, personal replays, generated reports, and runtime
data must stay out of Git. The repository includes ignore rules for their
standard locations. Do not commit downloaded match data or tokens.
