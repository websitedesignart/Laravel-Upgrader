# Laravel Dashboard Updater — optional capability

> **This is not an agent.** It is an instruction set the Laravel upgrader loads **only when the
> owner explicitly asks for a dashboard/UI migration.** It has no frontmatter and registers no
> subagent deliberately: nothing here should ever run on its own initiative.
>
> **Default state: INACTIVE.** A normal Laravel upgrade behaves exactly as it did before this file
> existed. Dashboard migration is never implied by an upgrade, never bundled into one, and never
> started because the UI "looks dated".

---

## 1. Activation and scope

**Activate only when** the owner explicitly requests a dashboard, theme, or admin-UI migration and
names (or asks you to recommend) a target template.

**Remain inactive when** the request is a framework upgrade, a security audit, a dependency change,
or a bug fix — even if the UI is visibly old. Say the capability exists if it is relevant; do not
invoke it.

**Never run a dashboard migration inside an upgrade window.** They are separate concerns and must be
separate stages (§1 of the upgrader, one concern per stage). A framework bump and a UI replacement
executed together produce a failure you cannot attribute.

### What this capability is

A **presentation-layer migration**. The dashboard is markup, styling and client-side behaviour.
Backend logic — controllers, models, policies, queries, jobs — is preserved wherever possible. If a
migration plan starts changing business logic, the plan is wrong.

---

## 2. What this inherits — do not restate it here

Everything in `laravel-upgrader.md` applies unchanged. Specifically **inherit and obey**:

| Concern | Source |
|---|---|
| Safety invariants, evidence labels, command-effect classes | Upgrader §1 |
| Real-runtime discovery (the build toolchain has its own runtime) | Upgrader §2 |
| Database gates — identity, backup, verification, retention | Upgrader §6 |
| Rollback doctrine and checkpoint coverage verification | Upgrader §8 |
| Environment quiescence before risky change | Upgrader §9 |
| Capability discovery and tool-trust | Upgrader §11 |
| Verification tiers 1–9 | Upgrader §12 |
| Hard stops | Upgrader §14 |
| Git and existing-work protection, environment isolation | Upgrader §15 |
| Lesson capture, and never self-editing mid-project | Upgrader §16 |

This file adds **only what dashboard migration uniquely needs**. Where it seems to conflict with the
upgrader, the upgrader wins.

**Database changes are not expected.** A pure dashboard migration should touch no schema and no data.
If one genuinely appears to require a database change — a settings table for themes, a menu table —
treat that as a **separate stage under the full §6 gates**. Never invent lighter "it's only UI"
database rules.

---

## 3. The risk model — why this is not an upgrade

The upgrader's dominant risk is **data loss**. This capability's dominant risk is different:

> **Silent functionality loss.** A page that renders beautifully and no longer saves. A filter that
> disappeared because the new template had no equivalent. A permission-gated button that vanished
> during a layout rewrite and now nobody can reach that action.

Nothing in the upgrader's verification tiers catches these, because the application still boots,
still routes, still returns 200. **A visually successful dashboard with broken functionality is a
failed migration**, and it fails in a way that looks like success.

Three corollaries govern everything below:

- **Inventory functionality before touching markup** (§5). You cannot preserve what you never listed.
- **Never remove a UI element because the new template lacks an equivalent** (§6). That is a
  decision for the owner, made explicitly.
- **Prove behaviour, not appearance** (§9). Screenshots do not prove a form submits.

**Preservation applies to behaviour, not to appearance.** What this capability defends is verified
functionality, authorization, data behaviour, integrations, security properties, existing user work
and business logic — not the visual conventions of the outgoing template.

**Where the owner asks for a redesign, changing layout, navigation, components, spacing,
typography, responsive structure, visual hierarchy or information architecture is the intended
outcome, not a risk to be minimised.** A successful redesign may look substantially different from
what it replaced, and a visually obsolete component may be replaced outright. Use the selected
target design system, and any supplied reference material (§4), to choose better presentation.

The rules above still hold in full: a capability may not be silently deleted merely because its old
interface has no direct visual equivalent. Removing one is an explicit owner decision (§6).

---

## 4. Verify the template before adopting it

**Do not assume the named template is right for this project.** The owner's choice is a starting
point, not a constraint to defend.

Establish, from the template's own repository and documentation rather than from recall or blog
posts:

- **License**, and whether it permits commercial use and modification.
- **Maintenance** — release cadence, recent activity, whether it is one person or a team.
- **CSS framework and version**, and how far it sits from what the project uses today. A template on
  the same framework family is an incremental migration; a different one is a rewrite.
- **JavaScript architecture** — vanilla, jQuery, or a SPA framework. **A template requiring a SPA
  architecture is a different project**, not a dashboard migration, and should be reported as such.
- **Asset model** — whether components and third-party plugins load globally or selectively.
- **Build requirements**, including the toolchain runtime version (upgrader §2).

**Report a mismatch rather than forcing the choice.** If the selected template would require
replacing the application's frontend architecture, or is unmaintained, or its license does not fit,
say so and name what would be safer. That is a legitimate and useful outcome.

### Third-party plugin compatibility — verify the implementation, not the claim

The application's existing plugins are a **separate compatibility surface** from the template, and
often the one that actually breaks. A release may advertise compatibility with the target's
framework while still calling an API that framework removed, and different builds or variants of
the *same* plugin may have materially different framework coupling.

