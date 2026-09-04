# SETARS-SUPPORT-PAGE

setars（個人開発者）が公開している、自作アプリの一覧（ショーケース）と無料の
お問い合わせ窓口を兼ねた1ページのポータルサイト。GitHub Pages等で公開する
想定の**単一の静的HTMLファイル**（`index.html`）で構成されている。

- リポジトリ: https://github.com/D-ogigi/SETARS-SUPPORT-PAGE
- 開発者は **個人（法人ではない）**。文言は必ず一人称単数（「私」「I」）を使う。
  「私たち」「We/Our/Us」は使わないこと（過去に修正済み）。

## 技術構成

- **ビルド不要の単一HTMLファイル**（`index.html`）。ブラウザで直接開けば動作する。
- CSS: Tailwind CSS（CDN経由 `cdn.tailwindcss.com`）＋ `<style>` 内の少量のカスタムCSS。
- JS: フレームワーク無し（vanilla JS）。`<script>` タグ2つ（Tailwind設定用、本体ロジック用）。
- 依存パッケージ・npm・テストランナーは無し。

### 変更後の検証方法（ビルドが無いため）

1. 本体の `<script>` ブロックを抜き出して `node --check` で構文チェック
   （過去のやり取りで毎回この方法を使っている）。
2. HTMLタグの開閉バランスを `python3` の `html.parser.HTMLParser` で簡易チェック
   （`<meta>` `<input>` の自己終了タグに対する誤検知は無視してよい）。
3. 可能なら実際にブラウザで開いて言語切替・フォーム送信・アコーディオンの
   目視確認を行う（このセッションではブラウザでの目視確認はまだ一度もできていない）。

## デザイン方針（重要・必ず踏襲すること）

「AI感」のある見た目（絵文字、過剰なアイコン、紫の多用、不要なグラデーション）を
明示的に禁止し、Apple の Human Interface Guidelines を参考にした配色に統一済み。

- 背景はほぼフラットな `#fbfbfd`（Apple.comに近いニュートラルトーン）。
- アクセントカラーは2色のみ（Tailwind設定の `theme.extend.colors` に定義）：
  - `accent: #0071e3`（Apple Blue。リンク・フォーカスリング・ボタン等）
  - `ink: #1d1d1f`（ほぼ黒。App Storeボタン・言語トグルのアクティブ状態等）
- **紫・ピンク・インディゴ系のグラデーションは使わない。** 新しい装飾を追加する
  ときも、この2色 + Tailwindのグレースケールの範囲に収める。
- アイコンのフォールバック配色は `ICON_COLORS`（青・ティール・オレンジのフラット3色、
  紫は含まない）。
- ブランド名は **setars**。バックロニムは
  **S.E.T.A.R.S. = Simple Effective Tools for Automation, Remote-control & Support**
  （ヘッダーに小さく表示、ホバーでツールチップ）。
  タグラインは「さりげなく、役に立つ。」/ "Quietly useful, every day."

## データ駆動の設計（最重要）

**新しいアプリを追加・変更する際は、`index.html` 内の `myApps` 配列を編集するだけで
よい。** HTMLを手で追記する必要はない。配列を1件追加・編集するだけで、以下が
自動的に同期更新される：

1. ショーケースのアプリカード（アイコン・名称・紹介文・ダウンロード/Coming Soon表示）
2. お問い合わせフォームの「対象アプリ」プルダウンの選択肢
3. 「レビューを書く」リンク（`storeUrl` が `apps.apple.com/.../id数字` 形式の
   実際のApp StoreのURLである場合のみ、正規表現で自動生成される。未公開アプリには
   表示されない）

`myApps` の各項目のフィールド：

| フィールド | 説明 |
|---|---|
| `id` | 内部識別子。フォームの値やプライバシーリンクの紐付けに使用 |
| `nameJa` / `nameEn` | 表示名（多くの場合同じ） |
| `taglineJa` / `taglineEn` | カードの紹介文 |
| `icon` | アイコン画像パス。読み込み失敗時は単色＋頭文字に自動フォールバック |
| `storeUrl` | App StoreのURL。`apps.apple.com/.../id数字` 形式だと自動で「レビューを書く」リンクも生成される |
| `privacyId` | ページ下部のプライバシーポリシー `<details id="...">` とのひも付け |
| `isReleased` | `false` なら「Coming Soon」バッジ表示＋ダウンロードボタン非表示 |

