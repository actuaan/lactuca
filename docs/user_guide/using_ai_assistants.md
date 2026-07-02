# Working with AI assistants

## Professional responsibility

Lactuca is a **professional actuarial tool**. Results depend on inputs, tables, and
assumptions **you** choose. AI assistants can draft code faster, but they can also
produce plausible-looking **wrong** premiums, reserves, or API usage.

You remain fully responsible for every numerical result and for compliance with
applicable regulations and professional standards. See the {doc}`../eula` §10.2–10.3
for the full disclaimer. Spanish legal text: {doc}`../eula_es`.

---

## Verifying AI-generated code

Before relying on any snippet:

1. **Run** — Execute it in an activated Python environment (`import lactuca` succeeds).
2. **Cross-check** — Compare magnitude with a {doc}`../cookbook` recipe or a simplified
   manual check.
3. **API audit** — Confirm method and parameter names against {doc}`../api/index` or IDE
   autocompletion.
4. **On failure** — Read {doc}`../errors_reference` and paste the **exact** traceback
   into the assistant; do not ask it to “fix” without error context.

---

## What these files are / are not

**For end users** who installed Lactuca via pip:

- Curated **context files** so Cursor, Copilot, ChatGPT, Claude, etc. use the real Lactuca API.
- Downloadable markdown and IDE templates (see below).

**Not included here:**

- Internal **developer** tooling (repository `.cursor/rules/`, contributor Copilot guides,
  audit skills). Those are not shipped with the package.

**Not a substitute** for the full documentation, {doc}`../api/index`, or
actuarial judgment.

---

## Download the context pack

Files are served from the documentation site (save locally for offline use):

| File | Purpose |
|------|---------|
| <a href="../_static/ai/lactuca-ai-core.md" download="lactuca-ai-core.md">lactuca-ai-core.md</a> | **Always attach** — API, pitfalls, `payment_times` / `tiered_amounts`, date utilities (`alb`, `years_between`, …), links |
| <a href="../_static/ai/lactuca-user.mdc" download="lactuca-user.mdc">lactuca-user.mdc</a> | Cursor rule template (copy to your project) |
| <a href="../_static/ai/copilot-instructions-lactuca.md" download="copilot-instructions-lactuca.md">copilot-instructions-lactuca.md</a> | VS Code Copilot instructions template |
| <a href="../_static/ai/prompt-templates.md" download="prompt-templates.md">prompt-templates.md</a> | Copy-paste prompts |
| <a href="../_static/ai/lactuca-ai-batch.md" download="lactuca-ai-batch.md">lactuca-ai-batch.md</a> | Optional — portfolio / batch |

Production URLs (terminal download):

```bash
curl -fsSL -o lactuca-ai-core.md https://www.lactuca.io/latest/_static/ai/lactuca-ai-core.md
```

- Index for agents: `https://www.lactuca.io/llms.txt`

---

## Match docs to your installed version

Context files declare `compatible_with_docs` in the header. Your installed version:

```python
import lactuca
print(lactuca.__version__)
```

Documentation defaults to **latest** (`https://www.lactuca.io/latest/...`). If your
package version differs, use the **version switcher** on the docs site or replace
`latest` with `/vX.Y.Z/` in bookmarked URLs.

---

## Quick start

1. Download <a href="../_static/ai/lactuca-ai-core.md" download="lactuca-ai-core.md">lactuca-ai-core.md</a>.
2. Open your AI tool (see sections below).
3. Attach the core file (or paste its contents at the start of the chat).
4. Use a template from <a href="../_static/ai/prompt-templates.md" download="prompt-templates.md">prompt-templates.md</a> or ask:

   > Using the attached Lactuca context, write a runnable script for a 15-year
   > temporary annuity-due, male age 50, table PER2020_Ind_1o, cohort=1960, i=3%. Use `äx` and
   > `discrete_precision`. Do not invent parameters.

5. **Verify** using the workflow above.

---

## Cursor

1. Copy <a href="../_static/ai/lactuca-user.mdc" download="lactuca-user.mdc">lactuca-user.mdc</a> to your project:
   `.cursor/rules/lactuca-user.mdc`
2. Or reference the core in chat: `@lactuca-ai-core.md` (after saving it in the project).
3. Keep the core next to your actuarial scripts for `@` mentions.

`alwaysApply: true` in the template loads Lactuca conventions in every chat; for smaller
context use `@file` only when working on Lactuca code.

---

## VS Code + GitHub Copilot

1. Copy <a href="../_static/ai/copilot-instructions-lactuca.md" download="copilot-instructions-lactuca.md">copilot-instructions-lactuca.md</a>
   to your project as `.github/copilot-instructions.md`, **or** paste the core into
   **Copilot → Customize Chat → Instructions**.
2. In chat, `#file:lactuca-ai-core.md` if the file lives in the workspace.

---

## ChatGPT / Claude / Gemini

- **Projects / Custom GPT:** upload `lactuca-ai-core.md` (+ optional `prompt-templates.md`)
  as knowledge files.
- **Ad hoc chat:** paste the core at the beginning; re-attach in long threads.
- **Web discovery:** agents may read `https://www.lactuca.io/llms.txt`.

JupyterLab, VS Code notebooks, and Google Colab use the **same context files** as
scripts. Save or upload `lactuca-ai-core.md` (and optionally `prompt-templates.md`) next
to your notebook. In VS Code, use the Copilot or Cursor chat panel as for `.py` files.
No notebook-specific extension is required.

---

## Prompt templates

Full templates:
<a href="../_static/ai/prompt-templates.md" download="prompt-templates.md">prompt-templates.md</a>

Template #1 references the package version via `compatible_with_docs` in the attached core
file (or substitute `import lactuca; lactuca.__version__` after you verify it matches).

Minimal example:

```
Context: attached lactuca-ai-core.md (Lactuca actuarial Python library).
Task: Price a 20-year term insurance, male 45, PASEM2020_NoRel_1o, i=3%.
Use Ax(n=20). If unsure about a parameter, say so — do not invent API.
```

---

## Portfolio and batch work

Attach <a href="../_static/ai/lactuca-ai-batch.md" download="lactuca-ai-batch.md">lactuca-ai-batch.md</a> in addition to the core.
See {doc}`batch_calculations` for full semantics (`benefits=`, `on_error`, `record_ids`).

---

## Updating context when you upgrade Lactuca

After `pip install --upgrade lactuca`:

1. Check `lactuca.__version__` against `compatible_with_docs` in your saved core file.
2. Re-download the pack from this page if versions differ.
3. Re-run a known {doc}`../cookbook` recipe to sanity-check.

---

## Further reading

- {doc}`getting_started` — installation and first calculations
- {doc}`notation_glossary` — symbols and parameter names
- {doc}`../cookbook` — verified recipes
- {doc}`../errors_reference` — `ValueError` messages
- {doc}`../activation` — licensing
