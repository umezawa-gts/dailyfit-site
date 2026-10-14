# サイト更新の手順（運用ルール）

> HTMLを更新するときに読む資料です。
> 構成の詳細は [INFRASTRUCTURE.md](./INFRASTRUCTURE.md)、変更の経緯は [CHANGELOG.md](./CHANGELOG.md) を見てください。
> 最終更新: 2026-10-15

---

## 最初に: 配信の仕組みが変わりました

**2026-10-15 に、サイトの配信を Netlify から GitHub Pages に移行しました。**

```
以前:  GitHub → Netlify → dailyfit.jp
現在:  GitHub → GitHub Pages → dailyfit.jp
```

変わったのは**配信するサーバーだけ**です。リポジトリも、HTMLの中身も、更新のやり方も、基本は同じです。

**Netlify の管理画面を見る必要はもうありません。** Netlify のプロジェクトは切り戻し用に残してありますが、本番には一切関与していません。

移行した理由は3つです。

1. Netlify 無料プランが private リポジトリで**コミット作者を1人しか許可しない**制限があり、2026年9月から1か月間デプロイが止まっていた
2. Netlify 無料プランは本番デプロイが**月20回程度で上限**
3. Netlify のプロジェクトが個人アカウント配下にあり、他の人が触れなかった

---

## 更新の手順

### 1. 必ず最新を取得してから作業を始める

```bash
git checkout main
git pull
```

**これを飛ばさないでください。** いま手元にあるファイルが古いと、他の人が入れた変更を上書きして消してしまいます。

2026年9月〜10月にかけて、以下が main に入っています。手元のクローンが9月以前のものなら、必ず pull してください。

| コミット | 内容 |
|---|---|
| `b6dc93a` | title / description の見直し、canonical と OGP の追加 |
| `5864de7` | `.nojekyll` 追加 |
| `228ea3c` | `CNAME` 追加 |
| `940de9b` | 内部リンクを拡張子なしURLに統一 |

### 2. 編集する

編集するのはリポジトリ直下の `.html` と `assets/` です。

### 3. コミットして push する

```bash
git add .
git commit -m "変更内容がわかるメッセージ"
git push origin main
```

**push すれば自動で本番に反映されます。** ビルドの実行や公開ボタンの操作は不要です。通常1〜2分で反映されます。

### 4. 反映を確認する

- 進行状況: リポジトリの **Actions** タブ（`pages build and deployment` が緑になれば完了）
- 実際のページ: https://dailyfit.jp を**スーパーリロード**（Cmd+Shift+R / Ctrl+Shift+R）で開く

キャッシュが残って古い表示になることがあるので、普通のリロードではなくスーパーリロードで見てください。

---

## やってはいけないこと

### リダイレクトは打てません

**GitHub Pages にはリダイレクト機能がありません。** `_redirects` ファイルはリポジトリに残っていますが、**Netlify 専用のファイルで、現在は何も効いていません。**

そのため **URL構造（ファイル名）を変えないでください。** 変えると旧URLが404になり、検索順位を引き継げません。

実際に2026年8月のURL変更でこれが起きています。3か月で152クリック分の流入が404に落ちました（詳細は CHANGELOG）。

どうしてもファイル名を変える必要がある場合は、事前に相談してください。

### 消してはいけないファイル

| ファイル | 消すとどうなるか |
|---|---|
| `CNAME` | カスタムドメインが外れ、**dailyfit.jp でアクセスできなくなります** |
| `.nojekyll` | GitHub Pages のビルド挙動が変わり、一部ファイルが配信されなくなる可能性があります |

### `<head>` の計測タグを消さないこと

各HTMLの `<head>` には Google タグマネージャーと Google 広告のタグが入っています。ここを消すと**広告の効果計測とアクセス解析が止まります。**

```
GTM-52FMBN8B      Google タグマネージャー（GA4 と Clarity はこの中）
AW-18023727991    Google 広告
```

あわせて `<body>` 直後の `<noscript>` の iframe も GTM の一部です。

### 内部リンクは拡張子なしで書く

```html
<!-- ○ -->
<a href="/fudomae">
<a href="/shirokanetakanawa">
<a href="/privacy">
<a href="/">

<!-- × -->
<a href="fudomae.html">
```

以前は Netlify が配信時に `.html` を自動で除去していましたが、**GitHub Pages は書いたHTMLをそのまま返します。**

`.html` 付きでもページは開きますが、各ページの `<link rel="canonical">` が拡張子なしのURLを指しているため、食い違うとGA4とClarityの集計が2つに割れます。

---

## 困ったとき

### push したのに反映されない

1. リポジトリの **Actions** タブを開く
2. `pages build and deployment` が失敗（赤）していないか確認
3. 失敗していればログのエラーを確認

### 表示が古いまま

スーパーリロード（Cmd+Shift+R）を試してください。GitHub Pages は CDN でキャッシュされるため、数分かかることがあります。

### サイトが表示されない

まず **Settings ▸ Pages** を開いて、

- `Your site is live at https://dailyfit.jp/` と出ているか
- Custom domain が `dailyfit.jp` のままか
- Enforce HTTPS が有効か

を確認してください。

それでも解決しない場合は、DNS（お名前.com）側の可能性があります。INFRASTRUCTURE.md の「DNS」の項を見てください。**MXレコードだけは絶対に触らないでください**（メールが止まります）。
