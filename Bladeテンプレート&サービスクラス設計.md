# SimpleShort Bladeテンプレート & サービスクラス設計

---

## 1. Bladeテンプレート `resources/views/index.blade.php`

```blade
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>SimpleShort - URL短縮</title>
</head>
<body>
    <h1>URL短縮サービス</h1>

    {{-- 登録フォーム --}}
    <form method="POST" action="{{ route('web.register') }}">
        @csrf
        <label>元のURL:</label><br>
        <input type="text" name="original_url" required><br><br>

        <label>短縮ID（任意、日本語可）:</label><br>
        <input type="text" name="custom_id"><br><br>

        <button type="submit">登録</button>
    </form>

    <hr>

    {{-- 登録済みURL一覧 --}}
    <h2>登録済みURL一覧</h2>
    <ul>
        @foreach ($urls as $id => $info)
            <li>
                <strong>ID:</strong> {{ $id }}<br>
                <strong>URL:</strong> <a href="{{ $info['original_url'] }}" target="_blank">{{ $info['original_url'] }}</a><br>
                <strong>短縮リンク:</strong> <a href="{{ url('/r/' . urlencode($id)) }}" target="_blank">{{ url('/r/' . urlencode($id)) }}</a><br>
                <strong>登録日時:</strong> {{ $info['created_at'] }}<br><br>
            </li>
        @endforeach
    </ul>
</body>
</html>
```

---

## 2. サービスクラス `app/Services/UrlService.php`

```php
<?php

namespace App\Services;

use Illuminate\Support\Facades\File;
use Illuminate\Support\Facades\Storage;
use Illuminate\Support\Str;
use Carbon\Carbon;

class UrlService
{
    protected string $file;

    public function __construct()
    {
        $this->file = storage_path('short_urls.json');
        if (!file_exists($this->file)) {
            file_put_contents($this->file, json_encode([], JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE));
        }
    }

    public function load(): array
    {
        return json_decode(file_get_contents($this->file), true) ?? [];
    }

    public function save(array $data): void
    {
        file_put_contents($this->file, json_encode($data, JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE));
    }

    public function create(string $originalUrl, ?string $customId = null): string
    {
        $id = $customId ?? Str::random(6);
        $id = trim($id);

        $urls = $this->load();

        if (array_key_exists($id, $urls)) {
            throw new \Exception('ID already exists');
        }

        $urls[$id] = [
            'original_url' => $originalUrl,
            'created_at' => Carbon::now()->toIso8601String(),
            'last_accessed_at' => Carbon::now()->toIso8601String(),
        ];

        $this->save($urls);

        return $id;
    }

    public function find(string $id): ?string
    {
        $urls = $this->load();

        if (!array_key_exists($id, $urls)) {
            return null;
        }

        // アクセス時に更新
        $urls[$id]['last_accessed_at'] = Carbon::now()->toIso8601String();
        $this->save($urls);

        return $urls[$id]['original_url'];
    }

    public function getAll(): array
    {
        return $this->load();
    }
}
```

---

## 3. ルーティング例（抜粋）

`routes/web.php`

```php
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\WebController;

Route::get('/', [WebController::class, 'index']);
Route::post('/register', [WebController::class, 'register'])->name('web.register');
Route::get('/r/{id}', [WebController::class, 'redirect']);
```

---

## 4. コントローラ例（Web用）

`app/Http/Controllers/WebController.php`

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Services\UrlService;

class WebController extends Controller
{
    protected UrlService $service;

    public function __construct(UrlService $service)
    {
        $this->service = $service;
    }

    public function index()
    {
        $urls = $this->service->getAll();
        return view('index', ['urls' => $urls]);
    }

    public function register(Request $request)
    {
        $request->validate([
            'original_url' => 'required|url',
            'custom_id' => 'nullable|max:20|regex:/^[\p{L}\p{N}]+$/u',
        ]);

        try {
            $this->service->create(
                $request->input('original_url'),
                $request->input('custom_id')
            );
        } catch (\Exception $e) {
            return back()->withErrors(['custom_id' => 'すでに使用されているIDです']);
        }

        return redirect('/');
    }

    public function redirect($id)
    {
        $decodedId = urldecode($id);
        $url = $this->service->find($decodedId);

        if (!$url) {
            abort(404);
        }

        return redirect($url);
    }
}
```

---
