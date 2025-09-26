# KUROZUMI.ver0.1
https://docs.google.com/spreadsheets/d/1pJq2KT4WSfRIFdjGhZxm2m8aYfeq1g2KbUVLxPEL8TI/edit?gid=1804339974#gid=1804339974の内容をMPAでのアプリ化
# KUROZUMI ver.0.1 — README

> **GAS × Google スプレッドシートの業務管理 MPA**
> 目標：白画面（ホワイトアウト）や画面遷移不具合を根絶し、DB（指定スプレッドシート）と堅牢に連携。
> UIは**AKIRA系モノトーン・メタリックのサイバーパンク**、日本語主体・かんたんな英語表記併記、ChatGPTサポート内蔵。

---

## 1. ゴール / できること（Features）

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
config.gs                   # 設定（環境/シートID/モデル名/権限）※ config.gsl → gs に統一推奨
customer.html               # 顧客管理
formHandler.gs              # フォーム受信・検証・DB書込（doPost含む）
gpt-helper.html             # GPTガイド/ショートカット
gpt.gs                      # OpenAI API呼び出し（UrlFetchApp）
index.html                  # ルートHTML（<main>内に部分差し替え）
logger.gs                   # ロギング（操作/例外/計測）
main.gs                     # ルーター doGet(e) / テンプレ評価 / CSRFトークン
menu.html                   # メニュー/ランディング
payment.html                # 支払い管理
production.html             # 生産管理
sales.html                  # 売上管理
spreadsheet.gs              # シートCRUD/バルク操作/スキーマ検査
staff.html                  # スタッフ管理
start.html                  # 最初のフォーム（オンボーディング）
store.html                  # 店舗管理
style.html                  # テーマ/CSS（AKIRAモノトーン・メタリック）
utils.gs                    # 共通関数（バリデ/型変換/日付/ID/権限）
```

> **重要な命名ケア**
>
> * `components/ui.hhtml` は **`ui.html` へリネーム**。
> * `config.gsl` は **`config.gs` へリネーム**。
>   これらの**誤名ファイル**はテンプレート読込に失敗し、**遷移後ホワイトアウト**の主因になり得ます。

---

## 3. 依存 / 事前準備

1. **スプレッドシート**（DB）：

   * 指定のスプレッドシート（提示URL）の**スプレッドシートID**を取得
     例: `https://docs.google.com/spreadsheets/d/<ここがID>/edit...`
2. **Apps Script プロパティ設定**（ファイル > プロジェクトのプロパティ > スクリプトのプロパティ）

   * `APP_ENV`: `DEV` or `PROD`
   * `SPREADSHEET_ID`: `<上記ID>`
   * `OPENAI_API_KEY`: `<OpenAIのAPIキー>`
   * （任意）`OPENAI_MODEL`: `gpt-4o-mini` など
3. **権限**：初回実行で各スコープ承認
4. **デプロイ**：デプロイ > ウェブアプリとして導入 > `誰でも（匿名）` or `自分のみ`（要件に応じて）

---

## 4. データモデル（推奨スキーマ例）

> シートは**1行目ヘッダ**固定。Apps Script 側は**ヘッダ名をキー**にアクセスし、列順変更に強くします。
> 既存シートに合わせて `spreadsheet.gs` のマッピングで**差分を吸収**します。

* **Stores**（店舗）
  `store_id, store_name, type, prefecture, address, phone, status, opened_on, closed_on, updated_at, updated_by`
* **Staff**（スタッフ）
  `staff_id, name, role, email, phone, store_id, employment_type, status, hired_on, left_on, updated_at, updated_by`
* **Customers**（顧客）
  `customer_id, name, kana, email, phone, address, note, updated_at, updated_by`
* **Sales**（売上）
  `sale_id, sale_date, store_id, staff_id, customer_id, item, qty, unit_price, tax_rate, payment_method, total, note, updated_at, updated_by`
* **Payments**（入出金）
  `payment_id, sale_id, method, amount, status, paid_at, updated_at, updated_by`
* **Production**（生産）
  `job_id, product, qty, status, start_at, end_at, store_id, note, updated_at, updated_by`
* **Meta_Lookups**（選択肢）
  `type, code, label_ja, label_en, sort`

> 初回起動時に `spreadsheet.gs` の **`ensureSchema()`** でヘッダ検証し、足りない列を通知（オプションで自動生成）。

---

## 5. 画面とシートのひもづけ

