# photocraft-psd

Adobe Photoshop の **PSD**（バージョン 1）および **PSB**（バージョン 2、「ラージドキュメント形式」）ファイルを読み書きする独立したリーダー／ライターです。読み込んだものはすべて保持するため、変更していないファイルはバイト単位で同一に書き戻されます。他の photocraft クレートには依存しません。

* Adobe が公開している *Photoshop File Formats Specification* に基づくクリーンルーム実装です。仕様に記載がない部分は、MIT ライセンスの psd-tools / ag-psd のドキュメントに従った挙動とし、コードのコメントにその旨を記しています。
* `#![forbid(unsafe_code)]`。`wasm32-unknown-unknown` 向けにビルドできます。コア API はバイトスライスを扱い、ファイル用ヘルパーはネイティブターゲットにのみ存在します。
* 不正な入力に対してパースが決してパニックしません。制限：寸法 ≤ 300 000、メモリ確保は残りの入力量と照合して検査、1 回のデコードで生成されるのは最大 `MAX_DECODED_BYTES`（2 GiB）、ディスクリプタのネスト深さ ≤ 64。

## 保証

| 性質 | 方法 |
|---|---|
| 変更していないファイルについて `PsdFile::from_bytes(b)?.to_bytes()? == b` | チャンネルデータは元のエンコーディングを保持します。未知のブロックやリソースは生のまま格納します。パディングが変わり得る箇所（タグ付きブロック、レイヤー情報、セクション末尾）では、正確なパディングを記録します。 |
| 任意のモデル `f` について `from_bytes(to_bytes(f)) == f` | 正規のデフォルト：`padding: None` は「偶数長までゼロ埋め」を意味し、ファイルがそのデフォルトを使っていた場合、パーサーは常に `None` を返します。 |
| 遅延デコード | パーサーはエンコード済みのバイトをコピーするだけです。チャンネルは要求されたときにデコードされます（`ChannelData::decode`、`ImageData::decode`）。 |
| 合成を行わない | `PsdBuilder` は統合画像を呼び出し側から受け取ります。与えられない場合は白いプレースホルダーを書き込み、リソース 1057 に `has_real_merged_data = false` を設定します。 |

## 公開 API（概要）

```rust
// Top level
PsdFile::from_bytes(&[u8]) -> Result<PsdFile>
PsdFile::to_bytes(&self) -> Result<Vec<u8>>
PsdFile::open(path) / save(path)            // native only
PsdFile { header, color_mode_data, resources, layer_info, layer_info_placement,
          global_layer_mask, global_blocks, layer_mask_trailing, image_data }
PsdFile::layers() -> &[LayerRecord]         // file order = bottom-most first
PsdFile::layers_mut() -> &mut Vec<LayerRecord>
PsdFile::layer(i) -> Option<Layer<'_>>;  iter_layers()
PsdFile::layer_tree() -> Vec<LayerNode>     // nested groups, children bottom-to-top
PsdFile::decode_merged() -> Result<Vec<u8>> // planar big-endian samples
PsdFile::composite_rgba8() -> Result<RgbaImage>
PsdFile::resource(id), global_block(key), icc_profile(), resolution(),
        has_real_merged_data(), merged_has_alpha(), validate()

// Header
Header { version: Version::{Psd,Psb}, channels, width, height, depth, color_mode, reserved }
ColorMode::{Bitmap, Grayscale, Indexed, Rgb, Cmyk, Multichannel, Duotone, Lab, Unknown(u16)}

// Layers
LayerInfo { merged_alpha, layers: Vec<LayerRecord>, padding }
LayerRecord { rect, channels: Vec<ChannelData>, blend_mode, opacity, clipping, flags,
              filler, mask: MaskData, blending_ranges, name, blocks, extra_trailing }
LayerRecord::name()            // prefers `luni`
LayerRecord::section_divider(), section_type(), layer_id(), fill_opacity(), is_visible(),
             block(key), block_mut(key), channel(id), channel_rect(id), decode_channel(id, depth, version)
ChannelData { id, compression: Option<Compression>, data }   // encoded bytes kept verbatim
ChannelData::encode(id, compression, decoded, w, h, depth, version)
ChannelData::decode(w, h, depth, version); set_decoded(...)
MaskData::{None, Mask(LayerMask), Raw(Vec<u8>)}
LayerMask { rect, default_color, flags, parameters: Option<MaskParameters>, real: Option<RealMask>, trailing }
BlendMode (27 modes + PassThrough + Unknown([u8;4])); BlendMode::key()/from_key()
Layer<'_>::rgba8() -> Result<RgbaImage>; user_mask() -> Option<Result<GrayImage>>; channel_bytes(id)

// Tagged blocks
TaggedBlock { signature /* 8BIM | 8B64 */, key, data, padding: Option<Vec<u8>> }
TaggedBlock::parsed() -> Option<Result<BlockData>>
BlockData::{UnicodeName, SectionDivider, LayerId, NameSource, BlendClippedAsGroup,
            BlendInteriorElements, Knockout, Protection, SheetColor, FillOpacity, MetadataSetting}
constructors: unicode_name, section_divider, layer_id, name_source, blend_clipped_as_group,
              blend_interior_elements, knockout, protection, sheet_color, fill_opacity
tagged::uses_long_length(version, key)   // PSB 8-byte length keys

// Image resources
ImageResource { signature, id, name, data };  ImageResource::parsed() -> Option<Result<ResourceData>>
ResourceData::{ResolutionInfo, LayerState, LayerGroupInfo, Thumbnail, GlobalAngle, IccProfile,
               GlobalAltitude, VersionInfo, Exif, Xmp}

// Compression
Compression::{Raw, Rle, Zip, ZipPrediction, Unknown(u16)}
compression::{encode_planes, decode_planes, PlaneLayout, packbits::{encode, decode}, predict, unpredict}
pixels::{samples_u16, samples_f32, u16_to_bytes, f32_to_bytes, unpack_bits, plane_to_u8}

// Descriptors
descriptor::{Descriptor, VersionedDescriptor, Value, Id, Class, ReferenceItem, UnicodeString, ObjectArray}
Descriptor::from_bytes / to_bytes;  VersionedDescriptor::from_bytes / to_bytes (version 16)

// Builder
PsdBuilder::new(w, h).color_mode(..).depth(8|16).version(..).compression(..).resolution(dpi).icc_profile(..)
builder.push_layer(LayerSpec) / begin_group(GroupSpec) / end_group()? / composite(PixelData)
builder.build() -> Result<PsdFile>;  builder.to_bytes()
PixelData::{Rgba8, Rgba16, GrayA8, GrayA16, Cmyka8, Cmyka16}   // CMYK given as ink, stored inverted

// Test generator (feature `testgen`)
testgen::{all_cases, merged_only, layered, small, sample_descriptor, pattern_plane}
```

