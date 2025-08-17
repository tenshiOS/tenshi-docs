# Data Indexing (The Graph)

<small><em>Subgraphs as the source of on‑chain truth</em></small>

---



```graphql
query PairSnapshot($pair: ID!, $from: Int!) {
  pair(id: $pair) {
    id
    token0 { symbol decimals }
    token1 { symbol decimals }
    reserveUSD
    volumeUSD24h: volumeUSD (where:{timestamp_gte:$from})
    feeTier
    txCount
  }
}
```
**Normalization** → humanize units from decimals; compute depth buckets (1k/5k/10k) for impact.  
**Auditability** → sign and include a digest of the raw GraphQL response in replies.
