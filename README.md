<div align="center">

# 🚚 Logistics Carrier Screen

**An AI agent that opens the federal SAFER register, reads a motor carrier's authority and safety record, and reports back — then films itself doing it.**

[![CI](https://github.com/coasty-ai/coasty-logistics-carrier-screen/actions/workflows/ci.yml/badge.svg)](https://github.com/coasty-ai/coasty-logistics-carrier-screen/actions/workflows/ci.yml)
[![Node](https://img.shields.io/badge/node-%E2%89%A520.11-339933?logo=node.js&logoColor=white)](https://nodejs.org)
[![Dependencies](https://img.shields.io/badge/dependencies-0-brightgreen)](package.json)
[![Runs offline](https://img.shields.io/badge/runs%20offline-%240.00-blue)](#try-it-in-30-seconds)
[![License](https://img.shields.io/badge/license-MIT-black)](LICENSE)

<img src="media/demo.gif" alt="A vision model operating SAFER-II carrier safety register (mainframe inquiry) through a real browser" width="820">

<sub><b>This is a real capture.</b> Every frame is a screenshot taken by a real browser driving real
software while a vision model read each screen and chose the next action - 5 steps, 5 model calls,
no script and no answer key. Provenance and per-frame hashes in <a href="media/capture.json">media/capture.json</a>.</sub>

</div>

---

- **Zero dependencies.** No `npm install`, no lockfile, no supply chain — pure Node built-ins.
- **Runs offline for $0.** No API key, no account. A bundled in-process mock runs the full agent loop on a fresh clone.
- **The demo video renders itself.** The frames come straight out of the run — against live Coasty they are the model's own input frames, so there is no storyboard that can drift.

## What this is

A complete, runnable [Coasty](https://coasty.ai) computer-use automation for **motor-carrier safety screening**. Before a broker tenders a load or a shipper onboards a new carrier, somebody has to check that the carrier actually exists, is authorised to operate, and is not out of service. That check lives on a public government site, and it is done by hand — one carrier at a time, a clerk copying the same handful of fields into a spreadsheet.

It gives an AI agent one goal in plain English, and the agent drives a real browser to accomplish it — here, the SAFER-II carrier safety register (mainframe inquiry) — no selectors, no scraping rules, no DOM parsing to maintain. That matters more here than on most targets: SAFER is a decades-old government application with no public JSON API for this view, and a scraper pinned to its markup is one redesign away from silently returning nothing. An agent reading the page the way a compliance clerk reads it does not care where the fields moved to.

**Zero dependencies. Runs offline for $0 on a fresh clone. ~$0.70 to run for real.**

```
"You are covering the safety analyst desk on the SAFER-II carrier register
 terminal. Sign on as operator SAF07, leaving the region/node and access
 code exactly as they are pre-filled. From the function selection menu,
 open the CARRIER SAFETY INQUIRY function. Run an inquiry for domicile
 state TX - TEXAS, leaving the operating status filter on ALL and the
 snapshot as-of date as pre-filled. The results grid lists every
 Texas-domiciled carrier in the register. Scan that grid and find the
 SINGLE carrier with the largest number of power units, then pull up the
 carrier detail record for that carrier's USDOT number. The detail record
 re-prints the source-inquiry summary along its top line, so everything you
 need for the report is on that one screen. Report, quoting each value
 exactly as displayed: (1) how many records the inquiry selected, (2) the
 carrier's LEGAL NAME, (3) its USDOT NUMBER, (4) its POWER UNITS, (5) its
 MC/MX DOCKET NUMBER, (6) its MCS-150 FORM DATE, (7) its SAFETY RATING, (8)
 its VEHICLE OOS PERCENT, and (9) whether that vehicle OOS percent is ABOVE
 or BELOW the national average printed next to it."
```

That prompt *is* the automation. When the site redesigns, the prompt still works. Point it at a different USDOT number and it is a different screening; nothing else changes.

## Try it in 30 seconds

No API key. No account. No install. No spend.

```bash
git clone https://github.com/coasty-ai/coasty-logistics-carrier-screen
cd coasty-logistics-carrier-screen
npm start
```

That boots a bundled offline mock in-process and runs the whole agent loop against it. Then render the demo video from the run's own frames:

```bash
npm run demo     # needs ffmpeg; writes media/demo.mp4 + demo.gif + poster.jpg
```

Check your setup any time with `npm run doctor`.

## Run it for real

**1. Get a Coasty API key** — create one at **<https://coasty.ai/developers/keys>**.
The raw key is shown *once*, at creation, so save it when it appears.
A `sk-coasty-test-…` **sandbox** key never bills and is enough to try this;
a `sk-coasty-live-…` key bills your wallet. A new key already carries the
`runs:read` and `runs:write` scopes this automation needs, so there is
nothing extra to enable.

**2. Give both consents, then run:**

```bash
export COASTY_API_KEY=sk-coasty-test-...      # from the link above
export COASTY_BASE_URL=https://coasty.ai/v1
export COASTY_ALLOW_LIVE=1                     # destination consent
npm start -- --live --confirm-cost-cents 120   # cost consent
```

Both consents are required and they are deliberately separate. A live key alone will not spend; a base URL alone will not spend. See [Safety](#safety).

| | |
|---|---|
| Expected cost | **70¢** (14 steps × 5 credits) |
| Worst case | **120¢** (24-step cap) |
| Model-input frames | **free** |
| Machine runtime | Coasty provisions and destroys its own VM |

`npm run estimate` prints this before anything runs.

SAFER is a public federal register — no login, no key, no cookie. This automation reads one carrier snapshot per run and stops.

## What the agent actually did

It was given the prompt above and nothing else - no selectors, no coordinates, no answer key -
then operated **SAFER-II carrier safety register (mainframe inquiry)** through a real browser:

```
software    SAFER-II carrier safety register (mainframe inquiry)
model       gpt-5.2
steps       5 (each = one screenshot, one decision, one action)
cost        ~$0.019
captured    2026-08-02
```

What it reported, read off the screen:

```
  (1) 11 RECORD(S)
  (2) LEGAL NAME: LONE STAR REEFER EXPRESS INC
  (3) USDOT NUMBER: 1533902
  (4) POWER UNITS: 342
```


## Safety

This repo is built so that **accidental spend is structurally impossible**, not merely discouraged:

- **Fail-closed destination.** An unset `COASTY_BASE_URL` resolves to the bundled offline mock. Production is never a default.
- **Two independent consents.** `COASTY_ALLOW_LIVE=1` authorises the *destination*; `--confirm-cost-cents N` authorises the *cost*, and N must equal the server-computed worst case exactly.
- **Idempotency by default.** The submit key is derived from the prompt, so a retried submit returns the original run instead of provisioning a second machine.
- **A hard cap per unit.** A worst case above `capCents` in [`automation.json`](automation.json) is refused before any request is made.
- **No real credentials.** The captured demo signs on to a simulated legacy system with a
  throwaway operator ID that the system itself displays. Nothing here reads a real
  password, token or cookie, and no secret is stored in this repo.

## Project layout

```
automation.json      the entire unit definition — prompt, target, budget, caps
src/client.mjs       Coasty client: fail-closed target, retry, idempotency
src/capture.mjs      model-input frames → mp4/gif/poster, with sanity checks
src/cli.mjs          run · demo · estimate
tools/mock.mjs       the bundled offline Coasty (real 1280×720 PNG frames)
tools/doctor.mjs     preflight
test/                36 tests, zero dependencies, fully offline
```

Adding a new automation is one `automation.json` and one prompt — `src/` never forks. See [AGENTS.md](AGENTS.md) for the authoring contract used by Claude Code and Codex.

## Tests

```bash
npm test     # node --test, no install, no network, no key
```

## Related

Part of the **Coasty automation catalog** — computer-use automations across 12 industries. See [the index](https://github.com/coasty-ai) for retail, finance, healthcare, legal, energy, public sector, HR, education, manufacturing, nonprofit and e-commerce.

- [Coasty docs](https://coasty.ai/docs) · [API reference](https://coasty.ai/docs/llms.txt)
- [computer-use-cookbook](https://github.com/coasty-ai/computer-use-cookbook) — the API, by endpoint, in 4 languages
- [open-cowork](https://github.com/coasty-ai/open-cowork) — the open-source AI coworker

## License

MIT © Coasty