| ページ    | HTML                                        | 主シート                        | 主機能                      |
| ------ | ------------------------------------------- | --------------------------- | ------------------------ |
| Start  | `start.html`                                | `Stores`/`Staff`            | 初期設定・最低限レコード投入（例：店舗/管理者） |
| Menu   | `menu.html`                                 | -                           | 各管理ページへ遷移                |
| 売上管理   | `sales.html` + `formBuilder`/`tableBuilder` | `Sales`/`Customers`/`Staff` | 売上登録/一覧/集計               |
| 顧客管理   | `customer.html`                             | `Customers`                 | 顧客登録/重複チェック              |
| 店舗管理   | `store.html`                                | `Stores`                    | 店舗登録/状態変更                |
| 生産管理   | `production.html`                           | `Production`                | 工程・進捗                    |
| 支払い管理  | `payment.html`                              | `Payments`                  | 入出金処理・売上紐づけ              |
| スタッフ管理 | `staff.html`                                | `Staff`                     | 人員/権限                    |

---

## 6. ルーティングとホワイトアウト根絶

### 6.1 ルータ（`main.gs`）

* **ホワイトリスト**：`ALLOWED_PAGES = ['start','menu','sales','customer','store','production','payment','staff']`
* **`doGet(e)`**：

  1. `page = (e.parameter.page || 'start')` を **trim & sanitize**
  2. 存在しない場合は **`not-found`** を返す
  3. テンプレ評価時は**例外を握りつぶさず**、**`safeError.html`** にフォールバック
  4. **CSRFトークン**（HtmlServiceテンプレ変数に埋め込み）

### 6.2 クライアント遷移

* `a[data-nav]` クリック → ローディング（スピナー） → `google.script.history.push`
* テンプレ命名ミス/`<?!= include('...') ?>` の **ファイル名不一致**が最大の地雷。
  **`ui.hhtml` → `ui.html`** に即リネームしてください。
* 失敗時は**必ず**`error-overlay`（再試行/トップへ戻る）を表示（**真っ白禁止**）。

---

## 7. フォーム送信が進まない時の対策（`formHandler.gs`）

* 送信側：`google.script.run.withSuccessHandler(...).withFailureHandler(...).submitForm(formData)`

  * `event.preventDefault()` を忘れない
  * 成功/失敗でボタン再有効化・通知
* 受信側：

  * **LockService** で二重送信防止
  * **型/必須/範囲**チェック → 400系メッセージ（ユーザー文）を返す
  * 例外は握らず**logger.gs**に記録、ユーザーへは**丁寧な日本語**で案内

---

## 8. ChatGPT サポート（`gpt.gs` / `gpt-assistant.html`）

* **プロパティ**に `OPENAI_API_KEY` を設定
* `gpt.gs` の `askGPT(prompt, context)` が `UrlFetchApp.fetch` でAPIコール
* **レート制御**（Utilities.sleep & キャッシュ）/ **タイムアウト** / **最小プロンプト日本語**
* 画面下部の **? ヘルプ FAB** から

  * 項目説明、操作手順、入力例、エラーメッセージの意味を即時提示
  * 個人情報はマスク/省略（**プライバシー配慮**）

---

## 9. スタイル（`style.html`）

* **モノトーン**＋**メタリック**（陰影/グラデ/境界線）
* アクセントは**1色**（通知/強調）
* **高コントラスト**・フォーカスリング・タブ移動OK
* 文言：**日本語が主**、右肩に小さく英訳（例：登録 / Register）

---

## 10. ログ & 監査（`logger.gs`）

* `logInfo(tag, message, payload)` / `logError(tag, message, err)`
* 重要操作は**行ID/ユーザー/タイムスタンプ**を付与
* 例外の**ユーザー表示文**と**技術ログ**を分離（利用者を不安にさせない）

---

## 11. セキュリティ / 可用性

* **入力サニタイズ**（HTML特殊文字/改行）
* **CSRFトークン**埋め込み & 照合
* **権限**：`Session.getActiveUser().getEmail()` を監査に保存（要件に応じて）
* **排他制御**：`LockService.getDocumentLock()`
* **バックアップ**：夜間CSVエクスポート（Apps Script トリガ）
* **バージョニング**：`APP_ENV`でデプロイ先を切替、`/version` エンドポイントで確認

---

## 12. よくあるトラブルと即解（Checklist）

1. **遷移後に白い**

   * ファイル名ミス：`ui.hhtml` → `ui.html` に更正
   * `<?!= include('components/ui') ?>` など **include 名一致**を確認
   * `config.gs` の**二重定義**（`const CONFIG`の重複）→ **一度だけ定義**に修正（下記）
   * 例外時は `safeError.html` を返す実装に
