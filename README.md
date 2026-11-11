# Conversation Handoff

**[English](README.md) | [العربية](README.ar.md)**

A [Claude Skill](https://docs.claude.com) that turns a long chat into a **`CONVERSATION_HANDOFF.md`** document, so you can continue the work in a brand-new conversation without re-explaining anything.

It does not just summarize what was said. It documents the **current working state**: the goal, what was decided vs. only proposed, the technical context, the open issues, and the exact point to resume from.

> **Success criterion:** a new assistant reads only the handoff file, then continues from where you stopped, without asking for context, re-debating decisions that were settled, or repeating finished work.

---

## Why use it?

Long conversations hit length limits, lose focus, or need to move to a new chat. A plain summary loses the details that matter. This skill keeps them:

- **Facts are separated from guesses.** A proposal is never silently promoted to a decision.
- **Technical details are kept literally:** table, column, class, file and project names, plus the latest approved code.
- **Failed attempts are recorded** in one line each, so they are not tried again.
- **There is one clear resume point** (`NEXT ACTION`) that is enough on its own to start a new chat.

---

## Status tags

Every important item in the document carries one tag:

| Tag | Meaning |
|---|---|
| ✅ **Confirmed** | A decision the user explicitly approved (verbal approval, a clear choice between options, or an order to implement that specific option). |
| 💡 **Proposed** | Suggested but not approved. Silence or changing the subject is **not** approval. |
| ⚠️ **Assumption** | Something the work was built on without verification. |
| ❓ **Open** | A question or problem that is not yet resolved. |
| 🔧 **Done** | Work that was actually completed (a file produced, code written and tested), with evidence when possible. |

Rules: when in doubt, a statement is marked 💡 or ⚠️, nothing is described as "working" unless it was tested in the conversation, and newer statements override older ones (the change is noted when it matters).

---

## Operating modes

The mode is detected from your request:

| Request | Mode | What happens |
|---|---|---|
| "Create the handoff summary" | **Create** | Builds a complete, latest version of the file from the whole conversation. |
| "Update the handoff summary" | **Update** | Merges new progress into the previous document (keeps what still matters, replaces what changed, moves finished items out of Next Steps and Open Issues). |
| "Prepare the conversation for transfer" | **Finalize** | Produces a complete, transferable version, runs the full quality check, and reports any gaps. |

If the mode is not clear, the skill assumes **Create**, or **Update** if a previous handoff already exists in the chat.

Trigger phrases include: `handoff`, `resume point`, `continue in a new chat`, `context handoff`, and the Arabic phrases `أنشئ ملخص الاستئناف`, `حدّث ملخص الاستئناف`, `جهّز المحادثة للانتقال`, `نقطة الاستئناف`. It also triggers when you say the chat is getting too long or is hitting limits, even if you never mention a file.

---

## Output structure

The generated `CONVERSATION_HANDOFF.md` follows a fixed template ([`assets/CONVERSATION_HANDOFF.template.md`](conversation-handoff/assets/CONVERSATION_HANDOFF.template.md)):

1. **Overview** — the main goal and the background needed to understand it.
2. **Objectives & Requirements** — goals, constraints, and conditions to respect.
3. **Confirmed Decisions** — only what the user approved, with reasons. Plus separate lists for 💡 proposals and ⚠️ assumptions.
4. **Current State** — what is done (🔧) and exactly where the work stands now.
5. **Technical Context** — tech stack and versions, architecture, important files, conventions, the latest approved code, attempts tried, and ADRs. Removed if the chat is not technical.
6. **Open Issues** — unresolved questions and missing information.
7. **Next Steps** — prioritized list.
8. **NEXT ACTION — Resume Point** — the most important section (see below).
9. **User Preferences & Working Rules** — language, format, tools, naming rules.

The file always ends with a fixed instruction telling the new assistant to read the whole document, not start from scratch, not re-discuss approved decisions, start from `NEXT ACTION`, and ask at most one specific question if essential information is missing.

### The NEXT ACTION section

It contains, in order:

1. The last step that was completed.
2. The last topic or problem being worked on.
3. The immediate next task (one specific, executable action, not a generic list).
4. What is needed to continue (files, code, information, and where to get them).
5. Warnings and decisions that must not be ignored.

---

## Quality check before delivery

Before handing over the file, the skill verifies that:

- a new assistant can understand the project without the original chat;
- the last point reached is clear;
- the exact next step is clear;
- approved decisions are separated from proposals and assumptions;
- technical details are preserved literally;
- missing information is listed explicitly instead of being invented.

---

## Installation

### Claude.ai / Claude Desktop

1. Download `conversation-handoff.skill` from this repository.
2. Open **Settings → Capabilities → Skills** and upload the file (menu names may differ slightly between versions).
3. Make sure the skill is enabled.

### Claude Code

Copy the skill folder into your skills directory:

```bash
# Personal (all projects)
cp -r conversation-handoff ~/.claude/skills/

# Or per project
mkdir -p .claude/skills && cp -r conversation-handoff .claude/skills/
```

---

## Usage

In the chat you want to continue elsewhere, just say one of:

```text
Create the handoff summary
Update the handoff summary
Prepare the conversation for transfer
```

Claude saves `CONVERSATION_HANDOFF.md` and presents it. Then open a **new chat**, attach or paste the file, and continue: the new assistant starts directly from `NEXT ACTION`.

---

## Language and formatting

- Explanatory text is written in **Arabic**, while section headings stay in **English**, as in the template.
- Code, file names, variables, classes, tables and columns are kept in **English**, exactly as they appear.
- You can adapt the language by editing `SKILL.md` and the template.

---

## Repository structure

```text
.
├── README.md                          # English documentation
├── README.ar.md                       # Arabic documentation
├── conversation-handoff.skill         # Ready-to-upload package
└── conversation-handoff/
    ├── SKILL.md                       # Skill instructions
    └── assets/
        └── CONVERSATION_HANDOFF.template.md   # Output template
```

> A `.skill` file is a regular ZIP archive with a different extension.

---

## Guiding principle

Do not summarize a conversation just to preserve what happened. Document the state of the work so that what is unfinished can be completed, without losing context or redoing finished work.

---

## License

Add a `LICENSE` file of your choice (for example MIT) before publishing.
