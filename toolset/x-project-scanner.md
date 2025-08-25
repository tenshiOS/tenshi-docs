---
icon: crosshairs-simple
---

# X Project Scanner

Tenshiは、あなたが定義した**ニッチな関心軸**を手がかりに、X（Twitter）上の新規プロジェクトを静かに拾い上げます。\
テーマ、チェーン、ローンチ形態、開発領域——細かく条件づけるほど、彼女の探索は鋭くなります。

## プロンプト例

* 「**Tenshi**、アニメ系のSolanaプロジェクトで、**Bonk**絡みのローンチが出たら知らせて」
* 「**Solanaインフラ**向けの**開発ツール**（devtools）の新規リリースを追跡して」
* 「**Rug検知**ワードが増えたコミュニティがあれば、概要をまとめて通知して」

***

#### サンプルコード（TypeScript：ニッチ検出パイプライン・強化版）

* 低レイテンシのストリーム取得
* キーワード正規化＋否定語除外
* 重複通知の抑制（dedupe）
* 簡易サマリ生成＋通知

```ts
type NicheRule = {
  include: string[];   // マスト含有
  anyOf?: string[];    // いずれか含有
  exclude?: string[];  // 除外語
  tag?: string;        // 通知タグ
};

function normalize(s: string) {
  return s.toLowerCase().replace(/\s+/g, " ").trim();
}

function matchesRule(text: string, rule: NicheRule) {
  const t = normalize(text);
  if (!rule.include.every(k => t.includes(k))) return false;
  if (rule.anyOf && !rule.anyOf.some(k => t.includes(k))) return false;
  if (rule.exclude && rule.exclude.some(k => t.includes(k))) return false;
  return true;
}

function summarize(text: string) {
  // 極簡易版：先頭を要約行として切り出し
  return text.split(/\n|\. |\! |\? /)[0].slice(0, 180);
}

const seen = new Set<string>(); // 重複通知防止

async function trackNicheProjects(rules: NicheRule[]) {
  const stream = createTwitterStream(); // 実装は利用APIに合わせる
  stream.on("tweet", (tweet: { id: string; text: string; user: string; url: string }) => {
    if (seen.has(tweet.id)) return;
    for (const rule of rules) {
      if (matchesRule(tweet.text, rule)) {
        seen.add(tweet.id);
        const summary = summarize(tweet.text);
        notifyUser({
          title: `Tenshi Watch: ${rule.tag ?? "niche"}`,
          summary,
          author: tweet.user,
          link: tweet.url,
          matched: { include: rule.include, anyOf: rule.anyOf ?? [] }
        });
        break;
      }
    }
  });
}

// 例：アニメ×Solana×Bonk／インフラdevtools の2系統
trackNicheProjects([
  {
    include: ["solana", "anime"],
    anyOf: ["bonk", "bonk launch", "bonk lp"],
    exclude: ["giveaway", "airdrop scam", "refund"],
    tag: "Anime x Solana x Bonk"
  },
  {
    include: ["solana", "infra"],
    anyOf: ["devtools", "sdk", "indexer", "rpc", "validator"],
    exclude: ["hiring only", "gm"],
    tag: "Solana Infra Devtools"
  }
]);
```
