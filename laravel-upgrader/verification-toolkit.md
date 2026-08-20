# Laravel Upgrader — Verification Toolkit

Portable patterns for the checks referenced in the `laravel-upgrader` agent. These are
**templates to adapt**, not drop-in files. Every one was derived from a real upgrade, but all
project-specific detail has been removed.

Conventions used below:
- `<APP>` — how you execute PHP **in the real runtime** (§2 of the agent). Examples:
  `docker exec <app-container>`, `<path-to-xampp>/php/php.exe`, plain `php`.
- Scripts are written to a scratch/tooling directory and run **from the project root**.
- Everything here is **read-only** unless explicitly noted.

---

## 0. Two traps that will bite you in any shell

**Piped exit codes.** `cmd | head` reports *head's* status, so a failing command looks successful
and `|| echo "failed"` never fires. Capture the real status:

```bash
n=$(git ls-files some/path | wc -l); echo "tracked files: $n"   # infer from output, not $?
out=$(cmd); rc=$?                                                # or capture separately
echo "EXIT=${PIPESTATUS[0]}"                                     # bash: first element of the pipe
```

**Path mangling under Git Bash / MSYS on Windows.** A leading `/` in an argument is rewritten to a
Windows path, so `/dashboard` silently becomes `C:/Program Files/Git/dashboard` and you get 404s
that look like real failures. Use `MSYS_NO_PATHCONV=1`, or run the command from PowerShell.

Also: escaping differs per shell. When a value contains spaces or template braces, prefer building
an **argv array** via `proc_open`/`Process` over interpolating into a shell string.

---

## 1. Runtime compatibility probe (disposable)

**Purpose:** decide incremental-vs-single-window (agent §9) with evidence instead of belief.

**Method:** for each candidate PHP version, boot the *current* application in a **disposable**
container/environment and record boot success or the fatal error. Then do the same for the target
framework's requirement. Compare the intervals.

- Mount the project **read-only** where possible; never let a probe write to the real tree.
- Use throwaway container names and remove them by **captured ID**, never by wildcard.
- Record the actual error text — "Laravel N cannot boot on PHP X because <exact exception>" is the
  evidence that justifies a coordinated window.

If the "runs current framework" and "runs target framework" PHP intervals do not intersect, the
runtime and framework must move together. Document that conclusion in the journal.

---

## 2. Checkpoint with structural verification

**Purpose:** a checkpoint you have actually proven, not a file that happens to exist.

Gates — **all must pass or the checkpoint is invalid**:

1. **Identify the target** positively (project label / compose project / working dir / image), not
   by name similarity.
2. **Live object list is non-empty.** An empty table list is a STOP, never "zero tables".
3. Dump: header present · completion marker present · object-name set **matches live** · non-empty.
4. Capture what source control cannot regenerate: `.env`, application/OAuth keys, uploads.
5. Capture `vendor/` (or prove a reinstall reproduces it — agent §8).
6. Write a manifest: paths, counts, timestamps, verification status.
7. Store **outside** version control and confirm ignore rules cover it.

Prefer argv arrays over shell strings — database names and paths containing spaces break naive
quoting, and on Windows `escapeshellarg` mangles template braces.

---

## 3. Restore rehearsal

**Purpose:** prove restore works *before* you need it.

Restore into a **disposable** target only. **Prove isolation before restoring**: confirm the target
shares no volume, network, or container ID with the live project. Clean up by captured ID.

Never rehearse a restore over the live database.

---

## 4. Regression baseline / compare

**Purpose:** turn "seems fine" into a diff.

Record **before** the upgrade, compare after:

| Signal | Notes |
|---|---|
| Framework boot + version | |
| Route count | Necessary, **not** sufficient — see §6 |
| Migration status | Count of applied migrations |
| HTTP probes | A few representative paths: status **and** body size |
| Row counts | Users, roles, permissions, pivot tables, key domain tables |
| Object/table count | |

Every difference must be **deliberate and documented, or it is a defect.**

Two cautions learned in practice:
- A probe that returns `0`/no response may be **your harness timing out** (e.g. a small
  `file_get_contents` timeout against a large debug error page), not the app failing. Confirm with
  an independent client before reporting a regression.
- Body-size changes are expected when a debug/profiler bar appears or disappears. Investigate the
  cause; do not assume either direction is fine.

---

## 5. Provider / alias existence check

**Purpose:** find every missing class at once instead of one boot fatal at a time.

Parse the app config for `Foo\Bar::class` provider entries and `'Alias' => Foo\Bar::class` entries,
then test each with `class_exists()` after the autoloader is loaded.

