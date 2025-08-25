---
icon: radio-tuner
---

# News Reporter

Tenshiは、ウォッチ対象のプロフィールから出る**発表/声明**の文面を即時評価し、\
**1〜10のヒートスコア**を付与します。\
指定した範囲（例：**≥8 を超強気**、**≤2 を極弱気**）だけ通知することで、\
市場が反応する前の**爆発的アルファ**や**重大なレッドフラグ**を先取りします。

## プロンプト例

* 「**Tenshi**、(@projectnamehere) の公式発表を追跡して、“**超強気**”か“**極端に弱気**”のときだけ知らせて」

### 実装サンプル（TypeScript｜拡張しやすい堅牢版）

```ts
type Threshold = { bullish: number; bearish: number };
type ScoreDetail = { score: number; hits: { bullish: string[]; bearish: string[] } };

const defaultBullish = [
  /raised/i, /\$?\d+(\.\d+)?\s?m\b/i, /partner(ed|ship)?/i,
  /list(ed|ing)?/i, /coinbase|binance|kraken/i, /mainnet live|launch(ed)?/i,
  /integration|collaborat(e|ion)/i, /grants?|ecosystem fund/i
];

const defaultBearish = [
  /hack|exploit|breach/i, /resign|depart(ure)?|step(s)? down/i,
  /shut(ting)?\s?down|sunset(ting)?/i, /delay|postpone|paused?/i,
  /sec|regulator(y)? action/i, /bankrupt|insolven(t|cy)/i
];

function normalize(text: string) {
  return text.replace(/\s+/g, " ").trim();
}

function scoreAnnouncement(text: string, lists?: { bullish?: RegExp[]; bearish?: RegExp[] }): ScoreDetail {
  const t = normalize(text);
  const bullish = lists?.bullish ?? defaultBullish;
  const bearish = lists?.bearish ?? defaultBearish;

  let score = 5; // 中立 = 5
  const bHits: string[] = [];
  const rHits: string[] = [];

  for (const re of bullish) {
    if (re.test(t)) { score += 1.2; bHits.push(re.source); }
  }
  for (const re of bearish) {
    if (re.test(t)) { score -= 1.8; rHits.push(re.source); }
  }

  // 追加: 強い単語/弱い単語の重み補正（任意）
  if (/\bconfirmed|official\b/i.test(t)) score += 0.5;
  if (/\brumor|unconfirmed\b/i.test(t)) score -= 0.5;

  // 範囲クリップ
  score = Math.max(0, Math.min(10, score));

  return { score, hits: { bullish: bHits, bearish: rHits } };
}

// 簡易スロットリング & 重複防止
const recentlyAlerted = new Map<string, number>(); // key: tweetId -> ts

function shouldAlert(tweetId: string, coolDownMs = 60_000) {
  const now = Date.now();
  const last = recentlyAlerted.get(tweetId) ?? 0;
  if (now - last < coolDownMs) return false;
  recentlyAlerted.set(tweetId, now);
  return true;
}

type Tweet = { id: string; text: string; author: string; url: string; ts: number };

function monitorAndAlert(
  account: string,
  threshold: Threshold = { bullish: 8, bearish: 2 },
  lists?: { bullish?: RegExp[]; bearish?: RegExp[] }
) {
  onTweet(account, (tweet: Tweet) => {
    const { score, hits } = scoreAnnouncement(tweet.text, lists);

    if (score >= threshold.bullish || score <= threshold.bearish) {
      if (!shouldAlert(tweet.id)) return;

      const polarity = score >= threshold.bullish ? "BULLISH" : "BEARISH";
      sendAlert({
        polarity,
        score,
        author: tweet.author,
        link: tweet.url,
        excerpt: tweet.text.slice(0, 200),
        matched: hits
      });
    }
  });
}

// 例: プロジェクト公式発表のみ強検知
monitorAndAlert("@projectnamehere", { bullish: 8, bearish: 2 }, {
  bullish: [...defaultBullish, /roadmap v\d+|milestone/i, /enterprise|institutional/i],
  bearish: [...defaultBearish, /token unlock|mass unlock/i, /contract\s+migration/i]
});
```

#### 実運用メモ

* **語彙リストはプロジェクト別に上書き**（例：インフラ系は「integration」「RPC」「indexer」を強めに）
* しきい値は**市場ボラ**で可変（高ボラ時は 8/2 → 8.5/1.5 など）
* **多言語対応**が必要なら、事前に訳語辞書または埋め込みベースの類似検索を併用
* 誤検知対策に**引用RT/スレッド要約**を優先評価、**画像内テキストOCR**は別キューで追補
