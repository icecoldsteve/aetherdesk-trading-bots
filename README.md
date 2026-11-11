# AetherDesk Trading Bots

Multi-agent Solana memecoin trading desk built with Grok Bots.

**Paper mode only by default.** No real capital is at risk until explicitly enabled.

## Architecture

Nine specialized agents with strict role separation. Only the Final Checker can approve a trade.

| Bot | Role | Key Rules |
|-----|------|-----------|
| Pulse Scout | Discovery | Finds new Solana pairs / volume spikes |
| Forge Filter | Risk Gate | Rug checks (InsightX, Bubblemaps, DexScanner, RugCheck), holder concentration, wallet quality, **GMGN-style smart-money / whale cluster detection** |
| Echo Sentinel | News Radar | X / news sentiment, kill switch on high-impact negative events |
| Thesis Weaver | Analysis | Structured thesis only (action, entry, stop, target, invalidation). Never sizes. |
| Atlas Chart | Technicals | Indicators for tokens with ≥30 days history |
| Aegis Risk | Capital Shield | Max 8% position, independent 2R minimum, post-stop cooldowns, daily loss limit |
| Gatekeeper | Final Gate | Consensus check across all bots before any trade |
| Afterglow | Post-Trade + Self-Improve | Grades closed trades, proposes prompt diffs for non-risk bots, versions with rollback. Human review required. |
| Vector | Execution | Jupiter swaps (paper by default). Network health gate prepared. |

## Shared Status Files

Bots write and read shared JSONL files so they do not have to wait for chat messages:
- `/workspace/filter/latest.jsonl`
- `/workspace/risk/latest.jsonl`
- `/workspace/afterglow/trades.jsonl` (closed trades + scores)
- `/workspace/afterglow/prompts/` (versioned prompts + win-rate metadata)
- Additional files per bot

### Filter status schema extension (whale / smart-money)
Each filter result should include:
```json
{
  "token": "...",
  "rug_score": 0-100,
  "wallet_quality": "pass|fail|caution",
  "smart_money_cluster": {
    "wallets_buying": 0,
    "same_block_or_window": true/false,
    "top_wallets_selling": true/false,
    "source": "gmgn|fallback|none"
  },
  "verdict": "pass|reject|review"
}
```

## Hard Risk Rules
- Maximum position size: 8% of stack
- Hard stop: –6%
- Independent risk/reward check: reject below 2R
- No averaging down
- Post-stop cooldown: 20 min same token + 5 min global
- Daily loss limit and kill switch from negative news

**These rules are immutable.** Afterglow may never auto-edit Aegis Risk, Gatekeeper, or Vector risk/execution limits.

## Self-Improvement Loop (Afterglow)

Inspired by closed-loop grading + coaching patterns, implemented safely for paper trading:

1. After every closed paper trade, Afterglow scores it 0–1 on a fixed rubric:
   - Followed the plan / consensus
   - Size within hard limits
   - Exited on time (stop / target / invalidation)
   - Did not chase
   - Process quality (separate from raw P&L)
2. Score < 0.9 → Coach proposes a 1–2 line prompt rewrite **only** for Scout, Filter, Echo Sentinel, Thesis Weaver, or Atlas Chart.
3. Proposed changes are written to `/workspace/afterglow/proposals/` and require explicit human approval before any prompt is updated.
4. Every approved prompt version is stored with its associated paper win rate / expectancy. If a new version underperforms the previous on a rolling window of trades, Afterglow recommends rollback.
5. Morning report: paper balance, win rate, expectancy, and the 3 biggest proposed (or applied) prompt changes.

The loop never changes the scoring rubric or any hard risk limits.

## Whale / Smart-Money Cluster Detection (Forge Filter)

Added as a non-blocking enrichment on top of existing rug and wallet-quality checks (does **not** override hard risk rules):

- Prefer GMGN Skills / GMGN OpenAPI (or equivalent public feeds) for smart-money and whale labels.
- Flag a positive cluster when ≥3 labeled smart-money or high-PnL wallets buy the same token within a short window (same block or ≤2 minutes).
- Flag a negative signal when top holders or previously-buying smart-money wallets begin unloading.
- Down-score or reject if a large portion of top holders show negative recent realized PnL (existing wallet-quality rule).
- Fallback: if GMGN unavailable, use public holder lists + basic clustering heuristics or skip the signal (never block solely on missing data).
- All signals are written to the shared filter status file for Gatekeeper and Afterglow to consume.
- Afterglow scores the quality of these signals in post-trade review; weak or noisy signals can be tuned via the proposal loop.

## Current Mode
- 7-day paper validation running on a $10,000 virtual stack
- Live trading and autonomous mode are disabled
- Hot wallet prepared but empty (public address available on request)

## Dashboard
Live sci-fi command center:  
https://aetherdesk-dashboard-smsak.vercel.app/

Source: https://github.com/icecoldsteve/aetherdesk-dashboard

## Contributing
This is an open architecture. Contributions welcome for:
- Better rug-check sources or InsightX integration
- Additional technical indicators for Atlas Chart
- Improved news sentiment scoring
- Execution safety improvements
- Dashboard enhancements
- Refined Afterglow scoring rubrics or proposal formats
- Improved GMGN or alternative smart-money data integrations

Please open an issue or PR. All changes should preserve the strict role separation, paper-mode default, and immutable hard risk rules.

**Disclaimer:** Trading cryptocurrencies involves substantial risk of loss. This system is for research and paper trading only unless you explicitly enable live mode with your own funds and accept total loss risk.
