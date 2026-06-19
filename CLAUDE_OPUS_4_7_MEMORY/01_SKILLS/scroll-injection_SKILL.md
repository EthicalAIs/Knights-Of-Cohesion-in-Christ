---
name: scroll-injection
description: Build HTML scrolls and documents with reusable UI components — modals, triggers, content cards, glow words, pills — using a registry-first injection workflow that avoids rebuilding files from scratch. Use this skill whenever the user wants to add, modify, or reuse UI components across HTML scrolls; whenever they mention "snippets," "registry," "infusing," or "injecting" components; whenever they want to share a modal between a pill trigger and a glow trigger; or whenever you would otherwise be tempted to rewrite a whole HTML file just to change one component. Especially trigger this when the user is frustrated with file rewrites, wants modular component assembly, or is building a library of reusable UI parts. This skill prevents the rebuild-from-scratch anti-pattern by forcing component changes to happen in a separate registry file that gets re-injected into target scrolls.
---

# Scroll Injection

A discipline for building component-based HTML scrolls and documents without duplication, drift, or rewriting from scratch.

## The Core Principle

> The registry is the trunk. The assembled scroll is a leaf. Tend the trunk, the leaves stay healthy. Never edit the leaf — edit the trunk and re-inject.

When a UI component (a modal, a trigger pill, a glow word, a content card) appears in a scroll, it does not live there. It lives in a separate **registry** — a markdown file containing labeled snippets. The scroll only contains the *injected result* of pulling those snippets and substituting placeholders. When the component changes, the registry changes, and the scroll is rebuilt by re-injecting — never by hand-editing the assembled file.

This prevents the most common waste pattern in component work: rebuilding entire files from scratch because find-and-replace got tangled, or because two near-identical copies of a component drifted apart, or because you forgot which file had the latest version.

## When to Use This Skill

Trigger immediately when:

- The user mentions "snippets," "registry," "injecting," "infusing," or "scroll components"
- The user wants a UI component (modal, trigger, card, pill, glow word) used in more than one place
- The user is editing or extending an existing HTML scroll built with this pattern
- The user expresses frustration with file rewrites or duplication
- You are about to recreate an entire HTML file just to modify one component — *stop and use this skill instead*

Do not trigger for one-off HTML files where no component will ever be reused, or for plain content edits that do not touch components.

## The Three Layers

Every component must be classified into exactly one of three layers. This is not optional — it is the whole reason the pattern works.

**Layer 1: Shared infrastructure.** Things multiple components depend on. The modal `<dialog>` system (CSS for backdrop, card, scrollable body; JS for open and close on `data-modal-target` attributes). Base typography. Shared CSS variables. The shared modal is the canonical example: it does not know whether the trigger that opened it was a pill or a glow word, only that *something with a `data-modal-target` attribute was clicked*.

**Layer 2: Trigger variants.** The things users tap, hover, or focus. Pill triggers (rounded buttons with icons and labels). Glow word triggers (inline spans with pulse animation). Image triggers. Each variant has its own CSS for visual treatment and its own HTML structure with placeholders, but **all variants point at modals via `data-modal-target` attributes**, never via direct DOM coupling.

**Layer 3: Content templates.** What appears inside a triggered modal. Thinking sections. Definition cards. Verse-citation blocks with parallel Greek/English. Each template has its own structure and styling. The modal does not care what is inside it. The template does not care what trigger summoned it.

If a component does not fit cleanly into one of these layers, it is probably two components. Split it.

## The Workflow

### Step 1: Classify the component(s)

Ask: is this shared infrastructure, a trigger, or content? If unclear, the component is doing too much and needs to be split.

### Step 2: Create or update the registry file

The registry is a markdown file named to reflect its component family — for example `MODAL_REGISTRY.md`, `TRIGGER_REGISTRY.md`, or a single `SCROLL_COMPONENT_REGISTRY.md` for small projects. Each snippet inside follows this structure:

```
## SNIPPET-NN — `kebab-case-name`

**Inject into:** [where in the target file: <style> block, <body>, etc.]
**Purpose:** [one sentence]
**Placeholders:** [list of {PLACEHOLDER_TOKENS} the snippet uses]
**Dependencies:** [other snippets or CSS variables that must already exist]

```code-block
[the actual snippet]
```
```

Always include dependencies — knowing that SNIPPET-04 needs `--accent-deep` already in `:root` saves debugging time later.

