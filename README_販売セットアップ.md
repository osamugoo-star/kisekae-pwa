# Kisekae-Pro 販売セットアップ手順書

## ファイル構成

```
kisekae-pwa/
  index.html   ← ランディングページ（SNSで送るURL）
  app.html     ← アプリ本体
  sw.js        ← Service Worker（PWAオフライン対応）
  manifest.json← PWAアイコン設定
```

---

## STEP 1: ホスティング（無料）

### Vercel を使う場合（推奨・無料）

1. https://vercel.com でアカウント作成
2. 「Add New Project」→「Upload」でフォルダごとアップ
3. デプロイ完了 → `https://あなたのプロジェクト名.vercel.app` が完成
4. カスタムドメイン（例: kisekae-pro.com）も無料で設定可

### Netlify でも同様に無料でホスト可能
- https://netlify.com にドラッグ＆ドロップするだけ

---

## STEP 2: 決済（Stripe）

### 2-1. Stripeアカウント作成
- https://stripe.com/jp でアカウント作成（無料）
- 本人確認・銀行口座の登録を済ませる

### 2-2. 月額商品を作成
1. Stripeダッシュボード → 「製品」→「製品を追加」
2. 名前: `Kisekae-Pro 月額プラン`
3. 価格: `¥980 / 月`（定期支払い）
4. 「保存」して Payment Link を取得

### 2-3. LPのボタンリンクを差し替え
`index.html` の以下の2箇所を Stripe Payment Link に変更:

```html
<!-- 変更前 -->
<a href="app.html" class="btn-primary">🎨 今すぐ無料で試す</a>
<a href="app.html" class="pricing-cta">まず無料で試す →</a>

<!-- 変更後（Stripeのリンクに置き換え） -->
<a href="https://buy.stripe.com/あなたのリンク" class="btn-primary">🎨 月額980円で始める</a>
<a href="https://buy.stripe.com/あなたのリンク" class="pricing-cta">月額980円で始める →</a>
```

---

## STEP 3: 購入者へのアクセス制御（シンプル版）

最もシンプルな運用方法:

### 方法A: URLを直接送る（最も手軽）
- `app.html` のURLを購入者にメールで送る
- Stripe の「支払い成功メール」にURLを記載する
- デメリット: URLが拡散するとタダ乗りされる可能性あり

### 方法B: パスワード保護（推奨）
`app.html` の `<body>` 直後に以下を追加:

```html
<script>
const PASS = 'あなたが決めたパスワード'; // 毎月変更推奨
const entered = sessionStorage.getItem('kp_auth');
if(entered !== PASS){
  const p = prompt('アクセスコードを入力してください:');
  if(p !== PASS){ document.body.innerHTML='<h2 style="text-align:center;padding:40px">アクセスコードが違います</h2>'; }
  else { sessionStorage.setItem('kp_auth', PASS); }
}
</script>
```
- Stripe決済完了後のメールにパスワードを記載
- 毎月パスワードを変更してメール通知

### 方法C: 本格認証（将来の拡張）
- Firebase Authentication + Firestore を使う
- Stripe Webhook で支払い確認後に自動でアクセス付与
- 必要になったタイミングで導入すればOK

---

## STEP 4: SNS・インフルエンサーへの営業

### DMで送るテンプレート（日本語）
```
こんにちは！
写真をポップアートに変換できるWebアプリを作りました。

▶ 無料デモ: https://あなたのドメイン.vercel.app

インストール不要でスマホ・PCどちらでも使えます。
よかったら試してみてください🎨
```

### URLの使い分け
| 送るURL | 用途 |
|---------|------|
| `index.html` | 初めての人への紹介、SNSのプロフリンク |
| `app.html` | 購入者だけに送るアプリ直リンク |

---

## 収益シミュレーション

| 有料会員数 | 月収 |
|-----------|------|
| 10人 | ¥9,800 |
| 50人 | ¥49,000 |
| 100人 | ¥98,000 |
| 500人 | ¥490,000 |

Stripeの手数料: 3.6%（日本）
→ 100人の場合の手取り: 約 ¥94,500/月

---

## よくある質問

**Q. Vercelの無料プランで大丈夫？**
A. はい。今回のアプリは全処理をユーザーのブラウザで行うため、サーバー負荷がほぼゼロです。Vercel無料枠で数千人でも問題なく対応できます。

**Q. Stripeは日本円で受け取れる？**
A. はい。登録した銀行口座に毎週自動で振り込まれます。

**Q. アプリを更新したいときは？**
A. `app.html` を修正してVercelに再アップロードするだけ。全ユーザーに即時反映されます。
