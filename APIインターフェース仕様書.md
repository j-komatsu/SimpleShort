# SimpleShort APIインターフェース仕様書

---

## API名：短縮URL登録

### エンドポイント

```
POST /api/register
```

### 説明

指定された元のURLを短縮IDで登録します。短縮IDは任意で指定可能で、日本語も使用できます（URLエンコード対応）。

---

### リクエスト

#### Headers

| Key             | Value              |
|------------------|-------------------|
| Content-Type     | application/json  |

#### Body（JSON形式）

| パラメータ     | 型       | 必須 | 説明                                     |
|----------------|----------|------|------------------------------------------|
| original_url    | string   | ○    | 短縮対象となるURL                         |
| custom_id       | string   | ×    | 任意の短縮ID。日本語も可（最大20文字）     |

#### リクエスト例（自動ID生成）

```json
{
  "original_url": "https://example.com/some/very/long/url"
}
```

#### リクエスト例（任意ID指定・日本語）

```json
{
  "original_url": "https://example.com/info",
  "custom_id": "おすすめリンク"
}
```

---

### バリデーション仕様

| 項目         | ルール                              | 補足                        |
|--------------|--------------------------------------|-----------------------------|
| original_url | 必須、正当なURL形式                  | `url` バリデーション使用     |
| custom_id    | 任意、最大20文字、日本語可           | `regex:/^[\p{L}\p{N}]+$/u`  |

---

### レスポンス

#### 成功時（200 OK）

```json
{
  "short_url": "http://localhost/r/%E3%81%8A%E3%81%99%E3%81%99%E3%82%81%E3%83%AA%E3%83%B3%E3%82%AF"
}
```

#### エラー時：ID重複（409 Conflict）

```json
{
  "error": "指定されたIDはすでに使用されています。"
}
```

#### エラー時：入力不備（422 Unprocessable Entity）

```json
{
  "message": "The given data was invalid.",
  "errors": {
    "original_url": [
      "The original url field is required."
    ]
  }
}
```

---

## API名：短縮URLアクセス（リダイレクト）

### エンドポイント

```
GET /r/{shortId}
```

### 説明

指定された `shortId` に対応する元のURLへリダイレクトします。日本語IDはURLエンコードで指定。

---

### パスパラメータ

| パラメータ名 | 説明                  | 型     |
|--------------|-----------------------|--------|
| shortId      | 登録済みの短縮ID       | string |

---

### レスポンス

#### 成功時（302 Found）

- リダイレクト先：`original_url` に自動転送

#### 失敗時（404 Not Found）

```json
{
  "error": "Not found."
}
```

---

## 共通仕様

### 認証

- 本APIは認証不要（オープンAPI）

### エンコード対応

- `custom_id` の値に日本語を使用する場合は自動でURLエンコードされます
- 取得・アクセス時には `urldecode()` で処理されます

---

## 補足事項

- 短縮URLの有効期限はありません（削除されるまで保持されます）
- バッチ処理により、最終アクセスから7日以上経過したデータは削除対象となります

---

以上が本システムのREST API仕様となります。