```php
require __DIR__.'/vendor/autoload.php';
$src = file_get_contents(__DIR__.'/config/app.php');
$cls = 'A-Za-z0-9_'.chr(92).chr(92);   // NOTE: backslash must be DOUBLED inside the class,
                                       // a single trailing one escapes the closing ']'
preg_match_all('/^\s*([A-Z]['.$cls.']+)::class,/m', $src, $providers);
preg_match_all('/^\s*[\'"]([A-Za-z0-9_]+)[\'"]\s*=>\s*([A-Z]['.$cls.']+)::class,/m', $src, $aliases);
// then: class_exists() each, report all misses together
```

Typical findings: a package moved namespace across a major; a facade moved to a sub-namespace; a
**dev-only** package hardcoded into config (a latent production failure under `--no-dev`) — prefer
removing it and letting package auto-discovery register it.

---

## 6. Named-route resolution check

**Purpose:** the check a route *count* cannot give you.

Bootstrap the app, collect every registered route name, then scan `app/`, `routes/` and view files
for static `route('name')` references and resolve each.

```php
preg_match_all('/(?<![\w:>])route\(\s*[\'"]([A-Za-z0-9_.\-]+)[\'"]/', $src, $m);
```

Dynamic names (variables/concatenation) cannot be checked statically — say so rather than implying
full coverage.

**Interpreting results:** a missing name is not automatically your fault. Separate
"**I removed this**" from "**never existed**" by checking whether the name is defined anywhere in
the route files. Long-lived commercial codebases often carry many dead `route()` references that
were already broken. Report the two groups separately — and fix the ones you caused.

---

## 7. In-process authenticated render

**Purpose:** exercise routing + middleware + authorization + the admin layout, which is where a
removed route or broken directive actually fatals.

```php
require __DIR__.'/vendor/autoload.php';
$app = require_once __DIR__.'/bootstrap/app.php';
$kernel = $app->make(Illuminate\Contracts\Http\Kernel::class);
// Bootstrap via the CONSOLE kernel: the HTTP kernel's bootstrappers expect a 'request' binding
// that does not exist yet ("Target class [request] does not exist").
$app->make(Illuminate\Contracts\Console\Kernel::class)->bootstrap();

$user = /* first user with the widest role */;
Illuminate\Support\Facades\Auth::login($user);
$request = Illuminate\Http\Request::create($path, 'GET');
$request->setLaravelSession($app['session.store']);
$response = $kernel->handle($request);
```

**Status alone is not enough** — with debug enabled a 500 still returns HTML. Also scan the body for
exception signatures (`Route [...] not defined`, `Undefined variable`, `Call to undefined`,
`SQLSTATE`).

> **Known ceiling — this is important.** `Auth::login()` **bypasses the login controller entirely**.
> This check can pass completely while login itself is broken. It is necessary but never sufficient.
> Use §8 for the login flow.

---

## 8. Real-HTTP login smoke test

**Purpose:** the test §7 structurally cannot perform. Covers the login POST, CSRF round-trip,
session cookies, redirects, and the middleware stack as the web server actually serves it.

1. `GET /login`, keep a cookie jar, extract the CSRF token from the rendered form
   (it renders as a `_token` hidden input — grepping for the literal string "csrf" finds nothing
   and looks like a false alarm).
2. `POST /login` with `_token` + credentials.
3. Assert success properly: not just HTTP 200, but that you are **no longer on a form** and did not
   land on an error page.
4. Then `GET` protected paths, treating **a redirect back to `/login` as a failure** (session lost),
   not a pass.

**Credentials must come from the environment, never a committed file, and never appear in output,
the journal, or a report.**

Still does **not** execute JavaScript — data grids and AJAX remain unproven. That needs §9.

---

## 9. Browser regression

For JavaScript, data tables, and real form submissions, use browser automation. Prefer an
accessibility-tree driven tool (deterministic) over screenshot matching.

Session artefact directories capture snapshots of **authenticated** pages — treat them as sensitive
and ensure they are ignored by version control.

---

## 10. Published-config vs vendor-default schema diff

**Purpose:** catch configuration drift as a set instead of one runtime fatal at a time (agent §7).

Include both config files, stubbing framework helpers so they can be evaluated standalone, then
compare **key structure** (dot-notation paths), not values:

```php
foreach (['env','base_path','storage_path','public_path','config_path','database_path'] as $fn) {
    if (!function_exists($fn)) { eval("function $fn(\$a=null,\$b=null){ return \$b ?? \$a ?? ''; }"); }
}
$app = include $appConfigPath; $vendor = include $vendorDefaultPath;
// flatten both to dot paths (skip integer keys — list entries are data, not schema), then diff
```

Report **MISSING in app** (package added these) and **EXTRA in app** (package dropped/renamed
these). Rebuild on the new schema, preserving local customisations verbatim and marking each.

Run it for **every** published config with a vendor counterpart — not only the one that just
crashed.

---

## 11. Security applicability probes