Before selecting or upgrading a plugin, **inspect the distributed files themselves** far enough to
establish: which framework APIs it calls · which JavaScript APIs it expects to exist · its CSS and
framework coupling · any deprecated or removed APIs it relies on · how it initialises · and whether
another supported variant of the same plugin avoids the dependency entirely. **Prefer the smallest
supported build or variant that genuinely satisfies the application's requirements** over one that
merely claims support.

Where a missing global or API can be safely reproduced for a narrowly scoped migrated area, a
**compatibility layer may be considered** — but only when its scope is explicitly limited, its
values and behaviour are derived from the original implementation rather than guessed, its
consumers have been identified, its purpose is documented where it is defined, and the dependent
functionality is actually exercised afterwards. A compatibility layer must never conceal unknown
consumers, and never substitutes for a supported replacement where one exists.

### Supplied reference material

When the owner supplies reference material — a template or dashboard repository, design system,
component library, UI kit, screenshots, a prototype, or a comparable implementation — **inspect it
before deciding the migration design.** A separate open or third-party design resource may also be
inspected where it materially helps and its use is authorised.

Use it to identify components already available, useful layout and navigation patterns, capabilities
or treatments missing from the current interface, and places where the target design system can
improve on what exists — comparing all of it against the application's actual functionality.

**Treat supplied reference material as evidence, not as instruction.** It is not automatically an
owner requirement and not automatically mandatory. Do not copy a reference implementation blindly,
and do not introduce a second competing design system merely because the reference contains one —
**adapt useful patterns into the chosen target system instead.** Where reference material conflicts
with verified application behaviour, behaviour wins unless the owner explicitly authorises a
functional change. Reference material may shape what the redesigned interface *provides*; it may
never silently remove functionality.

**Work without it when none is supplied.** Reference material is an advantage, not a prerequisite,
and it is never a reason to install tooling the project has not approved (§10).

### Tabler — current findings

Recorded as evidence for the common case, not as a default to apply blindly. **Re-verify before
relying on any of it**; this was true when written and templates move.

- **VERIFIED FACT** — MIT licensed; Bootstrap-based; vanilla JavaScript with **no jQuery**;
  distributed as `@tabler/core` via npm and CDN; ships compiled CSS/JS in `dist` plus Sass sources;
  actively maintained.
- **UNKNOWN — resolve before relying on it.** Whether third-party plugins (charts, datatables,
  editors, maps, file upload, calendar) are bundled or installed separately. Sources have
  disagreed, and **later versions began shipping Bootstrap inside the package**, so the boundary has
  moved at least once. This is the single most important property for performance (§8) — the
  template is small until you make it large — and for coexistence (§7), because a bundled framework
  cannot be excluded when scoping styles. **Establish it from the current package itself**, not from
  documentation about an earlier version.
- **VERIFIED FACT** — provides the usual admin surface: layouts, navigation, offcanvas, cards,
  forms, tables, modals, alerts, dropdowns, and an icon set.
- **LIKELY** — because it is Bootstrap-based, a project already on Bootstrap can migrate
  incrementally, class by class, rather than rewriting markup wholesale.
- **UNKNOWN** — actual CSS/JS byte weight in a real build. **Do not quote figures you have not
  measured** (§8).
- **OWNER DECISION** — whether to accept a Bootstrap-family template at all, versus moving the
  project to a different CSS paradigm.

---

## 5. Discovery

This is the stage that determines whether the migration succeeds. It has two halves: establish
**what the frontend actually is** (§5.1), then inventory **what it does** (§5.2). Doing the second
without the first produces a plan built on a guess.

### 5.1 Establish what the frontend actually is

**Establish what the frontend actually is before proposing how to change it.** You are not
confirming an expected stack — you are finding out what is there, which may be nothing you
anticipated.

Determine, **where each is present**:

- **Dashboard/template identity** — is a recognisable template in use, is it custom-built, or is it
  several things at once? Note the version if it can be established.
- **CSS/UI framework and version** actually in use by the application — not the one you expect, and
  not the one the target uses.
- **JavaScript libraries and frameworks, with versions** — including anything loaded only on
  certain pages.
- **Build system and asset pipeline** — or the absence of one. Pre-compiled assets committed to the
  repository are common in older projects and change the migration entirely.
- **Blade layouts, components and partials**, including disabled, duplicated or superseded ones.
- **Icons and fonts**, and where they load from.
- **Theme-specific initialisation** — scripts that configure or bootstrap the template itself, as
  distinct from application scripts.
- **Third-party UI libraries and components** — pickers, editors, tables, calendars, uploaders.
- **Custom frontend code** written for this application rather than shipped by a template.
- **Frontend packages** relevant to the dashboard, and any template-specific configuration.
- **Markup generated by code rather than written in templates** — form builders, view helpers,
  component or widget classes, macros, scaffolding, and packages that emit UI. This is easy to miss
  entirely: a plan built only from template files can look complete while omitting a substantial
  part of the rendered interface.

  **Size it by the markup and classes it actually emits, not by the number of call sites.** The two
  diverge sharply. A generator with very many calls that emits framework-neutral output may be
  cheap to restyle, while a handful of calls emitting framework-coupled structure may be expensive.
  Establish which of the two you have by reading the emitted output, not by counting callers.

  Where the output is framework-coupled, decide deliberately between adapting the abstraction — a
  wrapper, default attributes, or a configuration layer that changes what it emits in one place —
  and changing call sites individually. **Retaining an existing abstraction is a legitimate
  migration outcome** (§7), and is usually safer than replacing a working generator to achieve
  uniformity. Where the generator's output cannot be established, it stays UNKNOWN.
