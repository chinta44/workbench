# WORKBENCH — 開発ポータル

静的なカタログ型の開発ポータルです。作成した個人開発 Web アプリやツールを一覧表示し、ブラウザ経由で追加・編集・削除できます。apps.json にアプリ情報を保持し、画像は images/ フォルダへアップロードします。

作成者: GitHub Copilot Chat Assistant

---

## 主な特徴

- シンプルな静的 HTML（index.html）で動作
- apps.json を読み書きしてアプリ一覧を管理
- ブラウザ上でアプリの追加・編集・削除が可能（GitHub API 経由でリポジトリへ直接コミット）
- スクリーンショット画像を images/ にアップロード（自動コミット）
- タグでフィルタリング、更新日時順の並び替え


## リポジトリ構成（主要ファイル）

- `index.html` — アプリ本体（UI + GitHub API ロジック）
- `apps.json` — アプリ一覧データ（JSON 配列）
- `images/` — 各アプリのスクリーンショット格納ディレクトリ
- `favicon.ico` / `favicon.png` — アイコン


## 動作要件

- ブラウザ（`index.html` を開ける環境）
- GitHub リポジトリに対する書き込み権限を持つ Personal Access Token（ブラウザで操作するためトークンが必要）
  - `Fine-grained token` を推奨
  - 必要な権限: 対象リポジトリの `Contents: Read and write`


## ローカルでの確認（簡単）

`index.html` は静的ファイルなので、直接ブラウザで開くか、軽量なローカルサーバーで確認できます。

- 例: Python（カレントディレクトリをリポジトリルートにして実行）
  - Python 3:
    - `python -m http.server 8000`
    - ブラウザで `http://localhost:8000` を開く
- 例: Node (serve)
  - `npx serve .`


## 使い方（ユーザー向け）

1. `index.html` を開き、右上の「⚙（設定）」を押す
2. Settings モーダルで以下を入力して保存:
   - GitHub ユーザー名 / Organization 名（owner）
   - リポジトリ名（repo）
   - ブランチ名（通常は `main`）
   - Personal Access Token
     - トークンは `Fine-grained tokens` で発行し、対象リポジトリに対して `Contents: Read and write` を付与することを推奨します（`index.html` 内の注意書き参照）
3. アプリを追加するには右下の「＋」を押してフォームに入力し、画像を選択すると `apps.json` と `images/` にコミットされます
4. 各カードの「✎」で編集、「🗑」で削除（削除はリポジトリ上から `apps.json` を更新し、可能なら画像も削除します）

注意:
- ブラウザから GitHub API を直接叩くため、入力したトークンはそのブラウザの `localStorage` に保存されます（`index.html` 内の設定画面記載）
- 操作後、反映に数十秒かかる場合があります


## apps.json のフォーマット

`apps.json` は JSON 配列で、各要素がアプリ情報です。主なプロパティ:

- `name`: string — アプリ名（必須）
- `url`: string — アプリの URL（必須）
- `description`: string — 説明
- `platform`: string — プラットフォーム (例: `WEB`, `ANDROID`, `OTHER`)
- `tags`: array — タグの配列 (例: `["天気", "データ可視化"]`)
- `image`: string — `images/` 以下のファイルパス（存在しない場合は空文字列）
- `updatedAt`: ISO 8601 形式の日時文字列（自動セット）

例（抜粋）:

```json
[
  {
    "name": "天気くらべ帳",
    "url": "https://chinta44.github.io/weather-almanac/",
    "description": "気温比較アプリ",
    "platform": "WEB",
    "tags": ["天気"],
    "image": "images/tenki-20260730.png",
    "updatedAt": "2026-07-28T08:13:26.176Z"
  }
]
```


## GitHub API によるファイル操作の仕組み（要点）

- apps.json の読み取り: `GET /repos/{owner}/{repo}/contents/apps.json?ref={branch}`
- apps.json の更新: `PUT /repos/{owner}/{repo}/contents/apps.json`
  - PUT 時は content を Base64 (UTF-8) にエンコードし、必要に応じて既存の `sha` を指定
- 画像アップロード: `PUT /repos/{owner}/{repo}/contents/images/{filename}`
- 画像削除: `DELETE /repos/{owner}/{repo}/contents/{path}`

`index.html` に、Base64 エンコード/デコードや SHA の扱い、エラーハンドリング、コミットメッセージ生成などのロジックが実装されています。


## 開発・カスタマイズ

- バージョン番号は `index.html` 内の `WORKBENCH_VERSION` 定数で管理されています（例: `v2.2`）
- UI やスタイルは `index.html` 内の CSS を直接編集してカスタマイズ可能
- `apps.json` を手動で編集する場合は JSON の整合性に注意してください（コミット前に念のため JSON を検証することを推奨）


## デプロイ

- 静的サイトなので GitHub Pages にそのままホストできます（リポジトリ設定から Pages を有効にし、root または `/docs` を指定）
- Netlify / Vercel 等の静的ホスティングサービスも利用可能


## トラブルシューティング

- 「取得失敗 (404)」: `apps.json` がターゲットブランチに存在するか確認してください
- 認証エラー (401/403): トークンの権限・有効期限・入力ミスを確認してください。Fine-grained token を使う場合は対象リポジトリに `Contents: Read and write` を割り当てる必要があります
- CORS / ブラウザ問題: 通常 GitHub API はブラウザからのアクセスを許可しますが、ブラウザ拡張や企業ネットワークの設定でアクセスが制限される場合があります
- 反映遅延: GitHub の API 応答後、GitHub Pages 等で公開している場合はページの反映に時間がかかる場合があります


## 貢献

- バグ報告や改善提案は Issue でお願いします
- 変更を加える場合は fork → ブランチ → Pull Request の流れでお願いします
- 大きな設計変更を検討する場合は事前に Issue で相談してください


## ライセンス

このリポジトリ内に明示的な LICENSE ファイルは見つかりませんでした。使用・再配布を行う場合は注意してください。ライセンスを付与したい場合は `LICENSE` ファイル（例: MIT）を追加することをおすすめします。


---

README をリポジトリに追加しました。必要なら内容を更新して再コミットします。