**Purpose:** decide, from evidence, which security domains are APPLICABLE / NOT APPLICABLE /
UNKNOWN (agent §13.1) so the audit only covers surfaces the project actually has.

Each probe answers one question: *does this surface exist here at all?* A **negative result is a
finding too** — record N/A with the evidence, then skip the whole dependent domain.

| Domain | Cheap evidence that it exists |
|---|---|
| File uploads | Upload validation rules, file-request handling, storage writes, upload form fields |
| API | API route files, stateless auth guards, token tables/migrations, resource serializers |
| Webhooks | Unauthenticated POST routes, signature-verification helpers, CSRF exclusions |
| Queues / schedules | A non-sync queue driver, queued job classes, scheduled command definitions |
| Outbound HTTP | HTTP-client usage, cURL calls, integration/SDK packages |
| Multi-tenancy | A tenant/org foreign key on core tables, global scopes, tenant middleware |
| Exports | Spreadsheet/CSV/PDF generation packages and the routes that stream them |
| MFA / OAuth | Second-factor columns or packages; OAuth client tables; social-login integrations |

Two cautions:
- **Absence in one place is not absence.** A missing route file does not prove there is no API —
  check package-registered routes too (enumerate the live router, §6).
- **Grep proves presence, not reachability.** A hit means the domain is APPLICABLE and needs
  auditing; it does not by itself mean the code is live (agent §3 classification still applies).

---

## 12. Authorization matrix testing

**Purpose:** the check scanners cannot perform — whether a *valid but unauthorised* identity can
reach data or actions.

Build a small matrix from the project's own roles and resources, then test each cell:

| | Owner | Same-role, different owner | Lower privilege | Unauthenticated |
|---|---|---|---|---|
| View | expect allow | **expect deny** | expect deny | expect deny |
| Update / delete | expect allow | **expect deny** | expect deny | expect deny |
| Export / download | expect allow | **expect deny** | expect deny | expect deny |
| Admin action | deny | deny | deny | deny |

Method:
1. Authenticate as each identity (§7 in-process is fine for breadth; §8 real HTTP for the paths
   that matter most).
2. Request the resource **by identifier**, including identifiers the UI never shows.
3. Repeat against the **API** as well as the web route — they frequently enforce different rules.
4. Record the *actual* status and body, not the expected one.

**A deny must be a real deny.** Treat 200 with an empty body, a redirect to a login page, or a
"not found" that still confirms existence as results to inspect, not automatic passes. And a
missing authorization boundary returns **200** rather than erroring, so absence of exceptions
proves nothing.

**Read-only.** Use existing fixtures or read-only verbs wherever possible; never create, mutate or
delete real records to prove a point, and never run a mutating test suite against a real database.

---

## 13. Secret and exposure scanning

**Purpose:** find credentials and artefacts that should never be reachable or committed.

- Scan **version-control history**, not just the working tree — a rotated key still leaked if it
  was ever committed, and the fix is rotation plus history handling, not deletion of the file.
- Check ignore rules **before** looking for damage: environment files, key material, database
  dumps, backup and checkpoint directories, and browser/session artefact folders.
- Check what is reachable under the public web root: build artefacts, source maps, backups,
  archives, dumps, and any development or profiling tooling.
- Check runtime exposure: debug flags enabled, verbose error pages, and administrative or
  monitoring endpoints without authorization.

**Never print a secret's value** — anywhere, including the journal. Report presence, location, and
whether it is empty, and recommend rotation when it may have been exposed. Note that
`env('KEY', 'default')` returns `''` for a set-but-empty key, so "configured" and "non-empty" are
different questions (agent §7).

---

## 14. Command guard (optional hard control)

A `PreToolUse` hook can block destructive commands (`migrate:fresh`, `db:wipe`, seeding, `DROP`,
container prune, `git reset/clean/checkout/stash/push`) before they execute.

If you build one:
- **Anchor keywords to command position**, not raw substring. Otherwise it matches prose and blocks
  ordinary work — including writing a *code comment* that merely mentions a keyword.
- Keep a test harness with must-block and must-allow cases, and keep it passing.
- **Never weaken the guard to let an operation through.** Reword the command, or improve the
  matching deliberately and report it. A guard you route around is not a control.

---

## 15. Journal

Append-only JSONL, one record per stage start/finish: stage id, objective, authorised scope,
checkpoint reference, status, notes. Provide a `gate` operation that exits non-zero when the last
status was `BLOCKED` or `RECOVERY REQUIRED`, so the next stage cannot start.

Statuses: `VERIFIED` · `VERIFIED_WITH_REMAINING_FINDINGS` · `BLOCKED` · `SAFELY_STOPPED` ·
`RECOVERY_REQUIRED`.

`SAFELY_STOPPED` is a **success outcome** when investigation proves an approach is wrong — recording
why an approach was abandoned is as valuable as recording one that worked.

**No secrets in the journal.**