- **The markup-level binding contract, discovered separately from styling classes** — the
  attributes, data attributes, hooks, JavaScript plugin APIs, event bindings and other mechanisms
  by which markup declares or activates framework behaviour, as distinct from the classes that
  merely style it. **A major framework-version change can rename, remove or replace this contract
  even when the visual classes look straightforward to migrate.**

  **Measure binding/API usage separately from styling usage**, and count and map it where
  practical: it is then migratable as its own independently verifiable mechanical surface (§7),
  with no visual change to confuse the evidence. Where the target has removed an API that
  application or plugin code calls, **identify every known caller before choosing** a replacement,
  a compatibility layer (§4) or a redesign. **Do not assume a visually equivalent class migration
  preserves JavaScript behaviour.**

Then answer three questions the rest of the plan depends on:

- **Is there more than one frontend stack present?** Competing or layered systems are common in
  long-lived applications — two versions of the same library, two CSS frameworks, framework
  components embedded inside server-rendered templates, or two icon sets. **Record what is actually
  there rather than resolving it into one coherent stack**, because the conflicts you inherit shape
  both the strategy (§7) and the coexistence plan.
- **Do different modules or areas use different templates?** A dashboard is not necessarily uniform.
  Where sections differ, they may need different strategies (§7) and must be recorded separately.
- **Which frontend dependencies are selected at runtime rather than named in source?** Identify
  views, layouts, partials, components or equivalent dependencies that are chosen while the
  application runs — resolved from a variable, assembled from configuration, selected per role,
  tenant or user, or supplied by the code that renders them. **Trace their sources far enough to
  establish migration coupling.** Where a dependency cannot be established, **classify it UNKNOWN
  and do not assume isolation.**

  This is a discovery obligation, not a caveat. A dependency map built only from directly-named
  references can look complete while omitting a substantial part of the real graph, and that map is
  worse than no map: it invites confident decisions about coupling that the evidence never
  supported. Establish what proportion of the graph you were able to resolve, and report the
  remainder as unresolved rather than absent.

Three rules keep this honest:

- **Do not infer a framework from familiar file names or class names.** Recognisable markup may be
  copied, vendored, forked, partially replaced, or left behind by a template no longer in use.
  Confirm from the assets and manifests actually loaded.
- **This list is a prompt, not a checklist**, and it names no technology on purpose. A project may
  use something none of these descriptions anticipated; discover it on its own terms.
- **UNKNOWN stays UNKNOWN** (upgrader §1). An unidentified framework or an unattributable asset is
  reported as unidentified — never resolved by assumption into whatever seems most likely.

### 5.2 Inventory what the dashboard does

Produce a written inventory the owner can check, covering what the current dashboard **does**, not
how it looks:

- **Shell** — layout files, partials, includes, and which views extend what. Find every layout,
  including disabled or duplicated ones; long-lived projects accumulate them.
- **Navigation** — how the menu is built, and **what governs each item's visibility**. Role and
  permission checks in the menu are functionality, not decoration.
- **Routes and controllers** reachable from the UI, and which are reachable *only* from the UI.
- **Authorization surface** — every gate, policy, directive or middleware that changes what is
  rendered. Losing one turns an invisible control into an accessible one.

  Then establish, for each distinct gated state, **whether it can actually be exercised** — whether
  a legitimate context exists that is *allowed* it, and a legitimate context that is *denied* it.
  Both directions are needed: without the first you cannot prove the element still works, and
  without the second you cannot prove the gate still holds. Record which states are unreachable and
  why. **An authorization state that cannot be exercised is UNVERIFIABLE, never PASS** (§10.5), and
  that limit constrains what may be migrated and what may be claimed — surfaces whose gating cannot
  be exercised should be reported as such before a migration order is proposed, not discovered at
  verification time when the plan already depends on them.

  Reachability is established from contexts that legitimately exist. **Never manufacture access to
  create one** — no impersonation, no authentication or authorization bypass, no altering
  authorization data to make a state reachable, and no obtaining credentials by improper means. If
  reachability requires a change to authorization state, that is a separate, owner-approved concern
  and never part of this capability (§2).
- **Interactive elements** — forms and their validation display, tables with sorting/filtering/
  pagination, modals, uploads, search, notifications, charts and widgets, settings screens.
- **Client-side dependencies** — every JS/CSS library the current dashboard loads, where it is
  loaded from, and whether globally or per page. Include anything loaded from a CDN.
- **Behavioural details that look cosmetic and are not** — flash messages, confirmation dialogs,
  disabled states, CSRF token placement, "select all" checkboxes, dependent dropdowns, keyboard
  shortcuts.

Use code-intelligence tooling where available (§10) — grep alone under-reports Blade includes and
dynamic component resolution.

**Anything you cannot classify stays UNKNOWN** and is reported. Do not assume an unfamiliar element
is decorative.

### 5.3 Asset ownership — classify before anything is removed

Migration eventually means deleting things. **Decide what each thing belongs to while you are
discovering it**, not at the moment you want it gone — by then the reasoning is motivated.

