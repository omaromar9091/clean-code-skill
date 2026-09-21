# clean-code-skill

English | [العربية](README.ar.md)

A skill for OpenCode and Claude Code that reviews and refactors code against 11 clean-code rules. It works in small steps and keeps the behavior of the code unchanged.

## What it does

Point the agent at a file or a folder. It reads the code, writes a report of the problems it found sorted by severity, and, if you ask for it, applies the fixes one rule at a time, running your tests or the app after each step.

There are two modes: review only (a report, no edits) and refactor (edits). Either way it leaves alone the things other code depends on: route URLs, database columns, public APIs, UI text, and translations. It does not add dependencies without asking. Changes that alter behavior, such as new error handling, are listed separately so you can approve them first.

## The 11 rules

| # | Rule | In short |
|---|------|----------|
| 1 | Naming | Names say what a thing is or does |
| 2 | Comment | Explain why, not what the code already says |
| 3 | Consistency | One style across the whole project |
| 4 | Early Return | Exit early instead of nesting conditions |
| 5 | Function | Each function does one job |
| 6 | Single Responsibility | Each module or class has one reason to change |
| 7 | Abstraction | Hide noisy details, but don't over-abstract |
| 8 | Separation of Concerns | Keep UI, API, and business logic apart |
| 9 | DRY | Repeated code lives in one place |
| 10 | Don't Repeat Logic | Each business rule lives in one place |
| 11 | Error Handling | Handle failures explicitly instead of assuming success |

The rules fall into four groups: readability, structure, duplication, and error handling. The agent reads the details for a group only when it finds something in that group.

## Install

The skill is the `clean-code/` folder. Copy it into your agent's skills directory. Run these from a folder that does not already contain a `clean-code-skill` folder.

OpenCode, for all projects (Linux, macOS):

```bash
git clone https://github.com/omaromar9091/clean-code-skill.git
mkdir -p ~/.config/opencode/skills
cp -r clean-code-skill/clean-code ~/.config/opencode/skills/
```

OpenCode on Windows (PowerShell):

```powershell
git clone https://github.com/omaromar9091/clean-code-skill.git
New-Item -ItemType Directory -Force "$HOME\.config\opencode\skills"
Copy-Item -Recurse clean-code-skill\clean-code "$HOME\.config\opencode\skills\"
```

For a single project, copy `clean-code/` into `.opencode/skills/` inside that project. For Claude Code, copy it into `~/.claude/skills/` (all projects) or `.claude/skills/` (one project).

Restart the agent after copying.

## Usage

Naming the skill in your prompt is the most reliable way to use it:

```
Use the clean-code skill to review src/ (report only)
Use the clean-code skill to refactor routes.py
Apply only Early Return and Naming from the clean-code skill on services/
```

It also starts on its own when you ask to clean up, refactor, or review code. Arabic prompts work too:

```
راجع الكود ده بـ clean-code skill وطلعلي تقرير بس
نضّف ملف routes.py وطبّق Early Return وفصل المسؤوليات
```

Commit your work before you start, so any step can be undone. Projects with tests get better results, because the agent can run them after every change.

## Layout

```
clean-code-skill/
├── README.md
├── README.ar.md
├── LICENSE
├── clean-code/                  the skill; copy this folder
│   ├── SKILL.md                 workflow, rules table, limits, report format
│   └── references/
│       ├── readability.md       Naming, Comment, Consistency, Early Return
│       ├── structure.md         Function, Single Responsibility, Abstraction, Separation of Concerns
│       ├── duplication.md       DRY, Don't Repeat Logic
│       └── error-handling.md    Error Handling
└── examples/
    └── flask-refactor.md        a full before and after
```

`SKILL.md` is loaded whenever the skill starts. The files in `references/` are read only when needed, which keeps the context small.

## Feedback

Found a problem or have a suggestion? Open an issue: https://github.com/omaromar9091/clean-code-skill/issues

## License

MIT. See [LICENSE](LICENSE).
