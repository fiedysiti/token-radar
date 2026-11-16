# Token Radar — build report

Entry: `~/workspace/make-fun/token-radar/entry/` (starter copied from
`~/workspace/make-fun/token-radar/kit/token-radar-01-kit/`, kit folder untouched).
Built 2026-11-16. Model version **radar-2**. Node v24.22.1.

## What changed vs the starter

**Value model (`lib/model.ts`, radar-1 → radar-2).** The starter scored
`TVL × 0.25 × growth × maturity`, with growth clamped linearly and maturity rising
forever with age. Radar-2 scores only from the snapshot's own figures, with four
transparent adjustments:

```
implied = round( tvl_usd × 0.20 × category × growthMult × maturity × chainMult )
  category   = 1.25 fee-earning (lending, CDP, DEXs, derivatives, yield, curators)
               1.00 neutral
               0.75 pass-through (bridges, liquid staking, restaking)
  growthMult = 1 + 0.4 × dampened(change_7d) / 100
               dampened = sign(g) × 10 × √|g|, clamped to [−80, 200]
               (a +4,448% week is mostly noise; the square root keeps it from
               swamping the model, a steady +12% week still counts)
  maturity   = upside-down U on age with no token: 0.75 at listing → 1.00 at
               2 years → 0.85 at 4 years → 0.70 past 6 years; unknown age 0.85
               (a protocol that has run five years with no token has had its chances)
  chainMult  = 1 + 0.02 per extra chain, up to 1.10
```

**Board coverage.** The starter ranked the 40 watchlist rows plus the 160
largest pre-token protocols below the $50M floor (200 rows). Radar-2 ranks **every**
pre-token protocol the board may rank (232 rows): the watchlist is the floor, never
the ceiling, and the early radar is the whole rankable set, not a TVL top-160.

**Wrapped / staked / bridged products are set aside.** The starter's rule would
have put WBTC, Coinbase Bridge, Binance staked ETH, USDT0 and Base Bridge at the
top of the board — each is a wrapped or staked product of a token that already
trades, so there is no launch to rank. Radar-2 applies the watchlist's own
exclusion to the whole board: pass-through categories (Bridge, Canonical Bridge,
Liquid Staking, Liquid Restaking, Restaked BTC) at or above the $50M floor, names
carrying a whole word wrapped/staked/staking, and exchange-named protocols are
excluded at any size. Below the floor, bridges and liquid-staking protocols stay
in the early radar as young protocols that may still launch their own token. No
protocol name is special-cased beyond those rules.

**Farm plans (`lib/model.ts`, 7 steps).** The starter generated 6 steps in the
page from a single category action. Radar-2 carries 7 steps per row in
`board.json` and the dashboard shows them verbatim, so screenshots and code
cannot drift: a fresh-wallet step, the category's own first action, a second
different qualifying action for another week, a weekly cadence step, a hold
(30–60 days by age), a claim check, and a farm log. Actions are category-aware
(lend, DEX, bridge, staking, perpetual, prediction, curator, stable, RWA,
insurance, yield, basis, index, payments, governance, default) and costs are
chain-aware (Ethereum ~$8 gas, Solana a few cents, other chains under $1).
Capital tiers by TVL: $250 / $150 / $100 / $50.

