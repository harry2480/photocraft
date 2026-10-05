# photocraft-cms

Photocraft 向けの Pure Rust 製 ICC カラーマネジメントです。ワークスペース依存はなく（L0、公開可能）、
C コードも `unsafe` も使いません。公開仕様である ICC.1:2010 (v4.3) および ICC.1:2001-04 (v2)、
CIE 15、Adobe が公開している黒点補正の論文に基づいて実装しています。

## API

```rust
use photocraft_cms::{Builtin, Intent, Profile, Transform};

let src = Profile::parse(&icc_bytes)?;                    // v2/v4: matrix/TRC, gray TRC, mft1/mft2/mAB/mBA
let dst = Builtin::CoatedCmyk.profile();
let t = Transform::new(&src, dst, Intent::RelativeColorimetric, /*bpc*/ true)?;
t.convert_u8(&rgba8, 4, &mut cmyka8, 5, /*copy alpha*/ true);   // also convert_u16 / convert_f32 / apply (in place)
let mut out = [0.0; 4];
t.eval(&[1.0, 0.0, 0.0], &mut out);                        // exact single colour
```

* `Transform::proof(src, proof, display, intent, bpc, simulate_paper)`: ソフトプルーフのチェーン。
* `GamutCheck::new(src, proof, ΔE)`: 色域外判定（色域外警告）。
* `Lut3d::from_transform(&t, 33).to_rgba16f_bytes()`: 3D テクスチャとしての表示用変換。
* `transform::cached(src, dst, opts)`: プロセス全体で共有される変換キャッシュ。
* `Profile::to_bytes()`: パースしたプロファイルの元のバイト列（バイト単位で同一）、または ICC v4.3 エンコード。

### 評価

* 厳密パス: 浮動小数点のステージパイプライン（カーブ、行列、CLUT、Lab↔XYZ、PCS エンコード、BPC /
  絶対値スケーリング）。浮動小数点バッファではデフォルトでこれを使います（`TransformOptions::precise_float`）。
* 整数バッファ: matrix/TRC チェーンは、コード値ごとの入力テーブル + 行列 + `x^(1/4)` でインデックスする
  出力テーブルを使います（純粋なガンマカーブでも黒付近で高精度）。CLUT を含むチェーンはデバイスリンクに
  サンプリングし（3 入力は 33³、4 入力は 17⁴、1 入力は 4096）、四面体補間で評価します（4D: 先頭入力について
  2 つの四面体間を線形補間）。純粋に解析的なチェーン（例: Lab ↔ RGB）は厳密に評価します。16k ピクセル以上の
  バッファは rayon で並列実行します（ネイティブのみ）。
* レンダリングインテント: インテントに応じて AToB0/1/2 と BToA0/1/2 を使い、AToB0/BToA0 にフォールバック
  します。絶対的な色域維持はメディア白色点（`wtpt`。v2 のディスプレイプロファイルは D50 として扱う）で
  スケーリングします。BPC はニュートラルな黒点（Y のみ）を XYZ 上で線形にマッピングします。

## 組み込みプロファイル（すべて CC0-1.0、本クレートが生成）

| id | 説明 |
|---|---|
| `srgb` | sRGB IEC61966-2.1（Rec. 709 原色、D65 を Bradford 変換で順応、sRGB TRC） |
| `display-p3` | Display P3 |
| `adobe-rgb-compat` | Adobe RGB (1998) 互換（公開されている原色、ガンマ 563/256） |
| `prophoto-compat` | ProPhoto 互換（ROMM 原色、D50、ガンマ 1.8） |
| `linear-srgb` | sRGB 原色、リニア |
| `rec2020` | Rec. 2020 原色、Rec. 709 OETF |
| `gray-gamma-2.2`, `sgray` | グレー ガンマ 2.2、sRGB カーブのグレー（デフォルトのグレー作業用スペース） |
| `lab-d50` | Lab 恒等変換（v4 エンコード） |
| `coated-cmyk` | **Photocraft Coated CMYK（合成、TAC 300%、GCR 中）** — デフォルトの CMYK |

### CMYK プロファイル

Adobe の CMYK プロファイル（U.S. Web Coated SWOP、Coated FOGRA39）はプロプライエタリであり、
自由にダウンロードできる特性評価ベースのプロファイル（ECI/FOGRA、colord、Ghostscript）には再配布条件が
付いているかライセンスが不明確です。そのため Photocraft は独自のプロファイルを生成します（`src/synth.rs`）。
16 種の重ね刷りに対する Yule–Nielsen 修正 Neugebauer モデル（n = 2）を用い、紙/C/M/Y/RGB の重ね刷りには
ISO 12647-2 コート紙の Lab 目標値、放物線型のドットゲイン（50% で CMY 14 %、K 17 %）、GCR による墨版生成
（グレー成分 20 % から開始、K 最大 95 %）、総インキ量上限 300 %、L* 黒点スケーリングと、サンプリングした
色域境界に対するソフトな彩度圧縮を備えた知覚的テーブルを使います。`profiles/photocraft-coated-cmyk.icc`
（237 KB、AToB 11⁴ lut16、BToA 21³ lut16）がその出力で、パブリックドメイン（CC0-1.0）として提供されます。
再生成するには次を実行します。

```sh
PHOTOCRAFT_REGEN_PROFILES=1 cargo test -p photocraft-cms --release --test regen
```

これはそれらしいコート紙オフセット印刷用プロファイルであり、実測による特性評価ではありません。
本番の印刷にはプリンターのプロファイルを使ってください。

## 未実装 / 近似

* v4.4 の `mpet`（multiProcessElements、D2Bx/B2Dx）タグ、名前付きカラー、変換の端点としてのデバイスリンク
  プロファイルおよび抽象プロファイル、v2 の絶対的な色域維持における `chad` ベースの色順応。
* BPC の出力先の黒は BToA/AToB の往復変換を直接使います（挙動の悪い LUT プロファイル向けの Adobe の
  カーブフィッティングは未実装）。v2/v4 の知覚的参照黒の混合は、v4 LUT プロファイルの PRMG 黒以上には
  モデル化していません。
* 書き出し時にプロファイル ID (MD5) は計算しません（ゼロのまま。仕様上許容されています）。
