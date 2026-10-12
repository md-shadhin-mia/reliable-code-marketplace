# Reliable Code Skills

Two skills that help Claude write backend and frontend code with far fewer bugs:

- **backend-reliability**: define the contract, test edge cases (empty input, boundaries, duplicates, auth, downstream failures, concurrency), follow safe implementation rules, then verify by running type checks, lint, tests and the live service.
- **frontend-reliability**: handle every UI state (loading, error, empty, extreme data), prevent async races and double submits, clean up effects, check accessibility, then verify with a production build and real browser checks.

No process guarantees zero bugs. These skills make bugs visible early and always end with an honest report of what was and wasn't tested.

## Install in Claude Code

```bash
claude plugin marketplace add md-shadhin-mia/reliable-code-marketplace
claude plugin install reliable-code@reliable-code-marketplace
```

Or inside a Claude Code session:

```
/plugin marketplace add md-shadhin-mia/reliable-code-marketplace
/plugin install reliable-code@reliable-code-marketplace
```

Claude uses the skills automatically on backend or frontend coding tasks. You can also call them directly with `/reliable-code:backend-reliability` or `/reliable-code:frontend-reliability`.

## Install in the Claude app

Download the skill folders from `plugins/reliable-code/skills/`, zip each one, and upload it in the Skills section of your Claude settings.
