# SIO Intelligence

Monthly Content Intelligence & Lead Discovery for SIO Agency. It researches what crypto founders are actually saying, diagnoses their problems, and turns the problems **you approve** into a month of X and LinkedIn posts with deliberate CTAs. A second module finds real companies that may need SIO, with sourced evidence and separate scores. Nothing is published or sent automatically.

## Quick start

```bash
npm install
cp .env.example .env.local     # then fill in the values below
npm run migrate                # optional: the app also migrates on first request
npm run dev                    # http://localhost:3000
```

Production: `npm run build && npm run start`.

## Credentials and configuration

| Variable | Required | Purpose |
|---|---|---|
| `SIO_ADMIN_PASSWORD` | yes | Password for the single SIO workspace login |
| `SESSION_SECRET` | yes | 32+ random characters that sign the session cookie. `openssl rand -hex 32` |
| `ANTHROPIC_API_KEY` | for research and generation | Reasoning, extraction **and** web search (Anthropic's server-side web search tool) |
| `SIO_MODEL_REASONING` / `SIO_MODEL_EXTRACTION` | no | Default models (override per workflow in Settings) |
| `SIO_DB_PATH` | no | SQLite file (default `./data/sio.db`) |

- **Web search must be enabled** for your Anthropic organization in the Console. It is a paid, usage-billed tool. If it is off, research fails with a message saying so.
- Without `ANTHROPIC_API_KEY` the app still runs (login, business brain, approvals, calendar editing, exports, performance entry). Research and generation return a clear setup error. **Nothing is simulated.**
- Keys live only in server environment variables and are never sent to the browser.

## The monthly workflow

1. **Prepare** – Dashboard → create the cycle for next month. Your Business brain is loaded automatically.
2. **Research** – six source categories, targeted queries, evidence records with source URL, publication date and collection date. Coverage per category is reported as available, partial or unavailable.
3. **Diagnose** – clusters evidence into problems; symptom and hypothesised root cause are shown separately; six 1–5 scores; opportunity map.
4. **Approve research** – approve, reject, defer, edit, annotate, mark "matches my client experience". You can add your own client-experience problems (labelled as unverified assumptions).
5. **Approve strategy** – edit themes, pillars, weekly narrative and measurement, then approve.
6. **Generate content** – up to five posts per week (configurable), every post tied to approved evidence ids or flagged as a hypothesis.
7. **Review and export** – edit, regenerate or adapt single posts, approve, export CSV / Markdown / printable document.
8. **Learn** – enter or import results; month-end analysis feeds the next cycle.

Gates enforced in code, not just the UI: strategy requires at least one approved finding; the calendar requires an approved strategy; a calendar cannot be approved while posts lack drafts; lead outreach statuses require the lead to be approved first.

## Honest limits

- **X and Reddit** are reached only through the public web search index (domain-restricted searches). Coverage is partial and labelled so. Use *Import a source manually* on the Market research screen for specific posts or threads. The official X and Reddit APIs are not integrated.
- **Podcasts** are found only where a transcript or show page is indexed.
- **Publishing** to X/LinkedIn is intentionally not connected. **Lead outreach** is never automated.
- **Impressions and profile visits** are never fetched; enter or import them (CSV) on the Performance screen. Missing metrics display "n/a", never zero.
- **Jobs** run in the server process with durable state in the database (progress, retry, crash recovery by re-queueing). There is no separate worker service, so run a single server instance. For multi-instance deployment, move the job runner to a queue.
- **Database** is SQLite (a single file, WAL mode) to keep setup to one command. The schema is plain SQL in `migrations/`. Back up `data/`. Moving to Postgres/Supabase would mean porting the data layer.
- Auth is a single shared workspace password with signed cookies, login throttling and same-origin checks on writes. There are no per-user accounts.
- Lead decision makers are recorded only when the cited page was among the retrieved sources. A lead is saved only if its website is independently verified by search.
- The first real research run is the one thing that could not be exercised here without your API key. Everything around it is tested with the Anthropic SDK faked at its boundary (see Tests).

## Evidence and safety rules built in

Verbatim quotes are only labelled verbatim if they literally appear in the retrieved text. Confidence follows a deterministic rubric (interpretations are always low; "high" needs a verified primary quote or two independent sources). Reposts and syndicated copies are collapsed and never count as corroboration. Old evidence is demoted to "historical" and is not reused to claim a trend. Retrieved web text is wrapped as untrusted data and delimiter-escaped. Drafts are rejected and regenerated if they contain unapproved offers (free audits, discounts, guarantees), unsupported statistics, generic AI openers or posts outside the word range (X 200–280 words, LinkedIn 200–400 words, both configurable in Settings; X posts are single posts, not threads). Lead buying-intent and problem-evidence scores are capped by what the evidence supports, and fit / problem evidence / buying intent are never merged into one score.

## Tests

```bash
npm test          # 80 tests across 6 files
npm run typecheck
```

Covers: scoring and rubrics, deduplication, recency, quote integrity, content rules, CTA levels, calendar slot generation, profile persistence across sessions (on-disk), approval gates, calendar edit/regenerate isolation, exports, jobs (duplicate prevention, crash recovery), lead gating and scoring, performance validation and CSV import, session security, and an end-to-end research → approval → strategy → calendar → CTA → export run with a faked model, including malformed-output retries, unavailable sources and budget exhaustion.

## Layout

`src/lib/research` pipeline, evidence rules, search provider · `src/lib/content` strategy, calendar, rules, export · `src/lib/leads.ts` · `src/lib/approvals.ts` · `src/lib/performance.ts` · `src/lib/ai` model client and structured output · `src/app` screens and API routes · `migrations/` schema.
