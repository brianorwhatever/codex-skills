# Codex Supergraph Query Templates

## 1) API-key query over HTTPS

```bash
curl -sS https://graph.codex.io/graphql \
  -H 'Content-Type: application/json' \
  -H "Authorization: $CODEX_API_KEY" \
  --data-binary @- <<'JSON'
{
  "query": "query GetNetworks { getNetworks { id name } }"
}
JSON
```

## 2) API-key query with variables

```bash
curl -sS https://graph.codex.io/graphql \
  -H 'Content-Type: application/json' \
  -H "Authorization: $CODEX_API_KEY" \
  --data-binary @- <<'JSON'
{
  "query": "query GetTokenPrices($inputs: [GetPriceInput!]!) { getTokenPrices(inputs: $inputs) { address networkId priceUsd timestamp } }",
  "variables": {
    "inputs": [
      { "address": "So11111111111111111111111111111111111111112", "networkId": 1399811149 }
    ]
  }
}
JSON
```

## 3) Token discovery (`filterTokens`)

```graphql
query FilterTokens(
  $filters: TokenFilters
  $rankings: [TokenRanking]
  $limit: Int
) {
  filterTokens(filters: $filters, rankings: $rankings, limit: $limit) {
    results {
      buyVolume24
      sellVolume24
      circulatingMarketCap
      liquidity
      txnCount24
      token {
        info {
          address
          name
          symbol
        }
      }
    }
  }
}
```

## 4) Pair metadata (`pairMetadata`)

```graphql
query PairMetadata($pairId: String!) {
  pairMetadata(pairId: $pairId) {
    id
    pairAddress
    networkId
    liquidity
    price
    priceChange24
    volume24
    token0 {
      address
      symbol
      name
    }
    token1 {
      address
      symbol
      name
    }
  }
}
```

## 5) Pair bars (`getBars`)

```graphql
query GetBars(
  $symbol: String!
  $from: Int!
  $to: Int!
  $resolution: String!
  $countback: Int
  $removeEmptyBars: Boolean
) {
  getBars(
    symbol: $symbol
    from: $from
    to: $to
    resolution: $resolution
    countback: $countback
    removeEmptyBars: $removeEmptyBars
  ) {
    o
    h
    l
    c
    volume
  }
}
```

## 6) Single-token realtime (`onPriceUpdated`)

```graphql
subscription OnPriceUpdated($address: String!, $networkId: Int!) {
  onPriceUpdated(address: $address, networkId: $networkId) {
    address
    networkId
    priceUsd
    timestamp
    blockNumber
  }
}
```

## 7) Multi-token realtime (`onPricesUpdated`)

```graphql
subscription OnPricesUpdated($input: [OnPricesUpdatedInput!]!) {
  onPricesUpdated(input: $input) {
    address
    networkId
    priceUsd
    timestamp
    blockNumber
  }
}
```

## 8) WebSocket client (`graphql-ws`)

```typescript
import { createClient } from "graphql-ws";

const client = createClient({
  url: "wss://graph.codex.io/graphql",
  connectionParams: {
    Authorization: process.env.CODEX_API_KEY!,
  },
});

const unsubscribe = client.subscribe(
  {
    query: `
      subscription OnPriceUpdated($address: String!, $networkId: Int!) {
        onPriceUpdated(address: $address, networkId: $networkId) {
          address
          networkId
          priceUsd
          timestamp
          blockNumber
        }
      }
    `,
    variables: {
      address: "So11111111111111111111111111111111111111112",
      networkId: 1399811149,
    },
  },
  {
    next: (msg) => console.log(msg),
    error: (err) => console.error(err),
    complete: () => console.log("done"),
  }
);
```
