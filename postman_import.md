# SimpleShort API - Postmanインポート手順

---

## 方法①：Swagger Editor → Postman の直接インポート

1. [https://editor.swagger.io/](https://editor.swagger.io/) にアクセス
2. 左に `openapi.yaml` を貼り付け
3. メニューから `File > Export > Download JSON` を選択

---

## 方法②：Postmanに読み込む

1. Postman を起動
2. 「Import」ボタン → 「File」タブを選択
3. 先ほどダウンロードした `swagger.json` を読み込み
4. 自動的にコレクションが生成される

---

## 補足

- 読み込み後は環境変数を設定すると `{{baseUrl}}/api/register` のように使えます
- `Authorization` は不要（このAPIは認証なし）

---

## よく使うエンドポイント例（Postman上）

| メソッド | URL                        | 備考               |
|----------|----------------------------|--------------------|
| POST     | `{{baseUrl}}/api/register` | JSON Body付きで登録 |
| GET      | `{{baseUrl}}/r/abc123`     | リダイレクト確認   |
