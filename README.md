# AetherDesk Trading Bots

Multi-agent Solana memecoin trading desk built with Grok Bots.

**Paper mode only by default.** No real capital is at risk until explicitly enabled.

## Architecture

Nine specialized agents with strict role separation. Only the Final Checker can approve a trade.

| Bot | Role | Key Rules |
|-----|------|-----------|
| Pulse Scout | Discovery | Finds new Solana pairs / volume spikes |
| Forge Filter | Risk Gate | Rug checks (InsightX, Bubblemaps, DexScanner, RugCheck), holder concentration, wallet quality |
| Echo Sentinel | News Radar | X / news sentiment, kill switch on high-impact negative events |
| Thesis Weaver | Analysis | Structured thesis only (action, entry, stop, target, invalidation). Never sizes. |
| Atlas Chart | Technicals | Indicators for tokens with ≥30 days history |
| Aegis Risk | Capital Shield | Max 8% position, independent 2R minimum, post-stop cooldowns, daily loss limit |
| Gatekeeper | Final Gate | Consensus check across all bots before any trade |
| Afterglow | Post-Trade | Analyzes closed trades and suggests improvements |
| Vector | Execution | Jupiter swaps (paper by default). Network health gate prepared. |

## Shared Status Files

Bots write and read shared JSONL files so they do not have to wait for chat messages:
- `/workspace/filter/latest.jsonl`
- `/workspace/risk/latest.jsonl`
- Additional files per bot

## Hard Risk Rules
- Maximum position size: 8% of stack
- Hard stop: –6%
- Independent risk/reward check: reject below 2R
- No averaging down
- Post-stop cooldown: 20 min same token + 5 min global
- Daily loss limit and kill switch from negative news

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

Please open an issue or PR. All changes should preserve the strict role separation and paper-mode default.

**Disclaimer:** Trading cryptocurrencies involves substantial risk of loss. This system is for research and paper trading only unless you explicitly enable live mode with your own funds and accept total loss risk.
