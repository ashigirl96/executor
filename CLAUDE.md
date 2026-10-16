@AGENTS.md

Never use Fable for subagents, including in workflows. Use Sonnet for
subagents unless there is a good reason to use a different model, and always
explicitly pass the model when spawning a subagent.

## Agent skills

### Issue tracker

Issues live in GitHub Issues on the `ashigirl96/executor` fork, never upstream. See `docs/agents/issue-tracker.md`.

### Triage labels

The five default triage labels, unchanged. See `docs/agents/triage-labels.md`.

### Domain docs

Single-context: root `GLOSSARY.md` and `docs/adr/`. See `docs/agents/domain.md`.
