# KUROZUMI.ver0.1
Google Apps Script で構築する KUROZUMI マルチページアプリケーションのサンプル構成です。アプリの初期化順序に依存せず安全に起動できるよう、構成オブジェクトとセキュリティユーティリティを自己修復する仕組みを実装しています。

## 実装ハイライト
- **自己修復型設定ロード**：`AppConfig.gs` がデフォルト構成を提供し、既存設定が欠落・破損していても安全にマージします。
- **フォールバック可能なセキュリティレイヤ**：`SecurityUtils.gs` が CSRF トークン生成とページサニタイズを提供し、未ロード時も `main.gs` がフォールバック実装を注入します。
- **堅牢な doGet エントリーポイント**：`main.gs` がテンプレート読込・例外処理・CSRF トークン発行を担当し、テンプレート読込失敗時にはエラーテンプレートへフェイルオーバーします。
- **衝突しないスプレッドシート管理**：`SpreadsheetRepository.gs` がシート構造を idempotent に整備し、重複定義や余剰シートを自動的に排除します。

## ディレクトリ構成
```
apps-script/
  AppConfig.gs             # アプリ設定の生成とバリデーション
  SecurityUtils.gs         # セキュリティ関連ユーティリティ
  SpreadsheetRepository.gs # スキーマ初期化とシート管理
  main.gs                  # doGet エントリーポイント
```
## 移植手順
1. Apps Script プロジェクトに上記 3 ファイルを追加します。
2. 必要に応じて `AppConfig.gs` の `allowedPages` や `templateRoot` を運用環境に合わせて変更します。
3. HTML テンプレート群（`templates/index.html` など）を Apps Script プロジェクト内に配置します。
4. ウェブアプリとしてデプロイし、`?page=` クエリでページを切り替えられることを確認します。

## カスタマイズのヒント
- 設定を Script Properties や外部ストレージに置く場合は、`AppConfig.gs` の `mergeConfiguration` 部分にロード処理を追加してください。
- `SecurityUtils.gs` の `createCsrfToken` では charset をカスタマイズできます。`AppConfig.security.tokenCharset` で指定します。
- 追加のユーティリティを導入する際は、`main.gs` の `ensureSecurityUtils` と同様にフォールバックを用意すると安全です。
