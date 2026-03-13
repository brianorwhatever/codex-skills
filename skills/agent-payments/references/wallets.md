# Wallet Reference

MPP challenges require a funded wallet to sign payment credentials. Which tool to use depends on the network the challenge specifies.

| Network | Tool | Auth method | Currency |
| ------- | ---- | ----------- | -------- |
| Tempo (chain 4217) | `presto` | Passkey via `presto login` | USDC |
| Base | `awal` (Coinbase Agentic Wallet) | Email OTP via `npx awal auth login` | USDC |

Pick the tool that matches the network in the `WWW-Authenticate` challenge, then follow the corresponding section below.

---

## Tempo — presto

presto is a CLI HTTP client that handles the full MPP 402-challenge flow automatically: it detects the challenge, signs a Tempo transaction, and retries the request in one step.

### Install

```bash
curl https://presto-binaries.tempo.xyz/install.sh | bash
```

### Setup

```bash
presto login
```

This opens a passkey-based auth flow. No private keys to manage.

### Preflight check

```bash
presto -q --output-format json whoami
```

Confirm `ready` is `true` and `key.balance` has sufficient USDC before making paid requests.

### Making a paid request

```bash
presto -q -X POST \
  --json '{"query":"query { getNetworks { id name } }"}' \
  https://graph.codex.io/graphql | jq
```

presto handles the 402 challenge and payment automatically.

### Session management

presto can open payment channels for multiple requests to the same origin, avoiding per-request gas:

```bash
presto session list          # view active sessions
presto session close --all   # close all sessions when done
```

### Recovery

| Symptom | Action |
| ------- | ------ |
| `No wallet configured` | Run `presto login` |
| `Insufficient balance` | Tell the user — wallet needs funding |
| `Spending limit exceeded` | Tell the user — key limit reached |

---

## Base — awal

awal is Coinbase's Agentic Wallet CLI for Base. It holds USDC, sends payments, trades tokens, and handles x402 paid API requests — all without the agent touching private keys.

### Setup

Authentication uses email OTP:

```bash
npx awal auth login user@example.com
# sends a 6-digit code to the email

npx awal auth verify <flowId> <otp>
# completes sign-in
```

### Preflight check

```bash
npx awal status --json
```

Confirm the wallet is authenticated and has funds:

```bash
npx awal balance --json
```

### Making a paid request (x402)

```bash
npx awal x402 pay <url>
```

For discovering available paid services:

```bash
npx awal x402 bazaar search <query>
```

### Other operations

| Command | Purpose |
| ------- | ------- |
| `npx awal address` | Get wallet address |
| `npx awal send <amount> <to> [--chain]` | Send USDC (supports ENS) |
| `npx awal trade <amount> <from> <to>` | Swap tokens on Base |
| `npx awal balance [--chain]` | Check USDC balance |

All commands accept `--json` for machine-readable output.

### Recovery

| Symptom | Action |
| ------- | ------ |
| Not authenticated | Run `npx awal auth login <email>` then verify OTP |
| Insufficient balance | Tell the user — wallet needs funding on Base |
| Server not running | Run `npx awal status` to check health |

---

## Rules

- **Never print, log, or read private keys.** Both tools handle key management internally.
- **Always run a preflight check** before attempting paid requests.
- **If auth fails, fix it automatically** — run `presto login` or `npx awal auth login` as appropriate, then retry.
- **Do not mix tools.** Use presto for Tempo challenges, awal for Base challenges.
