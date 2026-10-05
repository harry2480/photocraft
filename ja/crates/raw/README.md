# photocraft-raw

クリーンルームで実装された、純 Rust のカメラ RAW デコーダー兼現像エンジンです。このクレートは独立しており（ワークスペース内の
依存なし）、`unsafe` を持たず、I/O を行わず（入力は `&[u8]`）、`wasm32-unknown-unknown` 向けにビルドでき
（そこでは逐次処理、ネイティブでは rayon による並列処理）、敵対的な入力に対しても決してパニックしません：すべてのオフセットは
境界チェックされ、サイズは確保前に `Limits` と照合されます。

```rust
use photocraft_raw::{develop, DevelopOptions, Demosaic};

let dev = develop(&bytes, &DevelopOptions { demosaic: Demosaic::Ahd, ..Default::default() })?;
// dev.rgb: interleaved 16-bit RGB in ProPhoto RGB (ROMM primaries, D50, gamma 1.8)
// dev.warnings: anything approximated or not applied
```

`photocraft-io` はこのクレートを使用しており、RAW ファイルを開くと、組み込みの ProPhoto 互換プロファイルがタグ付けされた
通常の 16 ビット RGB ドキュメントになります。

## 出典（クリーンルーム）

公開仕様、論文、ファイルの観察のみに基づいて実装しています：

* TIFF 6.0、TIFF/EP（ISO 12234-2）、および Adobe DNG Specification 1.7。
* ITU-T T.81（ISO 10918-1）Annex H：ロスレス JPEG、プロセス 14（"LJ92"）。
* Canon の CR2 コンテナについて公開されている説明（ヘッダー、RAW IFD、スライスタグ 0xC640）。
* 公開文書化されているメーカーノート／プライベートタグ（ExifTool のタグ表）：Canon SensorInfo
  （0x00E0）と ColorData（0x4001）、Nikon WB_RBLevels（0x000C）と BlackLevel（0x003D）、Sony
  BlackLevel（0x7310）、WB_RGGBLevels（0x7313）、SonyRawFileType（0x7000）と SonyToneCurve
  （0x7010）、PanasonicRaw の IFD0 タグ、Olympus ImageProcessing（0x2040）と CameraSettings
  （0x2020）のプレビュータグ。
* Sony cRAW（ARW 2）：H. Dietz, "Sony ARW2 Compression: Artifacts And Credible Repair"
  （Electronic Imaging 2016）、および 11 + 7 ビット方式についての RawDigger / diglloyd の解説記事。
  正確なビットレイアウトとトーンカーブのスケールは、サンプルファイルの観察によって確定しました。
* Panasonic RW2 RawFormat 5 と非圧縮の Olympus ORF：サンプルファイルの観察によって確定しました
  （ビットパッキング、ページレイアウト、サンプルの寄せ方）。
* デモザイク：Malvar, He & Cutler（ICASSP 2004）、Hirakawa & Parks, "Adaptive
  homogeneity-directed demosaicing"（IEEE TIP 2005）。
* McCamy の CCT 近似（1992）、Bradford 色順応変換。

dcraw、LibRaw、rawspeed、rawler、rawloader、darktable のコードは一切読んでおらず使用もしておらず、
カメラのカラーテーブルもコピーしていません。

## 対応マトリクス

| 形式 | 状態 |
|---|---|
| DNG | 非圧縮（8–16 ビット、パックあり・なし）およびロスレス JPEG。ストリップとタイル。CFA（Bayer）と LinearRaw。LinearizationTable、BlackLevel（+ repeat、DeltaH/V）、WhiteLevel、ActiveArea、DefaultCrop、ColorMatrix1/2、CameraCalibration、ForwardMatrix、AnalogBalance、AsShotNeutral / AsShotWhiteXY、BaselineExposure、Orientation、OpcodeList2 GainMap（レンズシェーディング） |
| DNG（非可逆 JPEG、JPEG XL、浮動小数点。GainMap 以外のオペコード） | 非対応／適用しない（報告あり） |
| CR2 | スライス付きロスレス JPEG、境界、メーカーノートからの撮影時ホワイトバランス、マスクされた境界で測定した黒レベル |
| CR2 sRAW / mRAW | 非対応 |
| NEF / NRW、ARW、PEF およびその他の TIFF/EP RAW | 非圧縮およびロスレス JPEG（Sony のロスレス ARW を含む）の CFA データ |
| Sony 圧縮 ARW（"cRAW"、SonyRawFileType 2） | デコード対応：11 ビットの最小／最大 + 7 ビット差分のブロック、SonyToneCurve で 14 ビットへ |
| Panasonic / Leica RW2、RawFormat 5（12 ビットおよび 14 ビットのパック） | デコード対応。PanasonicRaw の黒／白／WB／センサー境界を使用 |
| Olympus ORF、非圧縮 16 ビット（E-1、E-400…） | デコード対応。ImageProcessing の黒／WB／ValidBits／クロップを使用 |
| Nikon 圧縮 NEF（ロスレスおよび非可逆）、Sony "Compressed RAW 2"、Pentax 圧縮 PEF、RW2 RawFormat 4 以前、Olympus 圧縮 ORF | 非対応：GPL のデコーダーソース以外にこれらの符号化についての公開された説明が見つからず、このクレートはそれを使用できない（クリーンルーム）。`photocraft-io` は代わりに埋め込み JPEG プレビューを開く |
| CR3、RAF | 認識はするが非対応（プレビューが見つかればプレビューにフォールバック） |
| X-Trans およびその他の非 Bayer CFA | 非対応 |

## 現像パイプライン

1. リニアライゼーションテーブル、位置ごとの黒レベル、白レベルへのスケーリング。
2. ホワイトバランス（撮影時の設定、ファイルにない場合はグレーワールド、または明示的な乗数）を、
   最小の乗数が 1 になるよう正規化し、その後 1 でクリップして白飛びしたハイライトが白のままになるようにします。
3. デモザイク：`Bilinear`、`Mhc`（Malvar–He–Cutler）、または `Ahd`（デフォルト）。
4. DNG 仕様に従ったカメラ → XYZ（D50）（ColorMatrix を白の相関色温度で補間、または ForwardMatrix）、
   → リニア ProPhoto。露出（BaselineExposure + ユーザー EV）、ガンマ 1.8、16 ビット。
5. 向き。

カラーキャリブレーションを持たないファイル（CR2、NEF、ARW、RW2、ORF…）は、文書化された中立的なフォールバックを使います：
ホワイトバランス済みのカメラチャンネルをリニア sRGB の原色として扱います（色はもっともらしいものの、
キャリブレーションされたプロファイルより彩度が低くなります。DNG に変換すればキャリブレーションされた色が得られます）。トーン
カーブは適用しません：結果はシーン参照のレンダリングであり、カメラの JPEG よりフラットになります。

## ツール

`cargo run --release -p photocraft-raw --example rawinfo -- [--dump] [--demosaic ahd] [--png DIR] FILE...`
は、デコードされた内容を表示し、デコードと現像の時間を計測し、sRGB の PNG プレビューと
埋め込み JPEG プレビューを書き出すことができます。
