# インフラ構成

> このサイトが「いまどうなっているか」を引くための資料です。
> 変更の経緯は [CHANGELOG.md](./CHANGELOG.md) を見てください。
> 最終更新: 2026-10-15

## 全体像

```
GitHub (umezawa-gts/dailyfit-site, public)
   │  main へ push
   ▼
GitHub Pages  ──────────►  https://dailyfit.jp
   ▲
   └ DNS: お名前.com の A レコードが GitHub を指している
```

ビルドは行っていません。リポジトリ直下のファイルがそのまま配信されます。

## デプロイ方法

**`main` に push すれば自動で本番に反映されます。** 他に操作は要りません。

反映状況は Settings ▸ Pages、またはリポジトリの Actions タブで確認できます。通常1〜2分です。

## URL の構造

ファイルはリポジトリ直下に平置きです。GitHub Pages は `.html` を省略したURLでも配信します。

| ファイル | 公開URL |
|---|---|
| `index.html` | `/` |
| `fudomae.html` | `/fudomae` |
| `shirokanetakanawa.html` | `/shirokanetakanawa` |
| `privacy.html` | `/privacy` |
| `reschedule.html` | `/reschedule.html` |

**正規URLは拡張子なしの方です。** 各ページの `<link rel="canonical">` がそれを宣言しています。
サイト内リンクを書くときも拡張子なし（`href="/fudomae"`）で統一してください。`/fudomae.html` でもページは開きますが、canonical と食い違い、GA4・Clarity の集計が2つに割れます。

## DNS（お名前.com）

ネームサーバーは `01〜04.dnsv.jp`（お名前.com）のままです。

| ホスト | TYPE | 値 | 用途 |
|---|---|---|---|
| `dailyfit.jp` | A | `185.199.108.153` | GitHub Pages |
| `dailyfit.jp` | A | `185.199.109.153` | GitHub Pages |
| `dailyfit.jp` | A | `185.199.110.153` | GitHub Pages |
| `dailyfit.jp` | A | `185.199.111.153` | GitHub Pages |
| `www` | CNAME | `umezawa-gts.github.io` | www → apex へ転送される |
| `dailyfit.jp` | MX | `smtp.google.com.`（優先1） | **Google Workspace のメール受信** |
| `google._domainkey` | TXT | `v=DKIM1;...` | **メールのDKIM署名** |
| `dailyfit.jp` | TXT | `google-site-verification=...` | **Search Console の所有権確認** |
| `_github-pages-challenge-umezawa-gts` | TXT | `c2ed7aed...` | **GitHub のドメイン所有権確認** |

### 触ってはいけないレコード

**MX / DKIM / 2つの TXT は消さないでください。**

- **MX を消すとメールが止まります**（`@dailyfit.jp` 宛が届かなくなる）
- **DKIM を消すとメールが迷惑メール判定されやすくなります**
- **google-site-verification を消すと Search Console のプロパティが外れます**
- **`_github-pages-challenge-...` を消すとドメイン所有権が外れ、カスタムドメインが使えなくなります**

A レコードと www の CNAME 以外は、原則として変更不要です。

### 切り戻し手順（Netlify へ戻す場合）

Netlify のプロジェクト `stately-melba-60a0e8` は残してあります。

1. A レコードを `75.2.60.5` の1件に戻す
2. `www` の CNAME を `stately-melba-60a0e8.netlify.app` に戻す

反映まで最大1時間かかります。

## 計測タグ

| ツール | ID | 設置場所 |
|---|---|---|
| Google タグマネージャー | `GTM-52FMBN8B` | 各HTMLの `<head>` と `<body>` 直後 |
| GA4 | `G-9552DY4P6M`（プロパティ `549587971`） | GTM 経由 |
| Microsoft Clarity | `yfrx0ppifz` | GTM 経由 |
| Google 広告 | `AW-18023727991` | 各HTMLに直接 gtag を設置 |

**タグはHTMLに直接書かれています。** ページを編集するときに `<head>` を消さないよう注意してください。
GA4 と Clarity は GTM のコンテナ側に入っているので、HTML には GTM のスニペットしかありません。

管理は Google アカウント `umezawa@gtsleep.co.jp` です。

## 予約フォーム

店舗ページ（`/fudomae`・`/shirokanetakanawa`）の予約ウィジェットは、Google Apps Script の Web アプリに JSONP で送信しています。

```
https://script.google.com/macros/s/AKfycbwLcuFJ0nwYnKDqev21qUrXn2PXGwPfHSEAxMYqA-YHfblRYmDtQ4s2GkMePAJIZ8Zz4g/exec
```

送信成功時に GA4 の `reservation_complete` が発火します。フォームに触れた時点で `form_start` が発火します。

## 制約・注意点

### GitHub Pages ではリダイレクトが打てません

サーバー側の設定を書く場所がないため、301リダイレクトができません。`_redirects` ファイルは Netlify 専用で、GitHub Pages では無視されます。

**URL構造を変えるときは要注意です。** 旧URLが404になり、検索順位の引き継ぎができません。実際に2026年8月のURL変更でこれが起きています（CHANGELOG 参照）。

どうしても必要な場合は、旧パスの位置に `<meta http-equiv="refresh">` を書いたHTMLを置く方法がありますが、301ほどの効果はありません。

### Netlify の「Pretty URLs」はもうありません

Netlify は配信時にHTMLを書き換えて `.html` を自動で除去していました。GitHub Pages はソースをそのまま返します。
**書いたHTMLがそのまま配信されます。**

### `shirokanetakanawa.html` の素性

このファイルだけ、3箇所の内部リンクがシングルクォート（`href='/fudomae'`）で、属性の並び順も他ファイルと違います。ブラウザが描画したHTMLを保存し直したものと思われます。

動作に問題はありませんが、**他の2ファイルと書式が揃っていない**ことを承知のうえで編集してください。

### リポジトリは public です

Netlify の無料プランが private リポジトリで複数人のコミットを許可しないため、2026-10-14 に public に変更しました（詳細は CHANGELOG）。GitHub Pages へ移行した現在は private に戻すことも可能です（GitHub Pro 契約済みのため）。

**API キーや認証情報をコミットしないでください。** 現時点で機密情報は含まれていないことを確認済みです。
