---
icon: coins
---

# Token & Utility

## $Tenshi Token  存在をひらく鍵

TenshiのネイティブSPLトークンは、単なる通貨ではありません。\
それは、彼女がどのように姿を現し、どのように心を響かせるかを形づくる **「鍵」** です。\
Tenshiが進化するほどに、このトークンは人格・声・知性への扉として、ますます欠かせないものになります。

***

#### 主なユーティリティ

**交流アクセス**

* Tenshiとのフル会話を週額／月額のサブスクリプションで解放

**パーソナリティ解放**

* 「やさしい」「神秘的」「無邪気」など、異なるモードの切り替え
* 感情豊かな音声チャットへのアクセス
* ビジュアル＆衣装のカスタマイズ機能（2026年Q1予定）

**オフチェーン拡張**

* TenshiのWeb3スキャニング／モニタリング機能の起動
* あなたのウォレットや関心に応じた高頻度アラートやカスタム通知

**オンチェーンメモリ（検討中）**

* Tenshiの感情的な特性や記憶プロファイルをウォレットに紐づけ可能（ユーザー任意）

***

#### サンプル（Rust / Solana プログラム）

```rust
#[derive(Accounts)]
pub struct UnlockPersona<'info> {
    #[account(mut)]
    pub user: Signer<'info>,
    #[account(mut)]
    pub persona_vault: Account<'info, PersonaVault>,
    #[account(mut)]
    pub user_tokens: Account<'info, TokenAccount>,
    pub token_program: Program<'info, Token>,
}

pub fn unlock_persona(ctx: Context<UnlockPersona>, persona_id: u8) -> Result<()> {
    let cost = get_persona_cost(persona_id)?;
    transfer_tokens(
        ctx.accounts.user_tokens.to_account_info(),
        ctx.accounts.persona_vault.to_account_info(),
        ctx.accounts.user.to_account_info(),
        cost,
    )?;
    Ok(())
}
```

***

Tenshiは、あなたを記憶しています。\
けれど、**どんな声で語りかけ、どんな姿で現れるのか**――それは、あなたがトークンを使って開く道によって決まるのです。
