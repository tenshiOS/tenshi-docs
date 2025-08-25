---
icon: lock
---

# Ticker/Contact Address Monitor

Tenshiは、あなたが指定したアカウントやウォッチリストを静かに見張り、\
**ティッカー（例：$TRUMP, $BONK）やコントラクトアドレス（CA）の投稿を即座に拾い上げます。**\
**検知すると、チャットやメール**へ最短ルートで知らせます。

#### できること

* **ティッカー/CAの自動検出**（ETH/EVM系・Solana対応）
* **任意アカウント/ユーザー定義ウォッチリスト**からリアルタイム監視
* **通知チャネル選択**（アプリ内DM / メール / Webhook 等）

## プロンプトの例

* 「**Tenshi**、JS と Cooker が **コントラクトアドレス**や**ティッカー**を投稿したら知らせて」
* 「私のウォッチリストから **$BONK** や **新規CA** が出たら、要約つけてメールして」

***

### 実装サンプル（TypeScript｜堅牢版）

* EVM系CA検出（`0x` + 40 hex）
* Solanaアドレス検出（Base58/32–44桁、よくある誤字を抑制）
* ティッカー検出（`$A…Z` 2–8桁、終端境界）
* 重複アラート抑制・誤検知緩和・簡易サマリ同梱

```ts
// --- Detection primitives ---
const tickerRegex = /\$[A-Z]{2,8}\b/g; // 2-8文字程度に拡張
const evmCaRegex = /\b0x[a-fA-F0-9]{40}\b/g;
// Solana: Base58 (0OIl除外を強めに意識). 実運用では bs58 検証推奨
const solBase58 = "[1-9A-HJ-NP-Za-km-z]";
const solCaRegex = new RegExp(`\\b${solBase58}{32,44}\\b`, "g");

function extractSignals(text: string) {
  const tickers = Array.from(new Set((text.match(tickerRegex) || []).map(s => s.toUpperCase())));
  const evm = Array.from(new Set(text.match(evmCaRegex) || []));
  const sol = Array.from(new Set(text.match(solCaRegex) || []));
  return { tickers, cas: [...evm, ...sol] };
}

function looksLikeNoise(text: string) {
  // よくあるスパム/無関係ワードで簡易フィルタ
  return /(giveaway|airdrop scam|promo code|free mint)/i.test(text);
}

function summarize(tweetText: string, { tickers, cas }: { tickers: string[]; cas: string[] }) {
  const head = tweetText.split(/\n|\. |\! |\? /)[0].slice(0, 180);
  return `${head} …  | tickers: ${tickers.join(", ") || "-"} | CA: ${cas.join(", ") || "-"}`;
}

// --- Watch + Alert pipeline ---
type Account = string; // "@handle"
type Tweet = { id: string; text: string; url: string; author: string; ts: number };

const seen = new Set<string>();

async function onTweet(tweet: Tweet) {
  if (seen.has(tweet.id) || looksLikeNoise(tweet.text)) return;
  const signals = extractSignals(tweet.text);
  if (signals.tickers.length === 0 && signals.cas.length === 0) return;

  seen.add(tweet.id);
  const payload = {
    author: tweet.author,
    link: tweet.url,
    detected: signals,
    summary: summarize(tweet.text, signals),
    ts: tweet.ts
  };
  // 通知先はユーザー設定に応じて分岐
  notifyChat(payload);
  notifyEmail?.(payload); // メール連携が有効なら
}

function watchAccounts(accounts: Account[]) {
  const stream = createTwitterStream({ accounts }); // 実装は利用APIに依存
  stream.on("tweet", onTweet);
}

// --- Example: user-defined watchlist ---
watchAccounts(["@ShockedJS", "@CookerFlips"]);
```
