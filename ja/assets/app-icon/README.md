# PhotoCraft アプリアイコン

**額にダイヤモンドを持つ九尾の狐**: 版画（エングレービング）風の肖像で、座って振り返り、見る人のほうを
向いています。Crafting Apps 共通のフクロウテンプレートの構図（全面を塗りつぶした背景、タイルいっぱいに
頭と体を配置し、尾は端からはみ出す）に沿っています。

## パレット

使う色はちょうど 3 色です。

| 色 | Hex | 用途 |
|---|---|---|
| インク | `#0b0b0c` | 線画と輪郭 |
| ペーパー | `#efe9dc` | 図像 |
| PhotoCraft ブルー（アプリカラー） | `#2f7bf5` | 全面の背景 |

## ジオメトリ

512 単位のタイル（`viewBox="0 0 512 512"`）で、`rx=112` の角丸正方形、枠線なしです。macOS 向けの
レンダリングでは Apple の 824/1024 アイコングリッドに合わせて余白を付け、Windows と Linux 向けの
レンダリングでは各辺から 22 単位を切り落として、16–48 px でも図像が判別できるようにしています。

## 出所

オーナーによるオリジナルの描画で、ArtCraft で制作し（2880 px、版画スタイル、パレットに合わせて色調整）、
craftrules の `assets/logo-options/_tools/vectorize_tile.py` でベクター化しました。元の PNG は craftrules の
`assets/app-icons/photocraft/source.png` に置かれています。ライセンスは `LICENSE.txt` を参照してください。

## ファイル

- `photocraft.svg`: 正式なマスター（2048 px でトレース）。
- `photocraft-small.svg`: 軽量なトレース（1024 px でトレース）。hicolor の scalable アイコンとしても使用。
- `photocraft-1024.png`: macOS グリッド上での 1024 px レンダリング。
- `photocraft.icns`: macOS バンドルのアイコン（`CFBundleIconFile`）。
- `photocraft.ico`: Windows アイコン（16–256 px）。`apps/photocraft/build.rs` が `.exe` に埋め込みます。
- `hicolor/<size>/apps/ai.storyteller.photocraft.png`（16–512）と `hicolor/scalable/...svg`:
  Linux のアイコンテーマ。

アプリは実行時にウィンドウアイコンと Wayland のアプリ ID も設定します（`apps/photocraft/src/main.rs`）。

## 再生成

`photocraft.svg`（と `photocraft-small.svg`）を差し替えてから、`packaging/icons.sh` を実行します。
`resvg` が必要で、macOS では `iconutil` も必要です。`.ico` は `cargo xtask ico` でパックされます。