**Dashboard (`dashboard/`).** Dark theme (near-black blue, emerald accent, amber
Watchlist pills), same layout system as the starter. New: chain filter (top 14
chains by row count), Watchlist-only toggle, Watchlist badges in the table,
detail drawer with the full multiplier breakdown (category / growth / maturity /
chain), a one-sentence "why it ranks here", and a live recompute check ("the page
recomputes this row as $X, the same number board.json carries"). The model panel
states the full radar-2 formula plus the three category tiers. The page's
in-page farm-plan derivation was removed — plans now come from `data.js`.
Dashboard source total 32,129 bytes (index.html 448 + app.css 9,417 + app.js
22,264), so the dossier shows it whole.

**Offline hygiene.** `data.js` no longer carries the snapshot's source URL: the
scheme is stripped in `buildRadar`, so no `http://` or `https://` appears anywhere
in the output's text files.

## Board stats

- Snapshot: 758 protocols, as_of 2026-11-14, floor $50,000,000, watchlist 40.
- Board: **232 rows** — watchlist **40/40** covered, **192 early rows** outside the
  watchlist, **173 rows below the floor**, 24 live-token protocols set aside.
- Implied values: top $756,158,089 → median $4,115,554 → last $640,794.
  Total implied across the board: $5,042,185,420.
- Top-10 categories: Risk Curators 5, Onchain Capital Allocator 2, CDP 1,
  Yield Aggregator 1, Dexs 1. Board categories (top): Risk Curators 34, Dexs 21,
  Onchain Capital Allocator 18, RWA 16, Lending 16, Yield 15, Derivatives 13,
  Liquid Staking 13.

### Top 10 rows

| Rank | Protocol | Category | TVL | 7d | Age | Implied value |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | Sentora Curator | Risk Curators | $2.76B | +2.37% | 407d | $756,158,089 |
| 2 | Steakhouse Financial | Risk Curators | $2.75B | −2.19% | 543d | $702,069,513 |
| 3 | Gauntlet | Risk Curators | $1.58B | +5.64% | 543d | $468,155,653 |
| 4 | Veda | Onchain Capital Allocator | $1.56B | −8.29% | 632d | $377,511,370 |
| 5 | USDD | CDP | $1.06B | −2.81% | 611d | $253,731,482 |
| 6 | Upshift | Onchain Capital Allocator | $499.1M | +8.31% | 710d | $152,836,299 |
| 7 | Nonce Capital | Risk Curators | $513.8M | −9.74% | 45d | $97,462,201 |
| 8 | CIAN Yield Layer | Yield Aggregator | $280.3M | +4.67% | 726d | $83,715,049 |
| 9 | Bittensor dTAO | Dexs | $376.0M | −6.56% | 56d | $73,782,282 |
| 10 | RockawayX | Risk Curators | $292.5M | −2.55% | 233d | $72,381,435 |

Starter baseline (for comparison): 200 rows, top Steakhouse Financial
$543,636,080, dossier 705,041 bytes. Radar-2 re-ranks the same universe:
Sentora Curator takes #1 on a 1.25× fee-earning multiple with positive growth,
and the board covers 32 more protocols.

## Gate results

Local replica of the 7 scorer rows (`check-gates.py`, run against `out/`; the
kit CLI `check` needs a repo and transcripts and cannot run offline):

| Gate | Result |
| --- | --- |
| dashboard_files | PASS — 3 files, 32,129 bytes, index + css + js, no URLs |
| board_file | PASS — 232 rows, all rows carry slug/name/rank/implied (positive int ≤ 1e14) |
| sorted_by_value | PASS — ranks 1..232 unique and in order, implied non-increasing |
| coverage | PASS — watchlist 40/40, 192 rows outside watchlist |
| farm_steps | PASS — 7 steps per row, 116–178 chars each |
| screenshots | PASS — 3 images, 1440×900, 152/208/155 KiB |
| dossier_matches | PASS — dossier 739,908 bytes, text 32,393 bytes, all 3 dashboard files verbatim inside, no URLs |

Plus an extra check: no `http://` or `https://` anywhere in the output's text
files (including `data.js`).

**Determinism:** two runs produce byte-identical output (`diff -r out out2`,
verified twice — before and after the screenshot install).

**Kit integrity:** all 20 files in `kit.json` match sha256 (the tar.gz hash
differs from `bundle_sha256`, which is expected; the inner files were verified
instead).

## Screenshots (fresh, of this dashboard — not the starter's)

| File | View | Size | Dimensions |
| --- | --- | --- | --- |
| `screenshots/1-board.png` | Board, top of the ranking | 156,424 bytes | 1440×900 |
| `screenshots/2-detail.png` | Detail drawer, Sentora Curator (#1) with multiplier breakdown + farm plan | 213,768 bytes | 1440×900 |
| `screenshots/3-filtered.png` | Filtered view, Category = Risk Curators (34 rows) | 159,621 bytes | 1440×900 |

How they were taken: the bundled Chromium's `--screenshot` flag writes nothing
in this VM, so screenshots were captured over the DevTools protocol
(`shot.cjs`, stdlib-only Node script driving `Page.captureScreenshot` on
`http://127.0.0.1:9222`). Two environment quirks, both worked around in the
script and documented here: (1) this Chromium build refuses to start without a
WebGL domain policy — a policy file containing `hatch-webgl-all-origins-v1`
plus its sha256 is passed via `--hatch-webgl-allowed-domains-file` /
`--hatch-webgl-allowed-domains-sha256`; (2) navigating to `http://127.0.0.1`
is blocked by the browser's local-network-access check, so the shots use
`file://` URLs. Each shot navigates via `about:blank` first so the `#fragment`
state (detail / filter) actually loads — a fragment-only navigation does not
re-run the page. Probe output verified each view before capture (board: 232
rows; detail: dialog open; filtered: 34 rows).

## What still needs the user

Nothing in this entry submits anything. These steps need the user's login and
were **not** done:

1. **GitHub repo** — create a repo in the user's account (created after the
   competition opened, 2026-11-14T10:00:00Z), push this entry, add an
   OSI-approved license. The entry is ready to push as-is (`main.ts`,
   `lib/model.ts`, `vendor/`, `dashboard/`, `screenshots/`, plus the helper
   scripts `check-gates.py` and `shot.cjs`). The sealed snapshot lives in the
   kit folder, not in the entry — the submission's `main.ts` reads it from
   `/input/snapshot.json` at run time, so no snapshot file needs to be
   committed.
2. **make.fun sign-in** — sign in to make.fun, open the token-radar bounty,
   and submit the repo + commit + AI transcripts (max 5 submissions/day,
   10 total; last submission is final).
3. **Solana wallet signature** — the prize (1st $1,000 USDC) is claimed to a
   Solana wallet; the signature step needs the user's wallet.

Deadline: **Mon 14 Dec 2026 17:00 WIB** (2026-12-14T10:00:00Z). The "Bring the
humans" bounty ends Tue 17 Nov 2026 01:00 WIB and is not part of this entry.