## 機能マトリクス

| 領域 | 読み込み | 書き出し | 型付き | 備考 |
|---|:-:|:-:|:-:|---|
| ヘッダー、PSD v1 / PSB v2 | ✓ | ✓ | ✓ | 深度 1/8/16/32、全 8 種のカラーモードと `Unknown` |
| カラーモードデータ | ✓ | ✓ | 生 | インデックスカラーのパレットは `composite_rgba8` で使用 |
| 画像リソース | ✓ | ✓ | 一部 | 1005、1024、1026、1033/1036（生）、1037、1039、1049、1057、1058、1060 |
| レイヤーレコード | ✓ | ✓ | ✓ | フラグ、クリッピング、フィラー、ブレンド範囲（生 + アクセサ） |
| レイヤーマスク | ✓ | ✓ | ✓ | 20 / 36 バイト形式、マスクパラメータ、リアルマスク。パースできないマスクは生のまま保持 |
| チャンネル id -1 / -2 / -3 | ✓ | ✓ | ✓ | `channel_rect` によるチャンネルごとの矩形 |
| Raw / RLE / ZIP / ZIP+予測 | ✓ | ✓ | – | 8/16/32 ビットでの予測（32：バイトプレーンのシャッフル + 差分）。RLE のカウントは PSD では u16、PSB では u32 |
| 統合画像データ | ✓ | ✓ | – | パース時に検証（サイズ、RLE カウント、チェックサムを含む zlib ストリーム） |
| タグ付きブロック（生のまま通過） | ✓ | ✓ | – | `8BIM` / `8B64` と正確なパディングを保持 |
| PSB の 8 バイト長キー | ✓ | ✓ | – | LMsk Lr16 Lr32 Layr Mt16 Mt32 Mtrn Alph FMsk lnk2 FEid FXid PxSD |
| Lr16 / Lr32 / Layr レイヤー情報 | ✓ | ✓ | ✓ | `layer_info` に取り込み、元の位置に再挿入 |
| luni lsct lsdk lyid lnsr clbl infx knko lspf lclr iOpa shmd | ✓ | ✓ | ✓ | `TaggedBlock::parsed()` |
| ActionDescriptor（全 OSType） | ✓ | ✓ | ✓ | 参照、`ObAr`（ベストエフォート）、`UnFl`、`Pth `、バージョン 16 ラッパーを含む |
| レイヤーツリー（グループ） | ✓ | ✓ | ✓ | 開閉フォルダーと境界の区切り。不正なネストにも寛容 |
| RGBA8 の抽出 | ✓ | – | – | RGB / グレー / CMYK レイヤー。統合画像はインデックス / ビットマップ / ダブルトーンにも対応 |
| ビルダー | – | ✓ | – | 8/16 ビットの RGB / グレー / CMYK、マスク、グループ、描画モード、不透明度、塗り、表示状態、クリッピング |