2. **フォーム進まない**

   * `preventDefault()` 忘れ
   * `withFailureHandler` 未指定
   * バリデ失敗・例外→`logger.gs`に記録、画面には**優しい日本語**
3. **CONFIG 重複宣言**

   ```js
   // config.gs
   var CONFIG = this.CONFIG || {
     ENV: PropertiesService.getScriptProperties().getProperty('APP_ENV') || 'DEV',
     SHEET_ID: PropertiesService.getScriptProperties().getProperty('SPREADSHEET_ID'),
     OPENAI_API_KEY: PropertiesService.getScriptProperties().getProperty('OPENAI_API_KEY'),
     OPENAI_MODEL: PropertiesService.getScriptProperties().getProperty('OPENAI_MODEL') || 'gpt-4o-mini',
   };
   ```

   > `const CONFIG = ...` を複数ファイルで書かない。**再読み込みに強いパターン**に。
4. **テンプレ評価エラー**

   * `HtmlService.createTemplateFromFile(page)` に存在しない `page` を渡していないか
   * **ホワイトリスト**＋**not-found**ハンドリングで封じる

---

## 13. テスト手順（最短）

1. `start` で店舗/管理者を1件登録 → `Stores`/`Staff` 反映を確認
2. `menu → sales`：売上を1件登録 → `Sales` に行が増え、`total` 計算が正か
3. 同時二重送信テスト：送信連打で**重複行が出ない**（LockService効いている）
4. 通信失敗シミュレーション：ネットワーク切断 → 画面は**白化せず**エラーオーバーレイ表示
5. ChatGPTヘルプ：`支払い方法`の意味を質問 → 回答が返ること

---

## 14. 運用（保守・拡張）

* **新ページ追加**

  1. `xxx.html` 作成（`managementPage` レイアウト継承）
  2. `ALLOWED_PAGES` に `xxx` 追加
  3. `spreadsheet.gs` に `readXxx / createXxx` 追加
  4. `menu.html` にリンク追加
* **新しい選択肢**：`Meta_Lookups` へ行追加 → フォームに自動反映
* **列追加**：シートにヘッダ追加 → `spreadsheet.gs` のマッピング更新
* **週次バックアップ**：CSV自動吐き出しをGoogleドライブへ（トリガ）

---

## 15. デプロイ手順（安全版）

1. `APP_ENV=DEV` で機能確認
2. バージョン固定で**新規デプロイ**
3. `APP_ENV=PROD` に切替、稼働URLを**固定**
4. 変更は常に**新バージョン**を作成（上書きしない）

---

## 16. 最低限のコード指針（抜粋）

* **例外は握らない**：必ず `logger.gs` に流し、UIは**人に優しい**メッセージ
* **データ変換の一元化**：`utils.gs`（日付/数値/税計算/ID）
* **I/Oの一元化**：`spreadsheet.gs` だけがシートに触る
* **UIの一元化**：`style.html` + `components/ui.html`
* **MPAの堅牢化**：**ホワイトリスト・フォールバック・再試行**の3点セット
* **日本語優先**、英語は括弧で最小限（例：**登録（Register）**）

---

## 17. 既知課題と対処（今回の症状の直接原因候補）

* **ファイル名不一致**（`ui.hhtml` / `config.gsl`）→ テンプレ読込失敗 → **白画面**
  ⇒ **即リネーム**と `include()` 参照名の整合
* **CONFIGの重複宣言**（以前のログの通り）→ スクリプト評価失敗 → **白画面**
  ⇒ **単一宣言パターン**に統一
* **ルーティングの想定外 `page`** → `createTemplateFromFile` で **Bad value**
  ⇒ **ホワイトリスト**＋**not-found**テンプレ
* **`withFailureHandler` 未実装** → 失敗時にUI無反応
  ⇒ 全送信ポイントに必ず実装、失敗UIを明示

---

## 18. ライセンス / クレジット

* 内部利用前提（必要なら MIT 等に更新）
* OpenAI API 利用（組織ポリシーに従う）

---

## 19. 付録：運用メッセージ（怒らせないガイド）

* 成功：`✅ 登録しました。/ Saved.`
* 確認：`ℹ️ 入力内容をご確認ください。/ Please confirm your input.`
* 軽微エラー：`⚠️ 入力に不足があります。赤い項目をご確認ください。/ Some required fields are missing.`
* 重度エラー：`😢 ただいま処理がうまくいきませんでした。もう一度お試しください。改善しない場合は管理者にお知らせください。/ Something went wrong. Please try again.`

---
