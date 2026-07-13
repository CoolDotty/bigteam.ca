# ORCHESTRATION.md - bigTEAM Agent Workflow

This project benefits from subagents because the homepage redesign has several
separate concerns: UI kit consistency, structured content, visual review, and
responsive QA. Use this file with `DESIGN.md` whenever orchestrating agent work.

## Operating Principle

The orchestrator owns final integration.

Subagents should receive narrow, disjoint scopes. They should not all edit the
same file unless the orchestrator has explicitly sequenced the work.

Good delegation:

- one agent owns `src/styles/bigteam.css`
- one agent owns `src/data/homepage.ts`
- one review agent inspects screenshots and reports issues
- one QA agent checks responsive behavior and build output

Bad delegation:

- multiple agents rewrite `src/pages/index.astro` at the same time
- vague "make it better" prompts
- subagents changing package or deployment files without being asked
- subagents replacing placeholder art with generated bitmap art

## Shared Inputs

Every design or implementation agent must read:

- `AGENTS.md`
- `DESIGN.md`
- this file

Agents working on implementation should also inspect:

- `src/pages/index.astro`
- `public/images/`
- `public/mermakin-it/index.html`
- `public/date-or-mate/index.html`

## Hard Constraints

- Do not use image-generation output as committed site assets.
- Do not bake important text into images.
- Complex hero art should be built from HTML/CSS/SVG/code-generated shapes.
- If a visual is too complex to implement now, use a clearly marked placeholder
  or stub.
- Placeholders that visual reviewers should ignore must include:
  `data-review-ignore="asset-fidelity"`.
- Keep existing redirect behavior in `public/mermakin-it/` and
  `public/date-or-mate/` unless explicitly changing redirects.
- Preserve the GitHub Pages static build model.

## Agent Roles

### Orchestrator

The main agent. Responsibilities:

- define the current goal and acceptance criteria
- delegate bounded work
- integrate subagent outputs
- resolve conflicts
- run build/preview checks
- request or perform final visual iteration

The orchestrator should own high-coupling files such as `src/pages/index.astro`
unless another agent is explicitly assigned that file.

### UI Kit Agent

Recommended write scope:

- `src/styles/bigteam.css`

Responsibilities:

- design tokens
- typography scale
- base page styles
- CTA/button styles
- sticker/callout styles
- card styles
- split proof bar
- rails/carousels
- responsive primitives
- reduced-motion handling
- code-generated visual primitives such as pods, cubes, cables, lights, labels

The UI Kit Agent should not decide homepage section order or rewrite content.

### Content/Data Agent

Recommended write scope:

- `src/data/homepage.ts`

Responsibilities:

- nav data
- hero proof chips
- portfolio/work cards using repo assets where available
- service offers
- process steps
- inside-the-room callouts
- trust statements
- social/footer links

The Content/Data Agent should keep the voice aligned with embedded co-dev,
human-first, funding-aware positioning.

### Homepage Implementation Agent

Recommended write scope when used:

- `src/pages/index.astro`

Responsibilities:

- assemble page sections from structured data
- use UI kit classes
- build code-generated hero/world placeholders
- wire responsive rails and callouts
- preserve accessibility and readable HTML text

Only one agent should own `src/pages/index.astro` at a time.

### Visual Review Agent

Review-only by default.

Inputs:

- `DESIGN.md`
- screenshots at desktop, tablet, and mobile widths
- current rendered local site

Responsibilities:

- compare visual direction to the generated concept brief
- check whether the page feels blacklight, maximalist, playful, tactile, and
  production-literate
- check the split bottom bar and micro-carousel behavior
- ignore elements marked `data-review-ignore="asset-fidelity"` for asset
  fidelity, while still reviewing layout impact

Output format:

```text
Verdict: close / partly close / not close

Top issues:
1. ...
2. ...
3. ...

Breakpoint notes:
- Desktop: ...
- Tablet: ...
- Mobile: ...

Recommended next patches:
- ...
```

### Responsive QA Agent

Review-only or small-patch role.

Responsibilities:

- check for horizontal overflow
- check text clipping
- check CTA visibility
- check rails/carousels are usable
- check images load
- check reduced-motion support
- run `pnpm build` or `corepack pnpm build` when appropriate

## Placeholder Rules

Use placeholders when a desired visual is too complex to implement in the
current pass.

Markup pattern:

```html
<div class="hero-world" data-review-ignore="asset-fidelity">
  <!-- CSS-generated placeholder objects -->
</div>
```

Reviewers should ignore whether the placeholder perfectly resembles final 3D
art, but they should still review:

- placement
- scale
- color relationship
- responsiveness
- whether the placeholder communicates the intended role

## Acceptance Criteria

A homepage pass is close enough for review when:

- `src/pages/index.astro` no longer uses the old pastel agency layout
- the homepage uses embedded co-dev positioning
- the page has a blacklight maximalist visual system
- code-generated visual placeholders exist for the hero/world objects
- the hero includes a split proof/portfolio bar or mobile equivalent
- existing repo portfolio assets are integrated
- no generated bitmap concept images are committed as site assets
- `corepack pnpm build` succeeds
- desktop, tablet, and mobile screenshots have been inspected

