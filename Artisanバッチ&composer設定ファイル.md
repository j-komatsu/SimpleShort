# SimpleShort Artisanバッチ & composer設定ファイル

---

## 1. Artisanバッチ処理 `cleanup:old-urls` の完全実装

### コマンド作成

```bash
php artisan make:command CleanupOldUrls
```

作成されるファイル：  
`app/Console/Commands/CleanupOldUrls.php`

### 実装内容（1週間以上アクセスされていないURLを削除）

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Support\Carbon;

class CleanupOldUrls extends Command
{
    protected $signature = 'cleanup:old-urls';
    protected $description = '1週間以上アクセスされていない短縮URLを削除する';

    public function handle()
    {
        $filePath = storage_path('short_urls.json');

        if (!file_exists($filePath)) {
            $this->info('URLデータファイルが存在しません。');
            return;
        }

        $data = json_decode(file_get_contents($filePath), true);
        $now = Carbon::now();
        $threshold = $now->copy()->subDays(7);

        $beforeCount = count($data);

        $filtered = array_filter($data, function ($item) use ($threshold) {
            if (!isset($item['last_accessed_at'])) return true; // 古い形式の安全対策
            return Carbon::parse($item['last_accessed_at'])->greaterThan($threshold);
        });

        file_put_contents($filePath, json_encode($filtered, JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE));

        $afterCount = count($filtered);
        $removed = $beforeCount - $afterCount;

        $this->info("完了：{$removed} 件を削除しました。残り {$afterCount} 件。");
    }
}
```

### 実行コマンド

```bash
php artisan cleanup:old-urls
```

---

## 2. `composer.json` の設定（Laravel 10 以降対応）

Laravel新規作成時に生成されるファイルをベースに、**URL短縮サービス向けの設定例**として記載します。

```json
{
    "name": "yourname/simpleshort",
    "description": "A simple Laravel-based URL shortener without a database.",
    "type": "project",
    "require": {
        "php": "^8.1",
        "laravel/framework": "^10.0"
    },
    "require-dev": {
        "fakerphp/faker": "^1.9.1",
        "phpunit/phpunit": "^10.0",
        "laravel/pint": "^1.0"
    },
    "autoload": {
        "psr-4": {
            "App\\": "app/"
        }
    },
    "scripts": {
        "post-autoload-dump": [
            "Illuminate\\Foundation\\ComposerScripts::postAutoloadDump",
            "@php artisan package:discover --ansi"
        ],
        "test": "phpunit"
    },
    "minimum-stability": "stable",
    "prefer-stable": true
}
```

---

## 3. 補足：依存関係インストール手順

```bash
# 依存パッケージのインストール
composer install

# Laravelのアプリキー生成（初回のみ）
php artisan key:generate
```

---
