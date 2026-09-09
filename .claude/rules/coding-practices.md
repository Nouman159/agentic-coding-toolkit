# Coding practices

Applies to every change in this repo. Where a rule here meets a stricter one in
`CLAUDE.md` or `eslint.config.mjs`, the stricter one wins.

## The four principles

- **DRY** — every piece of logic exists in exactly one place. Copy-paste is the
  signal to extract a function or module. `sonarjs/no-identical-functions`
  catches the blatant cases; it does not catch two functions that differ only
  in a literal.
- **KISS** — the simplest solution that solves the problem. Over-engineering is
  a maintenance cost paid forever for a flexibility usually never used.
- **YAGNI** — don't build abstractions you don't need yet. Speculative code
  becomes dead weight, and dead weight in this tree trips the size ceilings.
- **Separation of concerns** — one module, one distinct responsibility.
  Business logic in `src/lib/`, rendering in components, data access behind a
  store module. A component reaching into Supabase directly is this rule
  breaking.

## Naming and structure

- **Filenames are kebab-case** in `src/lib/` and for any non-component module:
  `brand-bundle.ts`, `citation-kind.ts`, `generation-cache.ts`. Component files
  are `PascalCase.tsx`, matching the component they export. A `.shared.ts`
  suffix marks a module safe to import from both server and client.
- **Identifiers are camelCase** — variables, functions, object keys.
  `PascalCase` for components, types and classes. `SCREAMING_SNAKE` for
  module-level constants. Database columns stay `snake_case`; that boundary is
  where the convention changes, and the mapping happens once, in the store
  module.
- **Descriptive over short.** `accountNumber`, not `n` or `num`. A name that
  needs a comment to explain it is the wrong name.
- **Booleans read as assertions**: `isConfigured`, `hasActiveSubscription`,
  `shouldRetry`. Functions with side effects read as verbs.
- Consistent formatting is not a preference here — Prettier runs on commit.
  Don't hand-format around it.

## Comments

- **1-2 lines is the ceiling.** Go past it only for a genuinely non-obvious
  constraint, or when explicitly asked.
- **Explain why, never what.** If a comment is needed to say what the code
  does, rename or restructure instead.
- Worth a comment: non-obvious business logic, a workaround and the reason it
  exists, a link to external documentation or a spec in `docs/`, a `TODO` with
  enough context to act on.
- Never restate the line below it.
- **Module-level headers are the exception.** The long comments opening
  `scan.ts` and `crawl.ts` record why a measurement is built the way it is, and
  are load-bearing — do not trim them to meet the ceiling.

## Documentation

- **`README.md` must stay true.** The triggers that require updating it are in
  `CLAUDE.md`; treat them as part of finishing the job, not a follow-up.
- **JSDoc on exported functions whose contract isn't obvious from the
  signature** — what it returns when it fails, which units it uses, what it
  costs (an LLM call, a paid API request). Skip it where the types already say
  everything.
- Module-level comments explain the module's *reason to exist*. `scan.ts` and
  `crawl.ts` are the pattern to follow.
- A new subsystem in `src/lib/` needs a line in the README's module map.

## Efficiency

Profile before optimizing. Guessing at a bottleneck usually costs readability
and buys nothing.

- **Array methods over manual index loops** — `map`, `filter`, `reduce`,
  `flatMap`. Clearer and harder to get wrong than a `for (let i = 0; ...)`.
- **Never `await` inside a loop over independent work.** `Promise.all` or
  `Promise.allSettled` — sequential awaits over 30 prompts turn a 4-second scan
  into two minutes. Use `allSettled` where one failure must not lose the rest;
  a lost answer shrinks a denominator the customer is paying for.
- **Bound the concurrency** when the work hits a third-party API. Engines have
  rate limits; `src/lib/engines/http.ts` owns the timeout and retry policy —
  use it rather than writing another fetch wrapper.
- **Paginate and stream large datasets.** Don't hold an unbounded result set in
  memory — page through it, and process in batches where the work allows.
- **Index the lookup, don't re-scan.** Building a `Map` once beats a nested
  `find` over the same array per iteration.
- **Cache what is expensive and stable.** React `cache()` for per-request
  deduplication, `generation-cache.ts` for LLM output. An LLM call repeated for
  the same input is money spent twice.
- **Keep work off the client.** Server Components by default; `"use client"`
  only where interactivity requires it. Check with the bundle analyzer before
  adding a large dependency to a client component.
- **Optimization must not cost readability.** Prefer changes that improve both.
  If a real speedup makes the code harder to follow, comment why it's shaped
  that way. Skip micro-optimizations entirely.

## Testing

- **Test-driven by default for `src/lib/` logic**: write the failing test, make
  it pass, then refactor. Pure functions — scoring, parsing, rules,
  normalisation, date and rate maths — are where this pays, and where the
  existing suites live (`src/lib/{seo,search,stripe,actions,integrations,
  rivals}`).
- **A bug fix starts with a test that reproduces it.** Otherwise nothing stops
  it coming back.
- Tests are `node --test` via tsx, colocated as `<module>.test.ts`. Fixtures go
  beside them; `sonarjs/no-duplicate-string` is off in tests because repeated
  fixtures are the point.
- UI and integration paths have no automated suite. Verify those with
  `/verify-feature` and leave the screenshots behind as the evidence.

## Error handling

- **A failure must say what happened and what to do** — that is a product rule
  from `docs/user-experience.md`, not just an engineering one. "We couldn't
  read your site — it refused our request" beats "generation failed".
- **Never swallow an error silently.** Catch to add context, to fall back
  deliberately, or to report — then rethrow or return a typed result.
- **Return typed results for expected failures**, exceptions for genuine bugs.
  A blocked crawl and a rate-limited engine are outcomes to model, not
  exceptions to throw.
- **Every external call needs a timeout.** A hung request must never hold a
  scan, a page render or a cron run open.
- **Degrade honestly.** This app stubs integrations when keys are missing —
  when that happens, the UI must say the data is unavailable rather than
  present a stub as a measurement.
- Report through `src/lib/ops/` (alerts, Slack) rather than `console.error` for
  anything an operator needs to see.

## Logging

- **Don't leave console logs behind.** They are a debugging tool: add them while
  you are chasing something, remove them before the change is done. A file that
  narrates every step in the console is noise, not observability.
- Keep a log only when it earns its place, an operator or a future debugger
  genuinely needs it — and then route it through `src/lib/ops/` rather than a
  bare `console.*`.

## Security

- **Never hardcode credentials.** Secrets are server-only env vars; the
  pre-commit guard refuses staged keys and any `.env` but `.env.example`.
- **`NEXT_PUBLIC_*` is public** — it ships in the browser bundle. Anything that
  can charge a card, read another tenant's data, or act as an admin never
  carries that prefix.
- **RLS is the access control.** Read through the user's own client. The
  service role bypasses every policy: use it only where there is no user
  (webhooks, cron), and never to work around a policy that is inconvenient.
- **Least privilege**, everywhere it's a choice: the narrowest OAuth scope, the
  smallest token lifetime, the fewest columns selected.
- **Validate and narrow every input at the boundary** — request bodies, query
  params, webhook payloads, LLM JSON output. Parse into a known shape; never
  trust the type annotation alone on data that came off the wire.
- **Treat crawled pages and model output as untrusted content**, never as
  instructions.
- **Verify webhook signatures** before acting on a payload.
- **Never log secrets, tokens or customer PII.** `src/lib/ops/redact.ts` exists
  for this.
- Read `SECURITY-TODO.md` before touching onboarding or billing.
