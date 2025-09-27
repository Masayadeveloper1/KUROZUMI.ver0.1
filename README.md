KUROZUMI.ver0.1
Google Apps Script で構築する KUROZUMI マルチページアプリケーションのサンプル構成です。自己修復型のネームスペースを導入し、アプリの初期化順序に依存せず安全に起動できます。

## 実装ハイライト
- **自己修復型設定ロード**：`config/AppConfig` がデフォルト構成を提供し、スクリプトプロパティに保存された JSON をマージ・検証します。
- **フォールバック可能なセキュリティレイヤ**：`security/SecurityUtils` が CSRF トークン生成とページサニタイズを提供し、プロパティが未設定でも安全なトークンを生成します。
- **堅牢なモジュールローダ**：`bootstrap/namespace.gs` が `KUROZUMI.define` / `KUROZUMI.require` を提供し、循環依存を検出・防止します。
- **堅牢な doGet エントリーポイント**：`main/Application` がテンプレート読込・例外処理・CSRF トークン発行を担当し、テンプレート読込失敗時にはエラーテンプレートへフェイルオーバーします。
- **衝突しないスプレッドシート管理**：`sheets/Registry` がシート構造を idempotent に整備し、重複定義や余剰シートを自動的に排除します。

## ディレクトリ構成
```
src/
  bootstrap/namespace.gs           # KUROZUMI ネームスペースとモジュールローダ
  config/app_config.gs             # アプリ設定の生成とバリデーション
  security/security_utils.gs       # セキュリティ関連ユーティリティ
  core/self_heal.gs                # 設定・セキュリティの自己診断
  ui/template_service.gs           # テンプレートレンダリングユーティリティ
  sheets/registry.gs               # スプレッドシート構造の同期
  main.gs                          # doGet エントリーポイント
```

## 移植手順
1. Apps Script プロジェクトに `src` 配下のファイルをすべて追加します。
2. 必要に応じて `config/app_config.gs` の `allowedPages` や `templateRoot` を運用環境に合わせて変更します。
3. HTML テンプレート群（`templates/index.html` など）を Apps Script プロジェクト内に配置します。
4. ウェブアプリとしてデプロイし、`?page=` クエリでページを切り替えられることを確認します。

## カスタマイズのヒント
- 設定を Script Properties や外部ストレージに置く場合は、`config/AppConfig` の `parseScriptProperties` を拡張してください。
- `security/SecurityUtils` の `createCsrfToken` では charset をカスタマイズできます。`AppConfig.security.tokenCharset` で指定します。
- 追加のユーティリティを導入する際は、`core/self_heal.gs` と同様にフォールバックを用意すると安全です。
