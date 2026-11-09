# arete-openspec — family store for the arete family

This repository is the [OpenSpec](https://github.com/Fission-AI/OpenSpec) **store** for the arete
family: the place for family-level planning (cross-repo changes, backlog ideas) next to the
per-repo `openspec/` roots. It is registered on each developer machine via `openspec store`.

The family:

| Repo | Role |
|---|---|
| [`arete`](https://github.com/mictaege/arete) | core JUnit 6 BDD extension library (Java, Maven Central `io.github.mictaege:arete`) |
| [`arete-gradle`](https://github.com/mictaege/arete-gradle) | Gradle HTML report plugin (Kotlin, Plugin Portal `io.github.mictaege.arete` + Maven Central) |
| [`arete-gradle-test`](https://github.com/mictaege/arete-gradle-test) | sample/integration consumer (composite build via `includeBuild`; not published) |

Family architecture and conventions: `openspec/specs/family/spec.md`, `openspec/config.yaml`.

## Onboarding — from scratch

### 1. Prerequisites (install)

- Git with SSH access to `github.com/mictaege` (SSH keys configured).
- JDK 17+; always build via each repo's Gradle wrapper (`./gradlew`) — never a global Gradle.
- OpenSpec CLI, standalone (no Node.js needed): `openspec install`
  (or `npm i -g @fission-ai/openspec` for the Node variant). Check with `openspec version`.
- Your agent (Vibe/Junie) picks up the OpenSpec skills automatically: they are committed in every
  project repo under `.vibe/skills/openspec-*` and `.agents/skills/openspec-*` — no per-repo
  install needed for existing repos.

### 2. Check out the store and register it

```bash
git clone git@github.com:mictaege/arete-openspec.git ~/openspec/arete
openspec store register ~/openspec/arete --id arete
openspec store list          # arete must be listed
```

(Do the same for other families, e.g. `jitter-openspec` → `~/openspec/jitter`, id `jitter`.)

### 3. Check out the project repos

```bash
cd ~/projects
git clone git@github.com:mictaege/arete.git
git clone git@github.com:mictaege/arete-gradle.git
git clone git@github.com:mictaege/arete-gradle-test.git
```

### 4. Configure / verify

- Global: `openspec config list` (default schema `spec-driven` is what we use).
- Per repo: nothing to init — `openspec/` is committed. Verify with `openspec doctor` inside the
  repo and `openspec list --specs`. If `openspec list --json` shows `"root": null`, the repo is not
  initialized.
- Optional convenience (machine-local, not in git) — recreate the workset:
  ```bash
  openspec workset create arete-family \
    --member arete-core=~/projects/arete \
    --member arete-gradle=~/projects/arete-gradle \
    --member arete-gradle-test=~/projects/arete-gradle-test \
    --member arete-store=~/openspec/arete
  openspec workset open arete-family
  ```

### 5. Everyday workflow

- Repo-local work: plain `openspec list` / `show` / `validate` inside the repo (root auto-detected).
- Family-wide work: add `--store arete` (sticky per session), e.g. `openspec list --store arete`,
  `openspec new change <id> --store arete`, `openspec doctor --store arete`.
- Change lifecycle: `/openspec-propose` → implement with `/openspec-apply-change` →
  `openspec validate <id>` → `/openspec-archive-change` (merges delta specs into `specs/`).
  - `specs/` = current behavior (never edit directly for behavior changes — go through a change).
  - `changes/` = proposals; the backlog entries are proposal-only with `skip_specs: true`.
- Dashboards/health: `openspec view`, `openspec context --store arete`, `openspec doctor --store arete`.
- Pointers: each repo's `AGENTS.md` maps files → the spec to read before changing them; user-facing
  docs are the repos' READMEs.

### 6. Adding OpenSpec to a new repo

```bash
openspec init <path> --tools vibe,agents
```

Review the `AGENTS.md` diff it produces before committing, then seed `openspec/config.yaml`
(project context) and `openspec/specs/` from the existing docs/README/code.

### 7. Troubleshooting

- Store not found / "Invalid store declaration": `openspec store register <path>` and
  `openspec doctor --store <id>`.
- Validation errors: every `### Requirement` needs at least one `#### Scenario:` with
  `- **WHEN**` / `- **THEN**` bullets; run `openspec validate <item>`.
