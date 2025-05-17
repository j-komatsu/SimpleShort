# SimpleShort API用Controller & PHPUnitテストコード

---

## 1. API用コントローラ `app/Http/Controllers/ApiController.php`

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Http\JsonResponse;
use App\Services\UrlService;

class ApiController extends Controller
{
    protected UrlService $service;

    public function __construct(UrlService $service)
    {
        $this->service = $service;
    }

    public function register(Request $request): JsonResponse
    {
        $validated = $request->validate([
            'original_url' => 'required|url',
            'custom_id' => 'nullable|max:20|regex:/^[\p{L}\p{N}]+$/u',
        ]);

        try {
            $id = $this->service->create(
                $validated['original_url'],
                $validated['custom_id'] ?? null
            );
        } catch (\Exception $e) {
            return response()->json([
                'error' => '指定されたIDはすでに使用されています。'
            ], 409);
        }

        return response()->json([
            'short_url' => url('/r/' . urlencode($id)),
        ]);
    }

    public function redirect($id)
    {
        $decodedId = urldecode($id);
        $url = $this->service->find($decodedId);

        if (!$url) {
            return response()->json(['error' => 'Not found.'], 404);
        }

        return redirect($url);
    }
}
```

---

## 2. ルーティング定義（API用） `routes/api.php`

```php
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\ApiController;

Route::post('/register', [ApiController::class, 'register']);
Route::get('/r/{id}', [ApiController::class, 'redirect']);
```

---

## 3. PHPUnitテスト `tests/Feature/ApiRegisterTest.php`

```php
<?php

namespace Tests\Feature;

use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Support\Facades\File;
use Tests\TestCase;

class ApiRegisterTest extends TestCase
{
    protected function setUp(): void
    {
        parent::setUp();

        // テスト実行前にJSONファイルを空にする
        $path = storage_path('short_urls.json');
        if (file_exists($path)) {
            file_put_contents($path, json_encode([]));
        }
    }

    public function test_register_api_returns_short_url()
    {
        $response = $this->postJson('/api/register', [
            'original_url' => 'https://example.com',
        ]);

        $response->assertStatus(200)
                 ->assertJsonStructure(['short_url']);
    }

    public function test_register_api_with_custom_id()
    {
        $response = $this->postJson('/api/register', [
            'original_url' => 'https://example.com/page',
            'custom_id' => 'おすすめリンク'
        ]);

        $response->assertStatus(200)
                 ->assertJson([
                     'short_url' => url('/r/' . urlencode('おすすめリンク'))
                 ]);
    }

    public function test_register_api_with_duplicate_id_returns_error()
    {
        // 1回目
        $this->postJson('/api/register', [
            'original_url' => 'https://example.com/1',
            'custom_id' => '重複テスト'
        ]);

        // 2回目（重複）
        $response = $this->postJson('/api/register', [
            'original_url' => 'https://example.com/2',
            'custom_id' => '重複テスト'
        ]);

        $response->assertStatus(409);
    }
}
```

---

## 4. テスト実行コマンド

```bash
php artisan test
# または
vendor/bin/phpunit
```

---