プライバシーポリシーは `translations.ja` / `translations.en` 内の
`privacy.<id>Title` / `privacy.<id>Body` キーと、HTML側の
`<details id="privacy-<id>">` をセットで用意する（アプリ固有の内容が必要な場合）。
特に用意していない場合は共通ポリシー `privacy-common` を使い回してよい
（写真・カメラ・マイク等、センシティブな権限を使うアプリは専用ポリシーを
用意した方がよい。Pitatrimはこの理由で専用セクションを作った）。

## i18n（日英バイリンガル）の仕組み

- `navigator.language` を見て初期表示言語を自動判定（`ja`で始まれば日本語、
  それ以外は英語）。
- ページ上部の言語トグルでリロード無しに切り替え可能。
- 静的なUI文言（見出し・ラベル・ボタン等）は `translations.ja` / `translations.en`
  オブジェクトで管理し、`data-i18n` / `data-i18n-html` / `data-i18n-placeholder`
  属性を持つ要素に対して `applyLanguage()` が一括反映する。
- アプリ固有の文言（名前・紹介文）は `myApps` 側の `nameJa/nameEn` 等で管理する
  （`translations` には入れない）。

## 現在のアプリのステータス（このファイルを読んだ時点の最新状態と必ず違う可能性があるので、
実際に `index.html` の `myApps` 配列を見て確認すること）

- **Spotrans Lens**: 諸事情により配信停止中。`myApps` 内でコメントアウト済み
  （ショーケース・プルダウンから非表示）。プライバシーポリシーの `<details>` も
  完全にコメントアウト済み。文言データ（`translations` 内の `privacy.spotrans*`）
  は将来の再開に備えて残してある。**再開する場合は `myApps` とプライバシー
  セクションの両方のコメントを外す。**
- **AirTouchPad**: リリース済み。`storeUrl` は暫定でGitHub Pagesのリンク
  （`https://d-ogigi.github.io/airtouchpad/`）になっている。App Store公開後は
  実URLに差し替えが必要。
- **Pitatrim**: 写真・書類をトリミングしてPDF化するアプリ。App Store未公開
  （`isReleased: false`, `storeUrl: "#"` のプレースホルダー）。画像処理は
  端末内で完結し、外部サーバーには送信しない／広告・課金・解析SDKは
  未使用という前提で専用プライバシーポリシーを作成済み。公開後は
  `storeUrl` を実URLに差し替え、`isReleased: true` にする。
- **次の名作アプリ（coming-soon）**: 名前未定の汎用プレースホルダー枠。

## まだプレースホルダーのままの箇所（要対応）

- **Buy Me a Coffee**: `https://www.buymeacoffee.com/setars` は仮のユーザー名。
  ヘッダーとフッターの2箇所にリンクがある。アカウント作成後に実ユーザー名へ
  差し替えが必要。
- **AirTouchPad / Pitatrim の `storeUrl`**: 上記の通り、正式なApp Store URL
  確定後に差し替えが必要。

## 既に設定済みのもの（プレースホルダーではない）

- お問い合わせフォームの送信先（FormSubmit）: `setars.support@gmail.com`
- X (Twitter): `https://x.com/SetarsSupport`（フッターにアイコンリンクあり）

## Gitのコミット規約

このリポジトリの過去のコミットメッセージは日時のみ（例: `20270728_2156`）
だったが、途中から**内容が分かる説明的なメッセージ**に変更した
（変更量が多く、日時のみでは後から追いにくいため）。特別な指示が無い限り、
今後もこの「内容が伝わる1〜2文の説明的コミットメッセージ」の方針を踏襲する。

`git push` はユーザーの明示的な許可を得てから行うこと（このプロジェクトでは
毎回「コミットしてプッシュしてください」と依頼されてから実行している）。

## セキュリティ・プライバシー上の注意

- 個人情報（実名・住所・電話番号・実メールアドレス等）や社外秘情報を
  ソースやコミットメッセージにそのまま書く前に、ユーザーに一度確認する。
- プライバシーポリシー本文中の連絡先がプレースホルダー
  （例: `support@example.com`）のままの箇所が一部残っている
  （AirTouchPadのポリシー内）。公開前に実際の連絡先へ差し替えが必要。
