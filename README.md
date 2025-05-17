# SimpleShort

Laravel製・データベース不要のシンプルなURL短縮サービス。

- ✅ 日本語IDも使える短縮URL
- ✅ REST API + Web画面対応
- ✅ JSONファイルでデータ保存
- ✅ 1週間未使用リンクを削除するバッチ処理付き

---

## 📦 技術スタック

| 項目           | 内容                  |
|----------------|-----------------------|
| 言語           | PHP 8.1+              |
| フレームワーク | Laravel 10.x          |
| テスト         | PHPUnit               |
| DB             | 使用しない            |
| データ保存     | JSONファイル（storage配下）|

---

## 🚀 セットアップ手順

### 1. リポジトリクローン

```bash
git clone https://github.com/yourname/simpleshort.git
cd simpleshort
```

### 2. パッケージインストール

```bash
composer install
```

### 3. .env 作成とキー生成

```bash
cp .env.example .env
php artisan key:generate
```

### 4. 開発サーバー起動

```bash
php artisan serve
```

ブラウザでアクセス：
```
http://localhost:8000
```

---

## 💡 機能一覧

| 区分    | 機能概要                                                                 |
|---------|--------------------------------------------------------------------------|
| Web画面 | フォームによるURL登録・日本語ID入力・一覧表示                          |
| API     | `/api/register` でURL登録、`/r/{id}`でリダイレクト                       |
| バッチ  | `php artisan cleanup:old-urls` で1週間以上未使用のURLを削除              |
| 保存先  | `storage/short_urls.json` にすべてのデータを保存                         |

---

## 📘 API仕様

### 登録エンドポイント

```http
POST /api/register
Content-Type: application/json
```

#### リクエスト例

```json
{
  "original_url": "https://example.com",
  "custom_id": "おすすめリンク"
}
```

#### レスポンス例

```json
{
  "short_url": "http://localhost/r/%E3%81%8A%E3%81%99%E3%81%99%E3%82%81%E3%83%AA%E3%83%B3%E3%82%AF"
}
```

#### エラーパターン（ID重複）

```json
HTTP/1.1 409 Conflict
{
  "error": "指定されたIDはすでに使用されています。"
}
```

---

## 🧪 テスト実行

```bash
php artisan test
# または
vendor/bin/phpunit
```

---

## 🧹 バッチ処理実行

```bash
php artisan cleanup:old-urls
```

---

## 📁 ディレクトリ構成

```
/app
  ├── Services/UrlService.php
  └── Http/Controllers/
        ├── ApiController.php
        └── WebController.php
/resources/views/index.blade.php
/routes/web.php
/routes/api.php
/storage/short_urls.json
```

---

## 📝 ライセンス

MIT License
