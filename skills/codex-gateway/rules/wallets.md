# Wallet Reference

MPP challenges require a funded wallet to sign payment credentials. Which tool to use depends on the network the challenge specifies.

| Network | Tool | Auth method | Currency |
| ------- | ---- | ----------- | -------- |
| Tempo (chain 4217) | `tempo` (`wallet` + `request` extensions) | Passkey via `tempo wallet login` | USDC |

Pick the tool that matches the network in the `WWW-Authenticate` challenge, then follow the corresponding section below.

---

## Tempo — tempo wallet / tempo request

`tempo` is the Tempo CLI. The `wallet` extension manages identity and funding; the `request` extension is an HTTP client that handles the full MPP 402-challenge flow automatically — it detects the challenge, signs a Tempo transaction, and retries the request in one step.

### Install

```bash
curl -fsSL https://tempo.xyz/install | bash
tempo add wallet
```

### Setup

```bash
tempo wallet login
```

This opens a passkey-based auth flow. No private keys to manage.

### Preflight check

```bash
tempo wallet -t whoami
```

Confirm `ready` is `true` and `balance.available` has sufficient USDC before making paid requests.

### Making a paid request

You MUST include the `X-Codex-Payment: mpp` header — `tempo request` does not add it for you:

```bash
tempo request -t -X POST \
  -H "X-Codex-Payment: mpp" \
  -H "Content-Type: application/json" \
  --json '{"query":"query { getNetworks { id name } }"}' \
  https://graph.codex.io/graphql
```

`tempo request` handles the 402 challenge and payment automatically, but only if the server returns a 402. Without the MPP header, the server returns 401 instead.

### Session management

Sessions open a payment channel on-chain once, then use off-chain vouchers for subsequent requests (no gas per request):

```bash
tempo wallet -t sessions list          # view active sessions
tempo wallet -t sessions close --all   # close all sessions when done
```

### Recovery

| Symptom | Action |
| ------- | ------ |
| `No wallet configured` | Run `tempo wallet login` |
| `Insufficient balance` | Tell the user — wallet needs funding |
| `Spending limit exceeded` | Tell the user — key limit reached |

---

## Rules

- **Never print, log, or read private keys.** Tools handle key management internally.
- **Always run a preflight check** before attempting paid requests.
- **If auth fails, fix it automatically** — run `tempo wallet login` as appropriate, then retry.
- **Do not mix tools.** Use tempo for Tempo challenges.
