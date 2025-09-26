# KUROZUMI.ver0.1
https://docs.google.com/spreadsheets/d/1pJq2KT4WSfRIFdjGhZxm2m8aYfeq1g2KbUVLxPEL8TI/edit?gid=1804339974#gid=1804339974の内容をMPAでのアプリ化
# KUROZUMI ver.0.1 — README

> **GAS × Google スプレッドシートの業務管理 MPA**
> 目標：白画面（ホワイトアウト）や画面遷移不具合を根絶し、DB（指定スプレッドシート）と堅牢に連携。
> UIは**AKIRA系モノトーン・メタリックのサイバーパンク**、日本語主体・かんたんな英語表記併記、ChatGPTサポート内蔵。

---

## 1. ゴール / できること（Features）

### ✅ v0.1 ハイライト

* Apps Script 側で `SpreadsheetRepository.ensureSchema()` を呼び出し、初期アクセス時に指定シート（Stores / Staff / Customers / Sales / Payments / Production / Meta_Lookups / Logs）を自動生成します。
* `index.html` は MPA 方式で `?page=sales` などのクエリを解釈し、`ALLOWED_PAGES` ホワイトリスト外は `start` にフォールバック。ホワイトアウトを防ぐため `safeError.html` でフェイルセーフ描画します。
* 画面内フォームは `data-handler` 属性で Apps Script の同名関数にルーティングされ、`components/managementPage.html` が送信/失敗ハンドリングを一元化。
* Google スプレッドシートを唯一の DB とし、`Utils.withLock`（DocumentLock）で多重送信を抑止。操作ログは `logger.gs` が `Logs` シートに追記します。
* ChatGPT サポートは `gpt.gs` の `askAssistant` から OpenAI API を呼出し、API キー未設定時はユーザーに日本語で案内。
* UI はモノトーン×ネオンサイバーパンク調テーマ（`style.html`）を適用。浮遊するサポートウィンドウとメニュー遷移はホワイトアウトしない設計です。

---

* **マルチページ型（MPA）**：`start → menu → 各管理ページ（売上/顧客/店舗/生産/支払い/スタッフ/…）`
* **フォームビルド**：メタ定義からフォーム生成（必須/型/選択肢/検証）
* **テーブルビルド**：列定義から一覧生成（検索/ソート/ページング/CSV出力）
* **DB**：指定スプレッドシートを**唯一のデータソース**としてCRUD（create/read/update/delete）
* **ChatGPTアシスト**：ページ右下のヘルプから操作説明/項目の意味/入力補完を提示
* **防御と保守**：

  * 書き込み時の**LockService**（重複送信/多重更新対策）
  * **入力検証・サニタイズ**（XSS/不正文字）
  * **エラー境界**：ユーザー画面は決して真っ白にしない（再試行/デグレードUI）
  * **監査ログ**（操作/例外/API呼出）
  * **環境切替**（DEV/PROD）
* **UI**：モノトーン基調＋メタリック、動作軽量、ダーク/ハイコントラスト、JP/EN最小語併記

---

## 2. リポジトリ構成（Project Structure）

```
/components/
  formBuilder.html          # 動的フォームUI（検証/バリデーション/送信）
  gpt-assistant.html        # 画面右下のChatGPTヘルプUI
  managementPage.html       # 管理ページ用共通レイアウト（カード/タブ/通知）
  tableBuilder.html         # 動的テーブルUI（検索/ソート/ページング）
  ui.html                   # 💡（注意）ui.hhtml → ui.html に統一推奨
