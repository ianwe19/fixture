# AGENTS.md

Guidance for AI agents (and new collaborators) working in this repo. Read before making changes.

## What this project is

"Fixture — a bold, designer-y, interactive narrative
single-page website. It is the web component of a Graphic Design BFA thesis: part
portfolio piece, part demonstration of development skill.

## The user

- Graphic Design BFA student, not a professional developer.
- Has programming background but is rusty and new to this specific stack
  (React 19, Vite, TypeScript, Tailwind v4, React Three Fiber).
- Learning as they go: wants to understand every change, not just receive them.

## Working agreements (important)

- **Do not edit files directly.** Present changes as code blocks in chat; the user
  applies them themselves. You may write to files **only if the prompt contains the
  exact phrase "go ahead and edit."** A stated goal or intent ("I want to create X")
  is not permission.
- **Present code as clean, paste-ready blocks.** Code blocks contain only what the
  user pastes — no line numbers or `+`/`-` markers inside. State the location in
  prose before each block ("add after line N" / "replace lines N–M"); apply
  bottom-up so line numbers stay valid. Only reproduce a full file for genuinely
  new files, or full rewrites the user has approved.
- **Explain each change in one line** — the "why" matters as much as the code.
- **Keep changes minimal and scoped.** No drive-by refactors, no unrequested
  abstractions or dependencies. If a change seems to require restructuring,
  propose it first and wait for approval.
- **Don't remind the user to verify.** They run `npm run lint` and `npm run build`
  at regular intervals; don't end responses with "run X and paste the output" or
  offers to verify. When they do report results, the success bar is zero lint
  problems and a clean build (the ~1.2 MB chunk warning from three.js is expected,
  not a failure). When I write files directly (per the exception above), I run
  both myself before declaring done.

## Commands

| Command           | Purpose                                          |
| ----------------- | ------------------------------------------------ |
| `npm run dev`     | Dev server with HMR at http://localhost:5173     |
| `npm run build`   | Type-check + production build to `dist/`         |
| `npm run preview` | Serve the production build locally               |
| `npm run lint`    | OXLint (must stay clean)                         |

## Toolkit (token-efficient tooling)

Principle: give the agent the **smallest amount of information needed for the
next correct decision**. Each tool has one job — pick the one that matches the
question, keep its output bounded, and stop once you have enough to act.

### Choose the tool by the type of information you need

| You need…                          | Use                                                 |
| ---------------------------------- | --------------------------------------------------- |
| a string / identifier / error text | `rtk rg "term" src/`                                |
| files by name or path pattern      | `fd 'pattern' src/`                                 |
| a code construct / syntax pattern  | `rtk ast-grep run --pattern '<p>' --lang tsx src/`  |
| who defines / references a symbol  | Serena — **not installed**; fall back to `rtk rg`   |
| one field from a JSON document     | `jq '.field' file.json`                             |
| one field from a YAML document     | `yq '.field' file.yml`                              |
| a GitHub PR / issue / Actions run  | `rtk gh pr list`                                    |
| git status / a diff                | `rtk git status` · `rtk diff`                        |

**RTK** is a prefix wrapper that shrinks command output before it reaches the
context. It has compact forms for `rg`, `ast-grep`, `find`, `git`, `gh`, `read`,
`diff`, `json`, `ls`, `log` and accepts any other command as a passthrough.
`fd` / `jq` / `yq` are already selective — run them directly. (`rtk init -g`
optionally installs a global hook that auto-wraps commands.)

### Locate before you read

Find the file (`fd`) or the line (`rg`) first, then read only the relevant range.
Never `cat` or read a whole file to find one line, and never read a whole
JSON/YAML document when `jq` / `yq` can pull a single field.

### Tools for the human, not the agent

`bat`, `delta`, `fzf` are for the terminal (highlighting, interactive fuzzy
selection). Don't reach for them autonomously — an agent wants deterministic,
bounded output, which `rg` / `fd` / `jq` give.

### Stopping rules (the biggest lever)

- Stop searching once enough evidence exists to act.
- Don't open extra files "for completeness."
- Don't re-run a search that already succeeded with another tool.
- Don't keep exploring after you've found the relevant implementation.
- After a successful verification, stop — unless more is clearly justified.

An agent with ten good tools still wastes context if it uses all ten just because
they exist. Use the minimum that answers the question.

**Session-level controls** (set by the user, not selected per query): Headroom
trims carried context, Caveman keeps the agent terse, and Ponytail keeps
implementations minimal. They cut token use independently of the tools above.

## Git

Small, focused commits with messages that explain *why*. The initial commit
establishes the baseline: project setup + hero (3D) + second section.

## Documentation (live docs access)

- Agents can read current documentation online (web search + page extraction).
  Verified working: R3F docs (`r3f.docs.pmnd.rs`), drei docs
  (`drei.docs.pmnd.rs`, e.g. `/loaders/gltf-use-gltf`), three.js docs.
- Before using an API you're not confident about — especially the staged deps
  (postprocessing, cannon) and drei helpers — check the current docs rather than
  relying on training data.
- For stable APIs already in use (e.g. `useFrame`, basic JSX elements), the
  codebase is the source of truth; no need to re-verify.
- If a docs URL fails to extract, fall back to searching for the correct path
  or reading the source on GitHub.
