# KUROZUMI.ver0.1
Google Apps Script × Google スプレッドシートで構築する業務管理 MPA（Multi Page Application）の実装です。AppConfig の初期化順序問題を解消し、初期アクセス時にスキーマ整備と防御的な UI フローを実現しています。
## 主な特徴
- **AppConfig の早期初期化**：`00_config.gs` を最優先でロードし、`main.gs` から参照される前に設定を確実に利用可能にします。
- **スキーマ自己修復**：`SpreadsheetRepository.ensureSchema()` が Stores / Staff / Customers / Sales / Payments / Production / Meta_Lookups / Logs シートを自動生成し、不要なシートを削除します。
- **マルチページ UI**：`index.html` が `?page=` クエリを解釈し、ホワイトアウト時には `safeError.html` で保護表示します。
- **操作ログ**：`AppLogger` が Stackdriver と Logs シートに二重出力し、監査トレイルを保持します。
- **ChatGPT サポート**：OpenAI API キー未設定時は丁寧な日本語メッセージで案内し、設定済みなら `askAssistant` が回答を返します。
## ディレクトリ構成
```
apps-script/
  00_config.gs            # AppConfig。Script Properties から設定を読み込み
  01_utils.gs             # LockService ラッパー、サニタイズ、include ヘルパー
  02_repository.gs        # スプレッドシート CRUD、スキーマ保証と不要シート削除
  03_logger.gs            # Logs シート / Stackdriver へ出力
  04_services.gs          # エンティティ作成・一覧・CSV エクスポート
  05_gpt.gs               # ChatGPT 連携（OpenAI API）
  06_api.gs               # フロントエンド公開関数
  10_main.gs              # doGet エントリーポイント
  appsscript.json         # マニフェスト
  index.html              # メイン MPA テンプレート
  safeError.html          # フェイルセーフ表示
  style.html              # AKIRA 系モノトーン × ネオンスタイル
  scripts.html            # 画面制御 JS（フォーム・テーブル・アシスタント）
  components/
    formBuilder.html      # 汎用フォームテンプレート
    tableBuilder.html     # テーブルテンプレート
    managementPage.html   # 管理画面コンテナ
    ui.html               # 概要セクション
    gpt-assistant.html    # 右下 AI サポート
```
## 初期セットアップ
1. Apps Script のスクリプトプロパティに以下を設定します。
   - `SPREADSHEET_ID`: 参照するスプレッドシート ID
   - `ENV`: `DEV` または `PROD`
   - `OPENAI_API_KEY`: （任意）ChatGPT 連携を有効化する場合
2. ウェブアプリとしてデプロイし、「アクセス権：誰でも」「実行権限：自分」を選択します。
3. 初回アクセス時にスキーマ整備と不要シート削除が実行されます。
## 運用 Tips
- 追加で必要なシートがある場合は `SpreadsheetRepository.schema` に列定義を追記してください。
- スクリプトのタイムアウトを避けるため、極端に大きなデータセットではフィルタリングやページネーションの導入を検討してください。
- OpenAI API 呼び出しに失敗した場合は Logs シートにエントリが残ります。
