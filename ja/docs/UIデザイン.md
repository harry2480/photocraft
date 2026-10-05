# UI デザインシステム

## テーマ

| テーマ | 意図 |
|---|---|
| **Pro**（デフォルト） | Photoshop 風の Spectrum ダーク：フラットなチャコールのパネル（#323232）、暗いタブストリップ、Spectrum ブルーのアクセント（#378ef0）、ピル型ボタン、チェックボックス、コンパクトな 12 px の文字 |
| Studio | ダークなスタジオ風：ほぼ黒、角丸のカード、ピル型タブ、バイオレットのアクセント、トグル |
| Studio Light | 明るいサーフェス上の Studio |
| Classic | Windows 2000 風のベベル、角張った角、ネイビーの選択色 |

テーマは太陽アイコン、ウィンドウ → テーマ、またはコントロールチャンネル経由の `ui.set {"theme":"classic"}` で切り替える。

## ルール

- **色と角丸はトークンから取得する。** `Tokens::get(ctx)` で読み取る。ウィジェット内で色をハードコードしてはならない。
- **共有ウィジェットは `widgets.rs` にある：** `card`（パネルグループ。Pro では Photoshop のタブストリップを描画する）、`value_field`、`slider`/`slider_row`、`toggle`/`checkbox`、`primary_button`/`secondary_button`、`dropdown`（シェブロンアイコン付き）、`hairline`/`vline`。
- **アイコン：** `assets/icons` にある Lucide の SVG（ISC ライセンス）を `icon_data.rs` 経由で埋め込む。アイコンを追加したらこのファイルを再生成する。
- **フォント：** Inter（UI）と JetBrains Mono（数値）。どちらも OFL。名前付きファミリー `medium` と `semibold` を `theme::medium()` と `theme::semibold()` で利用できる。
- **Photoshop のレイアウト文法（Pro）：**
  - 初期設定（Essentials）のドック順：カラー | スウォッチ、次にプロパティ | 色調補正、次にレイヤー | チャンネル | パス（レイヤーが残りの高さを埋める）。
  - オプションバーのラベルはコロンで終わる（"Size:"）。
  - ドキュメントタブの表記は "name @ 12.5% (RGB/8)"。
  - ツールバーのツールグループには角に三角形が付く。
- **すべての視覚的な変更を確認する。** `ui.screenshot` を使い、複数のウィンドウサイズとすべてのテーマで確認する。

## アイコンの追加

```sh
curl -sfL -o assets/icons/<name>.svg https://raw.githubusercontent.com/lucide-icons/lucide/main/icons/<name>.svg
# regenerate the embedded table
{ printf '%s\n' '//! Lucide icons (ISC licence), embedded and tinted at runtime.' '' 'pub static ICONS: &[(&str, &[u8])] = &['; \
  for f in assets/icons/*.svg; do n=$(basename $f .svg); echo "    (\"$n\", include_bytes!(\"../../../assets/icons/$n.svg\")),"; done; echo '];'; } \
  > crates/ui-egui/src/icon_data.rs
```

## インタラクションモデル（Photoshop CC に合わせる）

| 機能 | モジュール | 振る舞い |
|---|---|---|
| 文字ツール | `type_tool.rs` | クリック：プレースホルダー「Lorem Ipsum」が選択された状態でポイントテキストを作成。ドラッグ：段落ボックス。インラインのキャレットと選択範囲はテキストエンジンのレイアウトから描画する。⌥/⌘ で単語・行単位の移動、↩ で改行、⌘↩ または Esc で確定、外側をクリックしても確定。1 セッションにつきヒストリーは 1 ステップ（`coalesce`）。新しいレイヤーはそのテキストにちなんで命名され、空のレイヤーは削除される。 |
| 自由変形 | `transform_tool.rs` | ⌘T。コーナーは縦横比を保って拡大・縮小（⇧ で解除）、辺は 1 軸のみ拡大・縮小、⌥ で基準点を中心に拡大・縮小、⌘ + コーナーで自由な形に、外側をドラッグで回転（⇧ で 15° 単位にスナップ）、内側をドラッグで移動。プレビュー = 移動中のピクセルを除いたドキュメント + テクスチャ付きの 24×24 メッシュ。↩ またはダブルクリックで `edit.transform {rect, quad}` により確定。 |
| レイヤーマスク | `panels.rs` | マスクのサムネイルをクリックするとマスクが対象になる（角括弧状の枠。タブの表記は "Layer, Layer Mask/8"）。以降、ブラシ、消しゴム（背景色で描画）、グラデーション、塗りつぶしツールは `"target": "mask"` を送る。調整レイヤーと塗りつぶしレイヤーは自動的にマスクを対象にする。 |
| レベル補正／トーンカーブ | `tone.rs` | 調整の *下にある* 画像のヒストグラム。トーンカーブ：クリックでポイントを追加、外へドラッグで削除。すべての変更は coalesce された `layer.setAdjustment` なので、1 回のドラッグ = 1 回のアンドゥステップとなり、キャンバスは GPU 上でフル解像度で更新される。 |

フォントに記号がない場合（例：∠ ↦ ▔）は、ペインターで描画するか Lucide のアイコンを使う。欠落グリフの
四角を出荷してはならない。新しいパネルはすべてオフスクリーンのスナップショットツールで確認する（`docs/development.md`）。

## メニュー

`menu_catalog.rs` には Photoshop のメニューツリー（標準のコマンド名、順序、区切り線、デフォルトのショートカット）が収められている。id がエンジンまたは UI のコマンドと一致する項目は有効になり、それ以外は実装されるまで無効状態で描画される。新しいコマンドにカタログの id（たとえば `image.imageSize`）を付ければ、自動的に正しい場所で有効になる。

## 視覚的確認のための自動化

`ui.click {x,y}`、`ui.move`、`ui.key`、`ui.type`（スクリーンポイント単位の合成入力）を使ってメニュー、ポップアップ、コンテキストメニューを開き、その後 `ui.screenshot` を撮る。
