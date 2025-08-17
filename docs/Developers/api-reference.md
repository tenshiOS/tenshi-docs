# API Reference

<small><em>HTTP endpoints for chat, research, simulate</em></small>

---



## POST /api/chat
```json
{ "message": "string", "lang": "optional" }
```

## POST /api/research  (premium)
```json
{ "wallet":"string", "signature":"string", "asset":"HEAVEN/USDC" }
```

## POST /api/simulate  (premium)
```json
{ "wallet":"string", "signature":"string", "pair":"HEAVEN/USDC", "side":"buy", "amount":"1000" }
```