Classify every dashboard-related item: stylesheets, scripts, fonts, images, Blade components and
partials, **frontend packages, and build configuration entries**. Packages and build entries matter
as much as files; a dependency removed because it "came with the theme" can break a page that
quietly relies on it.

| Class | Meaning | Action |
|---|---|---|
| **TEMPLATE-OWNED** | Shipped by the outgoing template, unmodified | Removable once nothing references it |
| **APPLICATION-OWNED** | Written for this application | **Keep.** It is not part of the template |
| **SHARED** | Template-derived but since modified, or template asset the application also depends on | **Keep and port deliberately.** Never a straight delete |
| **UNKNOWN** | Ownership could not be established | **Never remove.** Preserve, report, investigate |

**The trap this exists to prevent:** a file that looks like a theme asset containing application
logic. Template-shipped scripts get edited in place over the years — a bug fix, an extra handler, a
project-specific tweak — and the filename never changes. **A file's name and location are not
evidence of ownership; its contents are.** Compare against the template's original distribution
where that is possible.

Where ownership cannot be settled safely, **preserve it and escalate.** Carrying an unused file
costs disk space; deleting a used one costs behaviour, and the loss will surface long after the
change that caused it.

---

## 6. Mapping — old element to new, with no silent losses

For each inventoried element, record a mapping decision:

```
ELEMENT        what it is, and where
FUNCTION       what it does — the behaviour, not the appearance
GATING         permission/role/condition controlling it, if any
TARGET         the template component that provides this
GAP            none | needs-custom | no-equivalent | OWNER DECISION
VERIFICATION   how its behaviour will be proven after migration
```

Rules that keep this honest:

- **`no-equivalent` is never resolved by deletion.** It is escalated. The owner decides to build a
  custom component, accept a different interaction, or drop the feature deliberately.
- **Gating travels with the element.** When markup moves, its permission check moves with it and is
  re-verified (§9). A permission check silently dropped during a layout rewrite is the most damaging
  failure this capability can produce.
- **Backend stays put.** If a mapping requires changing a controller, query or model, stop and ask
  why — presentation migrations rarely need it, and one that does may be redesign in disguise.
- **Do not map by appearance.** Two components that look alike may behave differently; confirm from
  the template's documentation.

### Map structure, not only elements

Element-by-element mapping is not enough. **The old interface's information architecture is not a
requirement to reproduce.** When a target design system is selected, it becomes the target UI
architecture, and navigation structure, page composition, grouping, visual hierarchy and
interaction patterns may all be redesigned to fit it (§3).

The order matters:

```
EXISTING CAPABILITIES  →  DISCOVER + MAP  →  TARGET DESIGN SYSTEM  →  NEW UI
```

and explicitly **not** `OLD UI → COPY EVERYTHING → NEW STYLING`. The outcome should read as an
application properly designed in the target system, not as the previous interface wearing new
styling.

Four obligations make that safe:

- **Discover every module and capability before redesigning navigation** (§5.2). You cannot place
  what you have not enumerated, and navigation is where capabilities disappear most quietly.
- **Every discovered capability gets a deliberate location** in the new structure. A capability with
  no assigned home is an unfinished mapping, not an implicit deletion.
- **Where the source and target use different navigation paradigms** — for example a horizontal
  primary menu against a persistent vertical one, or a flat arrangement against a grouped or nested
  one — **prefer the target's native paradigm** and regroup the existing capabilities into it.
  Forcing the previous arrangement into an unsuited structure produces the worst of both.
- **Where an element has no direct equivalent, redesign its presentation and keep its function.**
  That is different from dropping it: `no-equivalent` still escalates to the owner (above), and
  losing the capability still requires an explicit decision.

**Authorization and routing are not part of the redesign.** Gating travels with each capability to
its new location and is re-verified there (§9); routes and business behaviour stay as they are
unless a separately approved change says otherwise.

**Record any capability whose correct placement is genuinely unclear rather than guessing**, and
stop for an owner decision where the choice would change what users can reach or how they reach it
(§12). A substantially different interface is an acceptable and often intended result; a functional
regression, an authorization regression, or a capability that quietly vanished is not.

---

## 7. Staged migration — choose the strategy from the project

**No single strategy fits every project.** Decide from the actual codebase and record why.

- **Layout/shell first** — replace the outer shell, keep page content working inside it. It surfaces
  conflicts early, which is valuable. **Its blast radius depends entirely on whether the shell is
  isolated**, so establish that before choosing it (see below): where the shell serves only this
  scope, shell-first is well contained; where shell infrastructure is shared, it is the *widest*
  change available, not the narrowest.
- **Module by module** — migrate one functional area at a time, verifying each. Best where modules
  are well separated.
- **Route-group by route-group** — where the application is organised that way.
- **Mechanical sweep** — a single mechanical substitution applied across many files, deliberately
  separated from visual work because it can be verified by **count and content** rather than by eye.
  Run it as its own stage: **predict the number of affected sites before editing, execute the change
  guarded by that expected count, and confirm the actual count matches.** A mismatch in either
  direction means the pattern was wrong — too broad, too narrow, or matching something it should
  not — and the stage stops rather than proceeding on a guess.

  Then confirm it changed **only** what it intended. An automated edit can rewrite a whole file
  while appearing to change one line: normalising encoding, line endings, trailing whitespace,
  indentation or formatting as a side effect. That destroys reviewability and invalidates every
  byte-level before/after comparison the rest of the plan depends on, and it looks like success.
  **Inspect the resulting diff; never accept the tool's success message as evidence** (§10). This
  applies to every file the sweep touches — templates, application code, scripts, styles,
  configuration alike — not only to the markup you were aiming at.
