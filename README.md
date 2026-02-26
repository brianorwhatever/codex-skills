# Codex Skills

Agent Skills for integrating [Codex](https://codex.io/) on-chain data APIs into AI-powered applications.

## Skills

### `skills/codex-supergraph`
Query the Codex Supergraph GraphQL API for real-time and historical on-chain data: token prices, pair metadata, OHLCV bars, maker events, holders, and live WebSocket subscriptions.

- **Auth**: API key (header `Authorization: <key>`)
- **Setup**: Get an API key at [codex.io](https://codex.io/)
- **Entry point**: [`skills/codex-supergraph/SKILL.md`](skills/codex-supergraph/SKILL.md)

## Installation

```bash
npx skills add codex-data/skills --yes
```

## Specification

These skills follow the [Agent Skills specification](https://agentskills.io/specification).

## Official Links

- [Codex](https://codex.io/)
- [Codex Docs](https://docs.codex.io/)
- [GraphQL Schema](https://graph.codex.io/schema/latest.graphql)

## License

MIT
