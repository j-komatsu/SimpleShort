# URL短縮サービス「SimpleShort」 設計書

## 1. アプリケーション概要

- 長いURLを短縮IDで置き換え、アクセスしやすくするWebアプリ
- 任意のID指定（日本語含む）も可能
- データはDBではなくJSONファイルに保存
- Web画面、REST API、バッチ処理を含む

---

## 2. 技術スタック

| 項目              | 内容                             |
|-------------------|----------------------------------|
| 言語              | PHP                              |
| フレームワーク    | Laravel                          |
| テスト            | PHPUnit                          |
| データ保存方法    | JSONファイル（storage/配下）     |
| DB                | なし                             |

---

## 3. 主な機能一覧

### 3.1 REST API

| メソッド | パス              | 機能内容                                           |
|----------|-------------------|----------------------------------------------------|
| POST     | `/api/register`   | URL登録（短縮IDを指定または自動生成）             |
| GET      | `/r/{shortId}`    | 短縮URLアクセス → 元URLへリダイレクト             |

### 3.2 Web画面

| パス     | 内容                                                  |
|----------|-------------------------------------------------------|
| `/`      | URL登録フォームおよび登録済み一覧を表示する画面     |

### 3.3 バッチ処理（Artisanコマンド）

| コマンド名             | 処理内容                                      |
|------------------------|-----------------------------------------------|
| `cleanup:old-urls`     | 1週間以上アクセスされていないURLを削除        |

---

## 4. データ構造（storage/short_urls.json）

```json
{
  "おすすめリンク": {
    "original_url": "https://example.com/longpage",
    "created_at": "2025-05-17T08:00:00+09:00",
    "last_accessed_at": "2025-05-17T10:00:00+09:00"
  },
  "abc123": {
    "original_url": "https://another.com",
    "created_at": "2025-05-10T08:00:00+09:00",
    "last_accessed_at": "2025-05-11T08:00:00+09:00"
  }
}
```

---

## 5. バリデーション仕様

- 短縮IDは最大20文字まで
- 使用できる文字：英数字、日本語（記号や空白を除く）
- すでに登録済みのIDを指定した場合は `409 Conflict` を返す

---

## 6. ディレクトリ構成（予定）

```
/routes
  ├── web.php
  └── api.php

/app
  ├── Http
  │   ├── Controllers
  │   │   ├── ApiController.php
  │   │   └── WebController.php
  └── Services
      └── UrlService.php

/resources/views
  └── index.blade.php

/storage/short_urls.json

/tests
  └── Feature/UrlServiceTest.php
```

---

## 7. 今後の拡張案（任意）

- QRコード生成機能
- 短縮URLの有効期限設定
- 使用回数の記録
- IPアドレスによる制限付き共有
