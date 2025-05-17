# Swagger UI 静的HTML出力手順（SimpleShort API用）

---

## 手順①：Swagger Editor に YAML を貼り付ける

1. https://editor.swagger.io/ にアクセス
2. 左ペインに `openapi.yaml` をペースト
3. 右ペインに Swagger UI がリアルタイム表示される

---

## 手順②：HTMLとしてエクスポート

### 方法A：ブラウザで保存

- 「右クリック → ページを名前を付けて保存」
  - 種類：**Webページ、完全（.html + .files）**
  - 例：`simpleshort-api.html`

### 方法B：Swagger Editor の `Generate Client` 機能を利用し、出力を一部加工

---

## 補足：自分でホスティングする場合

公式テンプレートに `openapi.yaml` を読み込ませるだけでOK：

```html
<!-- index.html -->
<!DOCTYPE html>
<html>
<head>
  <title>SimpleShort API</title>
  <link rel="stylesheet" href="https://unpkg.com/swagger-ui-dist/swagger-ui.css" />
</head>
<body>
  <div id="swagger-ui"></div>
  <script src="https://unpkg.com/swagger-ui-dist/swagger-ui-bundle.js"></script>
  <script>
    window.onload = () => {
      SwaggerUIBundle({
        url: "openapi.yaml",
        dom_id: "#swagger-ui"
      });
    };
  </script>
</body>
</html>
```

- この `index.html` と `openapi.yaml` を同じフォルダに配置し、ローカル or GitHub Pagesなどで公開可能

---

## 注意点

- `url:` に指定するファイルは **CORS制限のないURL** であること（ローカルで動かすときは `file://` でも可）
- YAMLファイルに文法エラーがあると画面表示されません