### 生のまま保持、まだ型付けしていないもの（パーサーは M8 で予定）

* レイヤー効果：`lfx2`、`lrFX`、`lmfx`。内部のディスクリプタは `VersionedDescriptor::from_bytes(&block.data[4..])` で必要に応じてパースできます。
* テキストレイヤー：`TySh`、`tySh`、およびその内部の EngineData。
* スマートオブジェクト：`SoLd`、`PlLd`、`SoLE`、`lnk2`、`lnkD`、`lnk3`。
* ベクトルマスクとシェイプ：`vmsk`、`vsms`、`vogk`、`vscg`。
* 調整レイヤーと塗りつぶしレイヤー：`curv`、`levl`、`hue2`、`SoCo`、`GdFl`、`PtFl` など、さらに `Patt`、`Txt2`、`FMsk`、3D およびビデオのブロック。
* グローバルレイヤーマスク情報（生、アクセサあり）、サムネイル（生）、スライス、ガイド、パス、その他すべてのリソース id。

## テスト

`cargo test -p photocraft-psd --all-features` で、ユニットテスト、統合テスト、proptest を実行します：

* 生成されたすべてのケース（全深度、全カラーモード、全圧縮方式、PSD と PSB）について、バイトの安定性とモデルのラウンドトリップ。
* PackBits のエッジケース。
* 8/16/32 ビット予測の既知ベクトル。
* PSD と PSB での長さの扱い。
* 切り詰めスイープ：いくつかの小さなファイルについて、あらゆるプレフィックスが `Err` を返すこと。
* ランダム変異とランダムバイトの proptest：決してパニックしないこと。
* レイヤーツリーのケース。
* ビルダー → `rgba8` の正しさ。
* 未知の描画モードと未知のブロックが変更されずに通過すること。

`tests/corpus.rs` は、ワークスペースのルートに `corpus/psd/**/*.{psd,psb}` ディレクトリが存在する場合にそれを走査します。各ファイルについて、パースが成功し、ファイルがバイト単位でラウンドトリップすることをアサートし、ファイルごとの結果を出力します。ディレクトリがない場合は何も出力せずスキップします。

`fuzz/` に `cargo-fuzz` の雛形があります。これは独立したワークスペースで、photocraft のワークスペースからは除外されています。`crates/psd` から `cargo +nightly fuzz run parse`（または `descriptor`）で実行します。

## 仕様の曖昧な点と判断

* **タグ付きブロックのパディング。** 仕様では長さは「偶数バイトに切り上げる」とされています。実際には、ライターはパディングを長さに含めるか、データの後ろに付加するかのどちらかであり、2 または 4 にパディングします。パーサーは各ブロックの後に実際に続くパディング、すなわち次のシグネチャまたは領域の終端が現れる最小の k ≤ 3 を検出します。そのパディングがデフォルトと一致しない限り、それを格納します。
* **レイヤー情報のパディング**（2 か 4 か）も同じ方法で記録します。
* **16/32 ビットのレイヤー** は、メインのレイヤー情報が空の場合、グローバルな `Lr16` / `Lr32` ブロックから読み込みます。ブロックのデータは、内側の長さフィールドを持たないレイヤー情報の本体です。
* **マスクレコードのレイアウト。** 順序は、矩形、デフォルト色、フラグ、パラメータ（フラグのビット 4 が立っている場合）、そして残りが 18 バイト以上ある場合はリアルフラグ、背景、矩形です。残ったもの（例えば 20 バイト形式の 2 バイトのパディング）は `trailing` に保持します。
* **マスクチャンネルの矩形。** チャンネル -2 はマスクの矩形を使います。チャンネル -3 はリアルマスクの矩形を使います。
* **統合画像の ZIP データ** は、全プレーンにまたがる単一の zlib ストリームです。予測は全プレーンにまたがって行単位で適用されます。
* **深度 1 での ZIP 予測** はバイト単位の差分として扱います。
* **`ObAr`** は、ag-psd の読み方に従い、u32 のプレフィックスに続くディスクリプタ形式の本体として読みます。仕様には記載がありません。
* **ディスクリプタの `bool`** で 0/1 以外のバイトは、書き出し時に 1 に正規化されます。
* **Pascal 名** のパッドバイトはゼロとして書き出されます。レイヤー情報の長さがゼロであることだけを含むレイヤー＆マスクセクションは、空のセクションに正規化されます。これらがバイト単位で一致しない既知の唯一のケースであり、どちらも病的なケースです。
* **ヘッダーの制限。** パーサーはどちらのバージョンでも最大 300 000 × 300 000 を受け付けます。`PsdFile::validate()` は PSD について仕様の 30 000 という制限を適用します。
