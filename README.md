# clean-code-skill

**English** | [العربية](README.ar.md)

An agent skill that makes your AI coding agent review and refactor code against **11 clean-code rules**, in a safe, step-by-step workflow that **preserves behavior**.

Works with [OpenCode](https://opencode.ai) and Claude Code (and any agent that reads the `SKILL.md` format).

## Why this skill?

A plain list of rules makes an agent produce vague, inconsistent changes. This skill gives it:

- **A workflow:** scope, baseline, scan, report, apply in a safe order, verify, summarize.
- **Two modes:** review only (report), or refactor (apply changes).
- **Guardrails:** it won't rename routes, DB columns, or public APIs, won't touch UI text or translations, and won't add dependencies without asking.
- **Behavior-change awareness:** anything that changes behavior (like new error handling) is listed separately for your approval.
- **Concrete patterns:** smell, fix, and before/after for every rule, plus a full Flask example.
- **Honesty:** if your code is already clean, it says so instead of inventing findings.

## The 11 rules

| # | Rule | In one line |
|---|------|-------------|
| 1 | Naming | Clear, direct names that say what a thing is or does |
| 2 | Comment | Explain **why**, not what the code already says |
| 3 | Consistency | One consistent style across the project |
| 4 | Early Return | Exit early instead of nesting conditions |
| 5 | Function | Each function does one specific job |
| 6 | Single Responsibility | Each part of the code has one clear responsibility |
| 7 | Abstraction | Hide noisy details, without over-abstracting |
| 8 | Separation of Concerns | Keep UI, API, and business logic apart |
| 9 | DRY | Repeated code lives in one place |
| 10 | Don't Repeat Logic | Each business rule lives in one place |
| 11 | Error Handling | Don't assume success; handle failures explicitly |

The rules are grouped into four families (readability, structure, duplication, error handling) so the agent loads only the guidance it needs.

## Install

The skill is the `clean-code/` folder. Copy it into your agent's skills directory.

**OpenCode, global (all projects)**
```bash
git clone https://github.com/<your-username>/clean-code-skill.git
mkdir -p ~/.config/opencode/skills
cp -r clean-code-skill/clean-code ~/.config/opencode/skills/
```

**Windows (PowerShell)**
```powershell
git clone https://github.com/<your-username>/clean-code-skill.git
New-Item -ItemType Directory -Force "$HOME\.config\opencode\skills"
Copy-Item -Recurse clean-code-skill\clean-code "$HOME\.config\opencode\skills\"
```

**Per project:** copy `clean-code/` to `.opencode/skills/` inside the project.

**Claude Code:** copy it to `~/.claude/skills/` (global) or `.claude/skills/` (project).

Restart the agent after installing.

## Usage

Naming the skill explicitly is the most reliable way to trigger it:

```
Use the clean-code skill to review src/ (report only)
Use the clean-code skill to refactor routes.py
Apply only Early Return and Naming from the clean-code skill on services/
```

It also triggers on its own when you ask to clean up, refactor, review, or improve the quality of code.

Arabic works too:

```
راجع الكود ده بـ clean-code skill وطلعلي تقرير بس
نضّف ملف routes.py وطبّق Early Return وفصل المسؤوليات
```

## Structure

```
clean-code-skill/
├── README.md / README.ar.md
├── LICENSE
├── clean-code/                  <- the skill (copy this folder)
│   ├── SKILL.md                 workflow, rules table, guardrails, report format
│   └── references/
│       ├── readability.md       Naming, Comment, Consistency, Early Return
│       ├── structure.md         Function, Single Responsibility, Abstraction, Separation of Concerns
│       ├── duplication.md       DRY, Don't Repeat Logic
│       └── error-handling.md    Error Handling
└── examples/
    └── flask-refactor.md        full before/after
```

`SKILL.md` is short and always loaded when the skill triggers. The `references/` files are read only when needed, which keeps the agent's context small.

## Tips

- Try it on a small project first, then on a bigger one.
- Projects with tests get the best results, because the agent can verify each step.
- Commit before you start, so any step can be reverted.
- Tell the agent which rules matter most to you, or which folders to skip.

## Contributing

Issues and pull requests are welcome: new before/after examples for other languages and frameworks (JavaScript/TypeScript, C#, Java, Go...), better smell detection, and translations.

## License

MIT. See [LICENSE](LICENSE).