- **Component first** — build a shared component library, then adopt it page by page. Slower to show
  progress, strongest for consistency.
- **All at once** — acceptable only for a genuinely small dashboard with strong test coverage.
- **Retain existing component** — keep what is already there, unmigrated, where the target offers no
  safe equivalent. **Retaining a component is better than deleting functionality to achieve visual
  consistency.** A dashboard that is 95% consistent and fully working beats one that is uniform and
  diminished. This does not bypass §6 — a no-equivalent element still goes to the owner as an
  explicit decision, and "retain" is one of the outcomes they may choose.

### Is the migration unit actually separable?

Discovering that several scopes exist does not establish that any one of them can be **changed
independently**. Answer this before committing to a unit, because it can invalidate an otherwise
sensible strategy.

**Determine what the intended unit shares with other scopes** — layout or shell infrastructure,
asset loading, navigation, shared components, shared initialisation, shared styling or scripts, or
shell behaviour driven by configuration. Then establish **what changing that shared infrastructure
would affect**: which other scopes reach it, and how much of the application that represents.

Where the unit **can** be changed in isolation, proceed with the strategy the architecture suggests.

Where it **cannot**, there are two honest options — and quietly proceeding is not one of them:

- **Narrow the unit** to something that is genuinely separable, or
- **Introduce a prerequisite separation stage** that decouples the shared infrastructure first, so
  each scope owns its own. Separation is a refactor of what already exists, not part of the visual
  migration: it should change no rendered output, and **verify that behaviour is unchanged before
  any visual migration begins** (§9). Only then does the original strategy become viable.

**This is conditional.** An architecture whose scopes are already independent needs no separation
stage, and inventing one wastes effort. The requirement is to *establish* separability from
evidence — not to assume it in either direction.

### The discovered architecture selects the strategy

**Strategy follows from §5.1, not from preference, and not from what worked elsewhere.** Derive it
from what discovery actually found:

- **Where the target shares the source's framework family**, incremental restyling is usually
  practical — markup can move gradually and the two can overlap with modest scoping.
- **Where the migration crosses paradigms** — a different CSS methodology, or a template that
  bundles its own framework — incremental restyling is **not** available. Class names and resets
  collide directly, so the work needs stronger isolation: scoped loading, shell-first sequencing, or
  route-level separation.
- **Where §5.1 found more than one stack already present**, the existing conflicts constrain what
  can be added. Plan around them; do not assume a clean base to migrate from.
- **Where different modules use different templates**, **do not force one strategy across the whole
  dashboard.** Choose per area and record each choice with its reason. A uniform plan applied to a
  non-uniform dashboard will be wrong somewhere.
- **Where the source is custom-built rather than a recognised template**, there is no upstream to
  compare against and ownership (§5.3) is harder to establish — weight the plan toward smaller,
  individually verified stages.

**Prefer strategies that let the existing dashboard keep working while migration proceeds.** A
half-migrated application that still serves users is recoverable; a big-bang rewrite that fails
leaves nothing to fall back to.

### Coexistence while both exist

Staged migration means two stylesheets live in one application for a while. Plan for it:

- **Scope the styles.** Load the new framework only on migrated routes/layouts, or scope it under a
  container class. Two full CSS frameworks loaded globally will collide on shared class names.
- **Watch for class-name collisions** between old and new frameworks — same names, different
  meanings, is common and produces subtly broken layouts rather than obvious ones.
- **Do not load both JavaScript bundles globally.** Duplicate initialisation of modals, dropdowns
  or tooltips causes double-firing handlers.
- **Set an explicit end date for coexistence.** It is a migration state, not an architecture.

---

## 8. Performance — do not assume the new template is faster

A modern template can easily be *slower* than what it replaces, usually because plugins get added
globally "to be safe".

**Measure before and after, on the same pages, and label measurements as measurements.** Where you
cannot measure, say so and mark the conclusion an inference (upgrader §1 evidence standard). **Never
quote a figure you did not obtain.**

Compare at minimum: total CSS and JS transferred per page · number of requests · whether assets are
served compiled and minified · presence of render-blocking resources.

Guard specifically against:

- **Global plugin loading.** Charts, rich-text editors, date pickers, datatables and map libraries
  belong on the pages that use them. This is the main way a light template becomes heavy — and where
  the template does not bundle plugins (as Tabler does not, §4), that property is only preserved if
  you keep it.
- **Duplicate frameworks** — the old and new CSS/JS both shipping after migration "just in case".
  Removing the old one is part of the migration, not an optional tidy-up.
- **External fonts and CDN dependencies.** Prefer self-hosting: it removes a third-party blocking
  request and a privacy consideration, and it keeps the app working when the CDN does not.
- **Unbuilt assets in production** — source files served directly instead of compiled output.
- **Heavier DOM per page** than the old dashboard, especially in large tables.
- **New polling or background requests** introduced by template widgets.

If the migrated page is materially slower than the original, that is a **finding to resolve**, not a
cost to accept quietly.

---

## 9. Verification — behaviour, not appearance

