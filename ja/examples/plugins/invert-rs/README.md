# Invert: PhotoCraft プラグインのサンプル

約 70 行の `no_std` Rust で書かれた、完全な WebAssembly フィルタープラグインです。アロケーターもインポートも
なく、ビルド後のサイズは 1 KiB 未満です。カラーチャンネルを反転し、アルファはそのまま残します。これは
イメージ › 色調補正 › 階調の反転 と同じ動作です。ABI は [`docs/plugins.md`](../../../docs/プラグイン.md) に記載されています。

## ビルド

```sh
rustup target add wasm32-unknown-unknown
./build.sh
```

`build.sh` は `cargo build --release --target wasm32-unknown-unknown` を実行し、モジュールを
`crates/plugins/tests/fixtures/invert.wasm`（ホスト側のテストが使うテストフィクスチャ）にコピーします。
このクレートは PhotoCraft ワークスペースのメンバーではありません。

## インストール

```sh
# Into a running app (control channel) or the CLI, as a command:
plugin.install {"path": "examples/plugins/invert-rs/target/wasm32-unknown-unknown/release/photocraft_plugin_invert.wasm"}
plugin.run {"id": "org.photocraft.example.invert"}
```

または、編集 › 環境設定 › プラグイン で設定したフォルダーに `.wasm` をコピーします（*追加のプラグインフォルダー*
を有効にしておく必要があります）。するとフィルター › プラグイン の下に表示されます。
