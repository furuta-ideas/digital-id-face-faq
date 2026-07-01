# デジタルID×顔認証 FAQ

顔認証・デジタルID関連の **現場サポート用 FAQ 検索ツール**。単一HTMLファイルとして配布可能、外部依存はWebフォントのみ（CDN）。

## ファイル構成

| ファイル | 役割 |
| --- | --- |
| `index.html` | **配布物**。単独で動作するアプリ本体（FAQデータベース埋め込み済み） |
| `template.html` | HTMLテンプレート。`/*__FAQ_DB__*/`プレースホルダがビルド時に置換 |
| `parse_xlsx.ps1` | Excelファイル（.xlsx）→ 中間JSON（`faq_raw.json`）の変換 |
| `build.ps1` | 中間JSONから `index.html` を生成 |
| `faq_raw.json` | Excel `★マスターFAQ` シート由来の中間生成物（311件） |
| `serve.ps1` | ローカル動作確認用の軽量HTTPサーバ |
| `apple-touch-icon.png` | iOSホーム画面用アイコン |
| `.claude/launch.json` | Claude Code preview起動設定 |

## 仕様

| 項目 | 値 |
| --- | --- |
| バージョン | `20260608_顔認証FAQ_Rev15` (Excel `20260608_顔認証FAQ_Rev15.xlsx` 由来) |
| 初期パスワード | `2026` |
| ロックアウト | 3回連続失敗で30秒間 |
| 言語切替 | 全画面右上 **JP / EN** トグル。初期値は日本語、選択はlocalStorageに保存 |
| 検索アルゴリズム | 全角半角・大文字小文字・カタカナ／ひらがなを正規化 → スコア順 |
| 表示件数 | 1ページ最大4件、「他の質問を見る」で次ページ |
| 回答表示 | 14pt太字（モバイルでは12pt）、`●` 区切り最大3件 |
| 音声入力 | Web Speech API（日本語） |

## データソース

Excel `20260608_顔認証FAQ_Rev15.xlsx` の `★マスターFAQ` シートのみを使用。他のシートは取り込みません。

| 列 | 内容 |
| --- | --- |
| A | No |
| B | カテゴリー |
| C | 質問（Q） |
| D | 模範回答（A） |
| E | 引用元（取り込み対象外） |
| F | 備考（取り込み対象外） |

## Excel更新時の再ビルド手順

```powershell
# ① xlsx を zip にリネーム → 展開ディレクトリへ
Copy-Item .\20260608_顔認証FAQ_Rev15.xlsx $env:TEMP\faq.zip
Expand-Archive $env:TEMP\faq.zip $env:TEMP\faq_extract -Force

# ② 中間JSONを作成（★マスターFAQ＝sheet1）
.\parse_xlsx.ps1 -ExtractDir $env:TEMP\faq_extract -OutJson .\faq_raw.json

# ③ index.html を生成
.\build.ps1
```

## 動作確認

```powershell
.\serve.ps1 -Port 8770
# ブラウザで http://localhost:8770/ を開く
```

## 対応ブラウザ

- Chrome 90+（推奨：音声入力対応）
- Safari 14+
- Edge 90+
- Firefox（音声入力は非対応の旨が表示されます）