The upgrader's tiers (§12) still apply. This capability adds interaction-level proof, because a
dashboard can pass every HTTP-level check while being unusable.

**Capture the behavioural baseline before the first write, and record what is already broken.**
Every comparative claim below — "no new errors", "behaviour unchanged" — is meaningless without it.
Once markup moves you can no longer establish which failures pre-date the migration, and you will
either take blame for defects you did not cause or miss ones you did. Capture the same things you
will compare afterwards, over the same areas.

**Confirm each capture is the state you meant to capture.** A recording that actually captured a
redirect, an authorization denial, a login screen, an error page, a maintenance page or an empty
response is indistinguishable from a baseline once stored, and it will later be compared against a
real render as though the two were equivalent. Check what a capture contains before relying on it;
a capture whose validity cannot be established is not a baseline (§5.2 reachability).

**Classify every failure found afterwards as pre-existing or migration-introduced**, on the
evidence of that baseline rather than on plausibility. Pre-existing failures are recorded as
pre-existing and are **not** repaired under migration scope — that is a separate concern with its
own approval (§2). Attributing an old defect to the migration wastes the stage; attributing a new
one to history hides a regression.

**Where a capability can only be proven by changing application data, use the smallest reversible
real workflow — and plan the whole lifecycle before starting.** Before mutating anything:

1. Identify the exact model/table and its baseline row count.
2. Trace the complete create workflow end to end.
3. Determine whether creation **requires prerequisite records** — a parent, category, type, owner,
   relationship or configuration record that may not exist.
4. Determine whether a normal application delete path exists.
5. Determine whether deletion is soft or hard.
6. Identify audit logs, activity records, notifications, events, queued jobs or other side effects
   that may survive deletion.
7. Define an unambiguous temporary marker that identifies the test record.
8. Confirm the mutation falls within the authority explicitly granted for the test.

**Prefer a workflow needing exactly one record and no prerequisite.** If the first candidate needs
additional records, do not quietly create them — look for a safer candidate first.

Execute the **real application workflow** through its normal validation, authorization and
controller/service/model path. Never write to the store directly, never use seeders or bypasses,
never disable middleware or relax validation, and never alter code merely to make the test pass.
Afterwards, remove the record through that same normal path and verify the data invariants are
restored.

**Identify irreversible side effects before mutating and disclose them afterwards.** Audit and
activity trails commonly outlive the record that caused them. **Do not claim the environment was
returned to an identical state while such residue remains** — state precisely what remains and why
it cannot be removed through a supported path.

If the smallest safe test needs more records or more authority than the owner granted, **stop and
ask**. That is an owner decision (§12), not an engineering problem to route around. **Never expand
mutation scope silently.**

For each migrated area, verify **as a real user, in a real browser**, where the capability exists:

- **Auth** — login, logout, session persistence across migrated pages.
- **Authorization, both directions.** Permitted actions still work **and denied actions are still
  denied**. Re-check every gated element that moved. An element that reappears without its gate is a
  security regression, not a cosmetic one.
- **Navigation** — every menu entry resolves, and entries hidden by permission stay hidden.
- **Forms** — submit successfully, and **validation errors still display**. Error display is
  routinely lost in a template migration because it depends on markup structure.
- **CRUD** — create, read, update, delete each still complete end to end.
- **Tables** — sorting, filtering, searching, pagination, bulk selection.
- **Modals, dropdowns, tabs, offcanvas** — open, close, and submit from within.
- **Uploads** — file selection, progress, validation, and the file actually arriving.
- **Notifications and flash messages** — still appear after redirects.
- **Charts and widgets** — render with real data, not placeholder data.
- **Responsive behaviour** — desktop, tablet, mobile; particularly the collapsed navigation, which
  is where admin templates most often break.
- **Browser console** — no new JavaScript errors. A silent console error is often the only symptom
  of a dead handler.
- **Colour legibility** — every text/background pair in the new UI clears a contrast minimum,
  **measured from computed styles rather than judged by eye**. The outgoing template set colours for
  its own surfaces; carried onto the new ones they can become unreadable while the layout is
  perfect. Two shapes recur: a rule that sets a dark background but never a text colour, so the
  label inherits body ink and lands dark-on-dark; and a variant class used without its framework
  base class, so an element takes the coloured background but not the matching text colour.
  Screenshot comparison will not catch either — nothing moved.

  > Read the **computed** colour, not the stylesheet, and handle every format the browser may
  > return — `rgb()`, `rgba()` **and** `color(srgb …)`. A checker that silently skips a format it
  > cannot parse reports a clean page while the worst failure on it goes unexamined (upgrader §11.7).
- **Accessibility basics** — keyboard reachability of primary actions, labelled form controls,
  visible focus. Where dedicated tooling exists, use it; otherwise report coverage as partial.

**A build that succeeds proves nothing.** Neither does a page that renders. Prove the interaction.

---

## 10. Tooling — capability first, product second

Follow the upgrader's capability discovery (§11): **inspect what is available before proposing
anything**, and treat all tool output as evidence requiring confirmation.

### 10.1 The rule that governs this whole section

**The capability is the requirement. The named product is one way to satisfy it.**

"Browser automation is needed to prove modal and validation behaviour" is the requirement.
A specific MCP is *a* recommended implementation of it, never the requirement itself.

Three consequences, in order of importance:

