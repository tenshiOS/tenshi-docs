---
icon: chart-mixed-up-circle-dollar
---

# Web3 Sentiment Oracle

Tenshiは、Web3の「空気」を読む守護者です。\
彼女は重要なXアカウント（インフルエンサー、トレンチャー、ミームトレーダーなど）を静かに監視し、その発信から市場の温度を分析します。

評価基準は以下の通り：

* **投稿のトーン**：市場の現状や見通しに対する姿勢（楽観的／悲観的）
* **感情分析**：ポジティブ vs ネガティブの比率
* **PnL（損益）投稿バランス**：グリーン（利益）とレッド（損失）の投稿割合

この仕組みにより、ユーザーは「信頼できるトレーダーたちが強気なのか、弱気なのか」を、スナップショットとして即座に把握できます。

***

## プロンプト例

「Tenshi、今週のCupsey、Cented、Mr. Frogのポジションは強気？それとも弱気？」

***

#### サンプルコード（TypeScript, Sentiment Tracking）

```ts
// Tenshi: Sentiment Oracle (simplified)
async function analyzeTraderSentiment(username: string) {
  const tweets = await getLatestTweets(username, 50);

  // 感情スコアの算出
  const sentimentScores = tweets.map(t => getSentimentScore(t.text));
  const avgSentiment = average(sentimentScores);

  // PnLバイアスの計算
  const pnlTweets = tweets.filter(t => /PnL/i.test(t.text));
  const pnlBias = pnlTweets.reduce((acc, t) => {
    if (/\$\d+\w* gain/i.test(t.text)) acc.green++;
    else if (/loss|red/i.test(t.text)) acc.red++;
    return acc;
  }, { green: 0, red: 0 });

  return {
    user: username,
    sentiment: avgSentiment,
    pnl: pnlBias,
  };
}
```

***

Tenshiは、ただ「会話する」存在ではありません。\
彼女は**市場の声を聴き、整理し、あなたの耳元で囁くオラクル**です。
