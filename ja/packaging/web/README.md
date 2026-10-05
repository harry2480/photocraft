# PhotoCraft を Web でホスティングする

`photocraft-web-<version>.zip`（GitHub リリース、または `packaging/web/package.sh` で作成）には、
`photocraft-web-<version>/` 内に静的サイトが入っています。

| ファイル | 内容 |
|---|---|
| `index.html` | ページ本体。すべてを相対 URL で読み込みます。 |
| `photocraft-web-<hash>.js` | wasm-bindgen のグルーコード（自動生成、ES モジュール） |
| `photocraft-web-<hash>_bg.wasm` | アプリ本体。約 13 MB、圧縮時は 5 MB |
| `_headers`, `.htaccess` | Netlify/Cloudflare Pages および Apache 用のヘッダールールのサンプル |

サーバーサイドのコードはありません。フォルダーの中身を、静的ファイルを配信できる任意の場所にアップロードしてください。

## どのパスでも動作します

`index.html` 内の URL はすべて相対です（`apps/photocraft-web/Trunk.toml` の `public_url = "./"`）。
そのため、ドメインのルート（`https://example.com/`）、プレフィックス配下
（`https://example.com/tools/photocraft/`）、CDN バケットのいずれでも動作します。アセット名には
コンテンツハッシュが含まれるため、無期限にキャッシュできます。再検証が必要なのは `index.html` だけです。

## 必須のサーバー設定

- **MIME タイプ:** `.wasm` は `application/wasm` で配信してください。それ以外のタイプではブラウザが
  ストリーミングコンパイルを拒否し、アプリの読み込みが遅くなるか、まったく読み込まれません。`.js` は
  `text/javascript` で配信してください。多くのホストは両方とも設定済みです。nginx の場合は、`mime.types` に
  `application/wasm wasm;` があることを確認してください。
- **圧縮:** `.wasm`、`.js`、`.html` に対して gzip または Brotli を有効にしてください。ダウンロードサイズが
  約 13 MB から約 5 MB になります。事前圧縮（`brotli -k *.wasm`）しておき、サーバーに
  `Content-Encoding: br` を送らせることもできます。
- **キャッシュ:** ハッシュ付きの `.wasm` と `.js` には `Cache-Control: public, max-age=31536000, immutable`、
  `index.html` には `no-cache` を設定してください。
- **HTTPS:** WebGPU（およびクリップボード）はセキュアコンテキスト、つまり `https://` または
  `http://localhost` でのみ動作します。それ以外の平文 HTTP では、アプリは WebGL2 にフォールバックします。
- **特別な分離ヘッダーは不要:** PhotoCraft は `SharedArrayBuffer` を使わないため、
  `Cross-Origin-Opener-Policy` や `Cross-Origin-Embedder-Policy` は不要です。サイトがすでに COEP
  `require-corp` を送っている場合は、アプリのファイルに `Cross-Origin-Resource-Policy: same-origin`
  （ファイルが CDN 上にある場合は `cross-origin`）も付けてください。

nginx の例:

```nginx
location /photocraft/ {
    types { application/wasm wasm; text/javascript js; text/html html; }
    gzip on;
    gzip_types application/wasm text/javascript text/html;
    location ~* \.(wasm|js)$ { add_header Cache-Control "public, max-age=31536000, immutable"; }
    location ~* index\.html$ { add_header Cache-Control "no-cache"; }
}
```

ローカルでのテスト: フォルダー内で `python3 -m http.server 8765` を実行し、http://localhost:8765/ を開きます。

## ページへの埋め込み（iframe）

```html
<iframe
  src="https://example.com/photocraft/"
  title="PhotoCraft image editor"
  style="width: 100%; height: 720px; border: 0;"
  allow="fullscreen; clipboard-read; clipboard-write"
  allowfullscreen>
</iframe>
```

- アプリは iframe いっぱいに表示され、そのサイズに追従します。サイズはアプリではなく iframe 側で指定してください。
- キーボードショートカットは、他の埋め込みアプリと同様に、ユーザーが iframe 内をクリックした後に iframe へ送られます。
- **クロスオリジンでの埋め込み**も動作します。環境設定は iframe の `localStorage` に保存されます。
  サードパーティストレージを分離またはブロックするブラウザでは、訪問のたびに設定が失われることがあり、
  その場合アプリはデフォルト設定で起動します。
- **サンドボックス化された iframe** には最低限
  `sandbox="allow-scripts allow-same-origin allow-downloads allow-popups"` が必要です。
  `allow-same-origin` がないとストレージが使えません。`allow-downloads` がないと、保存と書き出し
  （ブラウザのダウンロード）がブロックされます。
- `X-Frame-Options: DENY` や、埋め込み元ページを除外する `frame-ancestors` CSP を送らないでください。

## レンダラーの選択とフォールバック用フラグ

PhotoCraft は wgpu で描画します。ブラウザが対応していれば **WebGPU** を使い、そうでなければ自動的に
**WebGL2** にフォールバックします。URL のクエリフラグでこれを上書きでき、iframe の `src` でも使えます。

| フラグ | 効果 |
|---|---|
| *（なし）* | 利用可能なら WebGPU、それ以外は WebGL2 |
| `?webgl` | WebGL2 バックエンドを強制（WebGPU ドライバーの挙動がおかしい場合に有用） |
| `?cpu` | CPU キャンバスパスを強制（最も遅いが、最も互換性が高い） |

例: `<iframe src="https://example.com/photocraft/?webgl" ...>`

WebGPU と WebGL2 のどちらにも対応していないブラウザでは、アプリの代わりにメッセージが表示されます。