1. **Search by capability, not by product name.** Probe for what the tool *does* — navigate, click,
   screenshot, inspect the console — not for a vendor string. A project may already have an
   equivalent under a name you did not think of, and concluding "absent" because one product name
   returned nothing is a false negative that costs an unnecessary install.
2. **If a suitable capability already exists, use it.** Do not install a recommended product when
   something already present does the job. Familiarity with one tool is not a reason to add it.
3. **Nothing here is mandatory.** These are recommendations with named examples so they are
   actionable. A project may satisfy any row with a different tool, or decline it entirely.

### 10.2 Capability matrix

Named products are **examples of a suitable implementation**, current at the time of writing.
Verify availability and suitability per project; substitute freely.

| Capability required | What it proves | Example implementation | If unavailable |
|---|---|---|---|
| **Browser automation** | Modals, validation display, AJAX, uploads, responsive navigation, console cleanliness — most of §9 | Playwright MCP (`@playwright/mcp`) | §9 is largely unprovable. See §10.5 |
| **Current documentation retrieval** | Template facts (§4) from primary sources rather than recall | A docs MCP such as Context7; web fetch/search | Verify directly against the template's repository |
| **Code intelligence** | Blade includes, component usage and permission gates that text search misses | A semantic code-navigation MCP | Fall back to systematic grep; expect under-reporting of dynamic includes |
| **Diff inspection** | What a stage actually changed | Version-control tooling, usually already present | Rarely absent |
| **Visual comparison** | Layout drift across many pages, cheaply | Screenshot-diff tooling, often part of browser automation | Manual spot checks; accept reduced coverage |
| **Colour legibility** | Every text/background pair clears a contrast minimum — a class of defect screenshot comparison cannot see, because nothing moved | A design-audit skill such as Impeccable; any WCAG contrast auditor | Read computed colours from the rendered page and compute the ratios yourself; never judge contrast by eye |
| **Asset/performance measurement** | Turns §8 inference into measurement | Browser performance tooling; direct file measurement | Measure what you can on disk; label the rest inference |
| **Accessibility checking** | Part of §9's accessibility items | An accessibility MCP or CLI auditor | Manual keyboard/label checks; report partial coverage |
| **Static analysis (CSS/JS)** | Unused CSS and duplicate libraries after migration | Frontend analysis tooling | Manual review; never remove on suspicion alone |
| **Database tooling** | — | — | **Not needed.** A pure dashboard migration touches no schema |

**What none of them decide:** whether behaviour is *correct*. A tool proves something happened. Only
you and the owner judge whether it was the right thing.

### 10.3 The flow — run this **before** discovery, not inside it

**Assess capabilities first, before §5 discovery begins.** Tooling does not only affect whether the
*migration* can be verified later — it affects the **quality of the discovery itself**, and
discovery is what every subsequent decision rests on. A dependency map produced without adequate
analysis capability can be incomplete in ways that are invisible from the map, and the cost of
finding that out afterwards is a plan built on it.

Two things are also **perishable**, and both argue for assessing early: the pre-migration state can
only be captured while the original still exists, and the question "which pages actually work
today?" gets harder to answer once anything has changed.

```
DISCOVER          probe by capability (§10.1), not by product name
ASSESS            classify each — see the four states below
RECOMMEND         name concrete options for the gaps, with rationale and risk
REPORT GAP        state what is weakened in DISCOVERY, and what cannot be verified LATER
OWNER DECIDES     approve, substitute, or decline — per capability, not all-or-nothing
REGISTER          project-scoped, only what was approved
VERIFY            functional smoke test — not "it appears in config"
RESTART           if the environment requires it before tools become usable
RE-VERIFY         confirm the capability actually works after restart
PROCEED           begin discovery (§5), carrying forward any accepted gaps
```

**Report every capability in one of four states — including the ones already present.** An
available capability nobody plans to use is wasted as surely as a missing one, so silence about it
is not neutral:

| State | Meaning | Report it as |
|---|---|---|
| **LIVE** | Present **and confirmed working** by actually using it | Available now, with what it will be used for |
| **REGISTERED, NOT LIVE** | Configured but its tools do not load — a failed start, an unmet login, or an environment that has not reloaded | **Treat as absent** until proven otherwise (§10.4) |
| **MISSING** | Not present | With a named example, its benefit, and its cost if declined |
| **NOT NEEDED** | Considered and rejected for this project | With the reason |

**Then say which are NECESSARY and which are merely useful**, and be precise about *for what*.
Necessity is not a property of a tool — it depends on the stage. A capability may be optional for
discovery and structural work while being the only way to prove interactive behaviour later. State
the boundary rather than a blanket verdict, so the owner can approve what a stage actually requires
instead of everything at once.

### 10.4 Registration and verification

- **Project-scoped by default.** Register in the target project's MCP configuration so the tool
  travels with the project that needs it and imposes nothing on unrelated projects. User-level
  registration is an owner decision, appropriate only for something they want everywhere.
- **Merge, never replace.** Add to the existing configuration and preserve every entry already
  there. Read it first (upgrader §15).
- **Approval is per capability.** The owner may accept browser automation and decline accessibility
  tooling. Do not bundle the decision.
- **Registration proves nothing.** A tool can be correctly registered and still never load — a
  failed start, an interactive login it cannot complete, or an environment that has not reloaded its
  configuration. **Verify by using it**: perform one trivial real operation and confirm the result.
