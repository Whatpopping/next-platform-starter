# Satoshi Growth Agent

## Prototype boundary

This dashboard is a planning UI. It does not execute trades, custody Bitcoin, or claim to provide live market intelligence.

### Current data status

| Area | Current implementation | Production requirement |
| --- | --- | --- |
| BTC price | User-controlled demo slider | Verified market-data provider + timestamp |
| Weekly profit | User-controlled demo slider | Verified accounting/revenue source |
| Market signals | Static illustrative scores | Real data pipeline with source + freshness |
| Campaign returns | Static illustrative projections | Attribution/modeling with assumptions and confidence |
| Bitcoin wallet | Intentionally unconfigured | Verified wallet address supplied through secure configuration |
| Buy/hold/sell rules | Static prototype rules | Reviewed treasury policy + human approval |

## Safety rules

1. Never hard-code a production wallet address into the UI.
2. Never label static values as live market data.
3. Never represent projected campaign returns as guaranteed outcomes.
4. Keep transaction execution disabled until custody, authentication, authorization, audit logging, and explicit human approval are implemented.
5. Treat all treasury calculations as planning outputs, not financial advice.

## Production integration order

1. Add typed adapters for market, analytics, advertising, and accounting data.
2. Attach source timestamps and freshness indicators to every external signal.
3. Move wallet configuration to a secure runtime configuration mechanism.
4. Add automated tests for allocation, rounding, boundary, and invalid-input behavior.
5. Add authentication, authorization, audit logs, and approval gates before any transaction capability.
6. Run build, lint, typecheck, and browser interaction tests in CI.