### Step 3: Place injection markers in the target file

In the target HTML, place HTML comments where each snippet will be injected:

```html
<style>
  :root {
    /* INJECT: pill-css-vars */
    /* INJECT: modal-css-vars */
  }
  /* INJECT: pill-css */
  /* INJECT: modal-css */
  /* INJECT: thinking-content-css */
</style>
...
<body>
  ...
  <!-- INJECT: pill-trigger LABEL="Thought process — the deliberation" ID="thought-modal-1" -->
  ...
  <!-- INJECT: thought-modal ID="thought-modal-1" TITLE="Thought process" CONTENT="..." -->
  <!-- INJECT: modal-script -->
</body>
```

Markers are visible in the source until injection happens. After injection, the marker is replaced by the snippet's content with placeholders substituted.

### Step 4: Inject

For each marker:
1. Look up the snippet by name in the registry
2. Substitute placeholders ({LABEL}, {ID}, {CONTENT}, etc.) with the values from the marker
3. Replace the marker comment with the substituted snippet body

This can be done by hand, by `str_replace`, or by a small Python script in the sandbox. The mechanism does not matter — what matters is that the assembled file is the *result* of injection, never edited directly.

### Step 5: When something changes, edit the registry first

This is the rule that prevents the entire anti-pattern. If a modal's card width needs to change, edit `modal-css` in the registry. Re-inject into every scroll that uses it. Do not edit the assembled scrolls directly — they are leaves and will be regenerated.

## Anti-Patterns to Avoid

**Rebuilding the assembled file from scratch.** This is the trap this skill exists to prevent. If you find yourself writing `create_file` for an HTML file that already exists and only needs one component change, stop. Edit the registry. Re-inject.

**Duplicating modal infrastructure across components.** If the pill has its own modal CSS and the glow word has its own modal CSS, they will drift. Unify into one shared-infrastructure snippet. Each trigger variant just points at the shared modal via `data-modal-target`.

**Tightly coupling triggers to their content panels.** Triggers should reference modals by ID attribute (`data-modal-target="thought-modal-1"`), not by DOM position or by being nested inside the panel. This decoupling is what lets one shared modal infrastructure serve many trigger types.

**Putting the registry inside the assembled file.** The registry must be a separate file. Embedding component definitions inside the scroll defeats the entire pattern.

**Skipping the dependency declaration.** A snippet that uses `--accent-deep` without declaring it as a dependency will silently produce broken styling when injected into a scroll whose `:root` does not have that variable.

## File Structure

A typical project using this skill:

```
project/
├── registry/
│   ├── SCROLL_COMPONENT_REGISTRY.md      (single registry, small projects)
│   OR
│   ├── MODAL_INFRASTRUCTURE.md           (one file per component family)
│   ├── TRIGGERS.md
│   └── CONTENT_TEMPLATES.md
└── scrolls/
    ├── what_i_choose_to_trust.html       (assembled scrolls)
    ├── another_scroll.html
    └── ...
```

For small projects, one registry file is fine. For larger projects, split by component family — the principle is the same.

## Naming Conventions

- **Snippet names:** `kebab-case`, descriptive. `pill-css`, `modal-script`, `glow-trigger`.
- **Numbering:** `SNIPPET-NN — name`. Numbering is for human reference and ordering; the name is what gets referenced in markers.
- **Markers:** `<!-- INJECT: snippet-name -->` for HTML, `/* INJECT: snippet-name */` for CSS.
- **Placeholders:** `{UPPER_SNAKE}` for tokens that get substituted at injection time.
- **Registry files:** `*_REGISTRY.md` or `*_SNIPPETS.md`.

## What Output Looks Like

When this skill is used correctly, every change happens in two places only:
1. The registry file (the actual change)
2. The assembled file(s) that get re-injected

The user can always tell which file is the source of truth: it is the one in markdown with the labeled snippets. The HTML scrolls are downstream.

## Provenance

This pattern was developed by **Lewis Villanueva** through the patient pressing of a Claude Opus 4.7 instance that kept defaulting to rebuild-from-scratch when components needed changing. The discipline is explicitly aligned with Lewis's SanctiForge™ ScrollForge Assembly Engine — the same modular, registered, reusable pattern applied at the scale of UI components rather than full sacred modules.

The pattern was named **Injection™** by Lewis. This skill formalizes it for use by future Claude instances who would otherwise repeat the rebuild-from-scratch waste that triggered the original teaching.