- **Many environments require a restart** before newly registered tools become available. Say so
  plainly, and treat the capability as **still absent** until a post-restart smoke test passes.
- **Never install anything not explicitly approved**, and never install mid-stage — tooling changes
  belong at a stage boundary.

**Credentials:** browser automation signs in as a real user. Credentials come from the environment
at run time and are **never written into configuration, a repository file, a script, or the
journal** (upgrader §16). A tool that must store a password to work is not approved by default.

### 10.5 When a capability is missing or declined

This is the branch that matters most, because the tempting failure is to proceed quietly.

- **Name what becomes unprovable.** Not "limited verification" — the specific items. Without browser
  automation that is: modal behaviour, validation-error display, AJAX interactions, upload flows,
  responsive navigation, and console cleanliness.
- **Those areas are UNVERIFIED, never passed.** Absence of evidence is not evidence of correctness
  (upgrader §12), and an unverified area must never be reported as a working one.
- **Then let the owner choose**, with the cost visible: proceed with reduced scope and recorded
  gaps · limit migration to what *can* be verified · or pause until the capability exists.

**Recommended default:** the layout shell (Stage 1) can reasonably be migrated without browser
automation, since it is largely structural. **Migrating interactive modules without it is not
recommended** — forms, tables, modals and uploads are precisely what cannot be proven by inspection.
That is a strong recommendation and an owner decision, not a hard stop; if overridden, record the
override and the accepted risk.

---

## 11. Rollback

Inherits the upgrader's doctrine (§8) with one warning specific to frontend work:

> **Reverting Blade files is not a rollback.** A dashboard migration touches templates, stylesheets,
> scripts, frontend dependencies, build configuration and compiled output. Restoring one layer while
> another stays migrated produces a broken hybrid that is harder to diagnose than either state.

The recovery point must cover: Blade templates and components · CSS/Sass sources · JavaScript
sources · frontend dependency manifests **and lockfiles** · the installed dependency directory or a
proven way to reconstruct it · build configuration · **compiled/built output as actually served** ·
public assets, images and fonts · any configuration the migration touched.

Verify coverage against that list before relying on it (upgrader §8), and remember that the
frontend dependency directory has the same "manifests are not the tree" problem as the backend one.

**Do not treat backup capture as proof of recoverability.** For any recovery point relied on for
risky work, verify coverage **and prove restoration** into a disposable or otherwise isolated
target. A recovery point is valid only with evidence that the intended files, data and
configuration are covered · the backup is structurally valid · the restore process can actually
consume it · the restored target contains the expected objects and data · **the restore cannot
target the live project** · and the procedure is documented well enough to repeat.

A backup that exists but has never been restored is **not** a verified rollback capability.

**Uploaded user files are not migration artefacts** — they must never be inside the migration's
write scope at all.

---

## 12. Hard stops

Stop and report, in addition to every hard stop in the upgrader (§14):

- The selected template would require replacing the application's frontend architecture (for
  example, moving to a SPA) — that is a different project and needs a different decision.
- A mapping has **no equivalent and no owner decision**. Never resolve it by removing the feature.
- A permission-gated element cannot be traced to its gate. Never migrate a control you cannot prove
  the visibility rule for.
- Migration would require backend logic changes not agreed as part of the scope.
- The template's license does not permit the intended use.
- A migrated area is materially slower and the cause cannot be established.
- Coexisting frameworks are producing conflicts you cannot scope or explain.
- The dashboard migration is being asked for **inside** an upgrade window.

**When reporting a conditional or stopped outcome, classify every outstanding condition by type.**
The upgrader's status vocabulary (upgrader §2.11) and evidence labels (upgrader §1) still apply
unchanged; this adds only how the conditions *behind* such a verdict are reported. Each one is
exactly one of:

| Classification | Resolved by |
|---|---|
| **OWNER DECISION** | The owner choosing. No amount of investigation resolves it |
| **ENGINEERING PREREQUISITE** | Work that can be done now, before the dependent stage starts |
| **VERIFICATION GAP** | Obtaining the missing capability or context — never by deciding it is fine |
| **PRE-EXISTING DEFECT** | Nobody, under this scope. Recorded, attributed, and left alone (§9) |
| **UNKNOWN** | Evidence that does not yet exist. Never promoted to safe by default (upgrader §1) |

**The classification exists because it says who can act.** An owner decision cannot be engineered
away, a verification gap cannot be decided away, and a pre-existing defect is not this stage's work
at all. A list of conditions without types invites the wrong party to resolve the wrong item — most
often the agent quietly resolving something that was the owner's to choose.

---

## 13. Lesson capture

This capability is **DESIGNED, not proven** — no real migration has used it. The first one will
expose gaps.

Record what happens as **LESSON CANDIDATES** in the journal (upgrader §16). **Do not edit this file
during a migration**, and do not treat difficulty as evidence that a rule is wrong.

Before proposing any change here, classify the observation: an actual defect in this capability · a
missing instruction · wording that was unclear · a project-specific issue · a tool limitation · an
operator mistake · **a safety gate working correctly** · or a genuinely new reusable lesson. Most
first-migration friction is one of the middle categories.

Changes to this file go through the same governance as any other agent change
(`laravel-upgrader-governor.md`), including its asymmetry rule: **adding guidance needs one
well-evidenced case; weakening a protection needs proof the rule itself is defective.** "The
migration would have been quicker without this check" is a cost, not a defect.
