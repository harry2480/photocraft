<p align="center">
  <a href="https://getartcraft.com/">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="docs/brand/artcraft-logo-white.svg">
      <img alt="ArtCraft" src="docs/brand/artcraft-logo.svg" width="200">
    </picture>
  </a>
</p>

<h1 align="center">PhotoCraft</h1>

<p align="center">
  <b>画像編集。Adobe Photoshop をクリーンルームで再実装し、純粋な Rust で作り直したオープンソースソフトウェアです。</b><br>
  レイヤー、マスク、調整レイヤー、レイヤースタイル、テキスト、ベクター、ブラシ、そして本物の PSD ファイル。<br>
  すべてを Rust だけで書いたネイティブアプリで。オープンソースで、オフラインで動き、あなたのものです。
</p>

<p align="center">
  <img alt="100% Rust" src="https://img.shields.io/badge/100%25-Rust-b7410e?style=flat-square&logo=rust">
  <img alt="macOS · Windows · Linux · Web" src="https://img.shields.io/badge/macOS%20%C2%B7%20Windows%20%C2%B7%20Linux%20%C2%B7%20Web-native-2f7bf5?style=flat-square">
  <img alt="License: MIT OR Apache-2.0" src="https://img.shields.io/badge/license-MIT%20%2F%20Apache--2.0-3a3a3a?style=flat-square">
  <img alt="Status: early alpha" src="https://img.shields.io/badge/status-early%20alpha-d69e2e?style=flat-square">
</p>

<p align="center">
  <a href="https://discord.gg/artcraft"><img alt="Discord で ArtCraft コミュニティに参加" src="https://img.shields.io/badge/Join%20us%20on%20Discord-5865F2?style=for-the-badge&logo=discord&logoColor=white" height="40"></a>
</p>

<p align="center">
  <a href="https://getartcraft.com/apps/photocraft"><b>getartcraft.com の PhotoCraft ページ</b></a> ·
  <a href="https://getartcraft.com/">ArtCraft</a> ·
  <a href="https://getartcraft.com/apps">Crafting Apps 一覧</a>
</p>

<br>

<p align="center">
  <img src="docs/images/photocraft-demo.jpg" alt="葛飾北斎『神奈川沖浪裏』を編集中の PhotoCraft。ドロップシャドウ付きのキャプションカード、Title と Credit のテキストレイヤー、自然な彩度とトーンカーブの調整レイヤー、画像のヒストグラムに重ねて描かれたトーンカーブエディター" width="100%">
  <br>
  <sub>ドロップシャドウ付きのキャプションカード、編集可能なテキスト、自然な彩度とトーンカーブの調整レイヤー。トーンカーブエディターを開いた状態。<br>
  『神奈川沖浪裏』葛飾北斎、1831 年頃</sub>
</p>

> [!NOTE]
> **ArtCraft は、あらゆる分野のアーティストが集うコミュニティです。** デジタル、ジェネレーティブ、音楽、
> ゲーム &mdash; 何かをつくる人なら、あなたも仲間です。**[Discord で気軽に声をかけてください](https://discord.gg/artcraft)。**

<p align="center">
  <a href="#features">機能</a> ·
  <a href="#everything-in-the-box">同梱されているもの</a> ·
  <a href="#psd-without-compromise">PSD</a> ·
  <a href="#built-for-agents">エージェント</a> ·
  <a href="#under-the-hood">内部構造</a> ·
  <a href="#get-started">はじめに</a> ·
  <a href="#the-crafting-apps">Crafting Apps</a> ·
  <a href="https://discord.gg/artcraft">Discord</a>
</p>

<br>

<table>
  <tr>
    <td width="25%" valign="top">
      <h3>🎛️ 手に馴染む設計</h3>
      メニュー、ショートカット、パネル、ツールは、⌘J から ⇧⌘D まで、手が覚えている場所にあります。Photoshop を知っていれば、もう PhotoCraft も使えます。
    </td>
    <td width="25%" valign="top">
      <h3>⚡ ネイティブで高速</h3>
      wgpu（Metal、Vulkan、DX12、WebGPU）上の GPU コンポジター、コピーオンライトのタイル、マルチスレッドのフィルター。Electron もウェブビューも、待ち時間もありません。
    </td>
    <td width="25%" valign="top">
      <h3>🗂️ 本物の PSD ファイル</h3>
      レイヤー付きの Photoshop ドキュメントを開き、編集し、保存できます。実際の PSD 135 個のうち 134 個がバイト単位で完全にラウンドトリップします。
    </td>
    <td width="25%" valign="top">
      <h3>🤖 エージェント対応</h3>
      すべての操作がコマンドなので、同じエンジンを UI、CLI、JSON コントロールチャネル、MCP サーバーのどれからでも操作できます。
    </td>
  </tr>
</table>

<br>

<a id="features"></a>
## 機能

ここにあるスクリーンショットはすべて、パブリックドメインの作品を実際のアプリで編集し、コントロールチャネル経由でオフスクリーン描画したものです。

<table>
  <tr>
    <td width="50%" valign="top">
      <img src="docs/images/photocraft-adjustments.jpg" alt="モネ『印象・日の出』にレベル補正と自然な彩度の調整レイヤー。右側にレベル補正エディターと、平均・標準偏差・中央値を表示するヒストグラムパネルが開いている" width="100%">
      <br>
      <sub>レベル補正と自然な彩度の調整レイヤー、ライブ更新されるヒストグラムパネル。<br>『印象・日の出』クロード・モネ、1872 年</sub>
      <h3>後悔しない編集</h3>
      調整レイヤーはすべての編集をライブのまま保ちます。レベル補正、トーンカーブ、自然な彩度、色相・彩度など十数種類を重ね、範囲をマスクし、並べ替え、オフにしても、元のピクセルは一切変わりません。
      <br><br>
      ピクセルに直接適用することもできる <b>16 種類の調整レイヤー</b>。チャンネルごとに編集できるトーンカーブ、ライブヒストグラム付きのレベル補正、白黒、チャンネルミキサー、グラデーションマップ、レンズフィルター、特定色域の選択、カラールックアップ（.cube、.3dl、.look）などを備えます。さらにシャドウ・ハイライト、色の置き換え、カラーの適用、HDR トーン、彩度を下げる、平均化（イコライズ）も使えます。
    </td>
    <td width="50%" valign="top">
      <img src="docs/images/photocraft-layer-styles.jpg" alt="アポロ 8 号の写真『地球の出』の上で、EARTHRISE テキストレイヤーの光彩（外側）をレイヤースタイルダイアログで編集中。境界線も有効になっている" width="100%">
      <br>
      <sub>ライブのテキストレイヤーに光彩（外側）と境界線を、レイヤースタイルダイアログで適用。<br>『地球の出』ウィリアム・アンダース / NASA、1968 年</sub>
      <h3>一枚を引き立てるスタイル</h3>
      ドロップシャドウ、シャドウ（内側）、光彩（外側・内側）、ベベルとエンボス、サテン、境界線、カラー・グラデーション・パターンオーバーレイを、テキストを含むあらゆるレイヤーにライブで適用できます。パターンはライブラリ（組み込み、編集 › パターンを定義、<code>.pat</code> の読み込み・書き出し）と PSD の <code>Patt</code> ブロックから取得します。
      <br><br>
      レイヤー間でスタイルをコピー＆ペーストしたり、すべての効果を一括で非表示にしたり、PSD のスタイルをそのまま開いて Photoshop と一致するように描画したりできます。
    </td>
  </tr>
  <tr>
    <td width="50%" valign="top">
      <img src="docs/images/photocraft-masks.jpg" alt="フェルメール『真珠の耳飾りの少女』。顔の周りの楕円選択範囲と、楕円マスク付きの色相・彩度レイヤーにより顔以外がすべてグレーになっている" width="100%">
      <br>
      <sub>楕円選択範囲を色相・彩度レイヤーのマスクにし、顔だけに色を残す。<br>『真珠の耳飾りの少女』ヨハネス・フェルメール、1665 年頃</sub>
      <h3>画像を理解する選択範囲</h3>
      精密な作業には長方形・楕円形選択、なげなわ、自動選択。コンピューターになぞらせたいときはクイック選択、オブジェクト選択、被写体を選択。髪の毛のような細い境界は選択とマスクで調整できます。
      <br><br>
      ぼかし、拡張、縮小、滑らかに、選択範囲を拡張、再選択。どの選択範囲もレイヤーマスク、ベクターパス、シェイプに変換できます。スマートな選択はあなたのマシン上で動作し、クラウドもアカウントも不要です。
    </td>
    <td width="50%" valign="top">
      <img src="docs/images/photocraft-type.jpg" alt="ビアスタット『カリフォルニア、シエラネバダの山中』の上で、見出し SIERRA NEVADA を Georgia でキャンバス上編集中。署名行と段落キャプションがあり、文字パネルと段落パネルが開いている" width="100%">
      <br>
      <sub>その場で編集する見出しと、署名行、本文の段落。<br>『カリフォルニア、シエラネバダの山中』アルバート・ビアスタット、1868 年</sub>
      <h3>美しく組める文字</h3>
      ポイントテキストと段落テキストをキャンバス上で直接編集でき、フォント、ウェイト、サイズ、行送り、トラッキング、揃え、カラーまで文字・段落の設定を一通り備えています。
      <br><br>
      テキストレイヤーは編集可能なまま保たれ、レイヤースタイルを適用でき、PSD を介してラウンドトリップします。
    </td>
  </tr>
  <tr>
    <td width="50%" valign="top">
      <img src="docs/images/photocraft-vector.jpg" alt="モネ『睡蓮』の上に、シェイプレイヤー（Lotus、Water、Sun、Badge、Dotted Ring）で作った蓮のバッジ。蓮のパスのアンカーポイントが選択されている" width="100%">
      <br>
      <sub>シェイプレイヤーで作ったバッジ。グラデーションで塗った蓮、星、点線のリング。<br>『睡蓮』クロード・モネ、1906 年</sub>
      <h3>ピクセル単位で正確なベクター</h3>
      長方形、楕円形、三角形、多角形、ラインの各ツールとペンツール。解像度に依存しないシェイプレイヤー、グラデーション塗り、破線や位置揃えに対応した線を使えます。
      <br><br>
      シェイプを結合（合体、前面シェイプを削除、交差、中マド）し、パスをパスパネルに保持し、ベクターマスクとして使い、線や塗りを適用できます。PSD コーパス内の 116 個のシェイプレイヤーすべてが Photoshop のピクセルと一致します。
    </td>
    <td width="50%" valign="top">
      <img src="docs/images/photocraft-filters.jpg" alt="ゴッホ『星月夜』で、角度 320 度のうずまきフィルターダイアログが楕円選択範囲の内側にライブプレビューされている" width="100%">
      <br>
      <sub>うずまきが選択範囲の内側だけにキャンバス上でライブプレビューされる。<br>『星月夜』フィンセント・ファン・ゴッホ、1889 年</sub>
      <h3>確定する前に確認できる</h3>
      すべてのフィルターダイアログは、選択範囲を通してキャンバス上でライブプレビューします。ぼかし（ガウス、ボックス、移動、放射状、表面、詳細、レンズ、シェイプ、そしてぼかしギャラリーのチルトシフト、虹彩絞り、フィールド、スピン、パス）、シャープ、ノイズを軽減、変形（うずまき、波形、つまむ、置き換え、シアー、ジグザグ…）、ピクセレート、表現手法（油彩、風、押し出し…）、描画（雲模様、ファイバー、逆光、照明効果）などがあります。
      <br><br>
      スマートオブジェクトにフィルターをかけると編集可能なまま残り、いつでも変更、非表示、並べ替え、マスクができます。
      <br><br>
      大きな半径のぼかしは全コアを使った累積和ボックスパスで処理します。3.6 MP の画像で半径 180 のガウスぼかしが 1 秒未満で終わります。
    </td>
  </tr>
  <tr>
    <td width="50%" valign="top">
      <img src="docs/images/photocraft-transform.jpg" alt="アンセル・アダムス『ティトン山脈とスネーク川』の回転したコピーを囲む自由変形ハンドル。ヒストリーパネルには開く、レイヤーを複製、自由変形などのステップが並んでいる" width="100%">
      <br>
      <sub>回転したプリントへの自由変形。すべてのステップがヒストリーに記録される。<br>『ティトン山脈とスネーク川』アンセル・アダムス、1942 年</sub>
      <h3>思いどおりの形に</h3>
      拡大・縮小、回転、ゆがみ、自由な形に、遠近法を備えた自由変形。正確な 90°・180° の回転と反転、再変形。レイヤー、テキスト、シェイプ、マスク、選択範囲がすべて一緒に変形します。
      <br><br>
      完全なヒストリー、最後の状態を切り替え、ヒストリーブラシにより、ブラシの一筆単位まで、あらゆるステップを取り消せます。
    </td>
    <td width="50%" valign="top">
      <img src="docs/images/photocraft-export-light.jpg" alt="クリムト『接吻』の上に、ライトテーマの書き出し形式ダイアログ。JPG、画質 90、50 パーセントに縮小、プレビューと推定サイズ約 684K を表示" width="100%">
      <br>
      <sub>ライトテーマの書き出し形式。プレビューとファイルサイズの見積もり付き。<br>『接吻』グスタフ・クリムト、1907–1908 年</sub>
      <h3>どこへでも書き出せる</h3>
      書き出し形式では、フォーマット、画質、透明部分、拡大・縮小を指定でき、プレビューとファイルサイズの即時見積もりも表示します。クイック書き出しなら PNG へワンクリックです。
      <br><br>
      ダークな Pro テーマ、軽やかな Studio テーマ、Classic な外観から選べます。
    </td>
  </tr>
</table>

<br>

<a id="everything-in-the-box"></a>
## 同梱されているもの

<table>
  <tr>
    <td width="33%" valign="top">
      <h4>🧰 34 のツール</h4>
      移動 · 長方形選択・楕円形選択 · なげなわ · 多角形選択 · 自動選択 · クイック選択 · オブジェクト選択 · 切り抜き · スポイト · ブラシ · 鉛筆 · 混合ブラシ · 色の置き換え · 消しゴム · コピースタンプ · 修復ブラシ · スポット修復ブラシ · ヒストリーブラシ · グラデーション · 塗りつぶし · ぼかし · シャープ · 指先 · 覆い焼き · 焼き込み · スポンジ · ペン · パスコンポーネント選択 · 文字 · 5 種のシェイプツール · 手のひら · ズーム
    </td>
    <td width="33%" valign="top">
      <h4>🖌️ 本格的なブラシエンジン</h4>
      シェイプ、散布、テクスチャ、デュアルブラシ、カラー、伝達、ブラシポーズ、ウェットエッジ、エアブラシ効果、滑らかさ（ひも付き補正を含む）を、筆圧、傾き、回転、方向で制御します。ブラシプリセット、選択範囲からのブラシ定義、決定論的で再生可能なストロークにも対応します。
    </td>
    <td width="33%" valign="top">
      <h4>🗃️ きちんと作られたレイヤー</h4>
      グループ、クリッピングマスク、ピクセルマスクとベクターマスク、塗りつぶしレイヤー（べた塗り、グラデーション、パターン）、調整レイヤー、スマートフィルターと非破壊の変形・ワープを備えたライブのスマートオブジェクト、整列・分布・リンク付きの複数レイヤー選択、アルファチャンネルとクイックマスク、27 種の描画モード、不透明度と塗り、ロック、カラーラベル、レイヤーフィルター、結合、統合、ラスタライズ、コピーしたレイヤー／カットしたレイヤー、選択範囲内へペースト。
    </td>
  </tr>
  <tr>
    <td width="33%" valign="top">
      <h4>🎨 あらゆるカラー、あらゆるビット深度</h4>
      RGB、グレースケール、CMYK、Lab のドキュメントをチャンネルあたり 8、16、32 ビットで扱えます。ビット深度とカラーモデルは実行時のデータなので、すべてのツールがすべての深度で動作します。
      <br><br>
      純粋な Rust による本格的な ICC カラーマネジメント。埋め込みプロファイル、4 つのレンダリングインテントすべてと黒点の補正に対応したプロファイルの指定・変換、GPU による色の校正（⌘Y）と色域外警告（⇧⌘Y）。
    </td>
    <td width="33%" valign="top">
      <h4>🗂️ フォーマット</h4>
      PSD と PSB に加え、PNG、JPEG、TIFF、WebP、GIF、BMP、TGA、ICO、QOI、PNM、OpenEXR、Radiance HDR、AVIF。8、16、32 ビットで読み書きが対称にでき、ネイティブの <code>.pcraft</code> フォーマットもあります。
    </td>
    <td width="33%" valign="top">
      <h4>🪄 日常に欠かせない機能</h4>
      自動トーン補正・自動コントラスト・自動カラー補正 · 平均化（イコライズ） · 画像解像度とカンバスサイズ · 切り抜きとトリミング · すべての領域を表示 · 編集 › 塗りつぶしと境界線を描く · 結合部分をコピー · 同じ位置にペースト · ガイド、定規、グリッド、スナップ · アクションの記録と再生 · コマンドパレット（⌘K）。
    </td>
  </tr>
</table>

<br>

<a id="psd-without-compromise"></a>
## 妥協のない PSD 対応

PhotoCraft の PSD サポートは、Adobe の公開仕様から書き起こした独立したクレートで、実際のファイルのコーパスに対してテストされています。

- **バイト単位で一致するラウンドトリップ:** コーパスの 135 ファイル中 134 ファイルが完全に同一の内容で書き戻されます。まだモデル化していないもの（生ブロック、ディスクリプター、追加情報）は破棄せず、そのまま保持します。
- **一致するピクセル:** コンポジットオラクルが、こちらの描画結果と Photoshop 自身の結合画像を比較します。グラデーション補間（クラシック、知覚的、リニア）、レイヤー効果、シェイプの線、クリッピング、塗りの不透明度をカバーしています。
- **大きなドキュメント:** PSB、16 ビット・32 ビットのファイル、CMYK や Lab のドキュメントをネイティブに開けます。

<a id="built-for-agents"></a>
## エージェントのための設計

すべてのメニュー項目、ツール、ダイアログは、500 以上のコマンドを収めた 1 つのレジストリのコマンドを実行します。UI、CLI、JSON コントロールチャネル、MCP サーバーはすべて同じコマンドを呼び出すので、クリックでできることはスクリプトや AI エージェントにもできます。

```sh
# Headless: open, edit, save
photocraft-cli run wave.psd \
  --cmd filter.sharpen.smartSharpen     --params '{"amount":80}' \
  --cmd layer.newAdjustmentLayer.curves --params '{"points":[[0,0],[64,48],[192,212],[255,255]]}' \
  --out wave-final.png

# Apply one action list to a folder of images
photocraft-cli batch --actions grade.json --in ./raw --out ./graded

# Let an agent drive it over MCP (headless, or bridged to the running app)
photocraft-cli mcp
```

デスクトップアプリには、認証付きでループバック専用のコントロールチャネル（`photocraft --control`）もあり、UI 状態の検査、ポインターイベントによるツール操作、オフスクリーンのスクリーンショット取得ができます。この README の画像はすべてその方法で描画しました。詳しくは [`docs/control-protocol.md`](docs/制御プロトコル.md) を参照してください。

<a id="under-the-hood"></a>
## 内部構造

- **エンジンファースト:** 純粋なデータとしてのドキュメントモデルとコマンドエンジンの上に、薄い egui UI が載っています。24 のクレートにわたるレイヤー構造はビルド時に強制されます。
- **2 つのコンポジター:** CPU コンポジターが参照用のオラクルとなり、wgpu コンポジターがキャンバスを GPU で描画します。両者は互いに照合してテストされています。
- **コピーオンライトのタイル:** 256² の疎なタイルにより、取り消しは軽く、巨大なキャンバスも軽快です。効果マップはレイヤー状態ごとにキャッシュされます。
- **ブラウザで動作:** エンジンと UI のすべてが WebAssembly にコンパイルされます。
- **クリーンルーム:** 公開仕様と観察した挙動のみから実装しています。プロプライエタリなコード、シェーダー、アセットは一切含みません。
- **テスト済み:** PSD のラウンドトリップ、合成データジェネレーター、コンポジターオラクル、複数ビット深度のチェックなど、1,700 を超えるテストがあります。

<a id="get-started"></a>
## はじめに

```sh
git clone https://github.com/storytold/photocraft
cd photocraft
cargo run --release -p photocraft -- image.psd   # the desktop app
cargo test --workspace                           # the test suite
```

新しいコントリビューターと AI エージェントは、まず [`AGENTS.md`](AGENTS.md)、次に [`docs/`](docs/) を読んでください。

macOS、Windows、Linux、Web 向けのインストーラーは各 [GitHub リリース](https://github.com/storytold/photocraft/releases) に添付されています。Linux では AppImage、`.deb`、`.rpm`、tarball、Flatpak バンドルから選べます。バンドルには [Flathub](https://flathub.org/setup) の freedesktop ランタイムが必要で、`flatpak` が一緒にインストールするよう提案してくれます。

```sh
flatpak install --user photocraft-<version>-linux-x86_64.flatpak   # or -linux-aarch64
flatpak run ai.storyteller.photocraft
```

メンテナー向け: リリースのビルド、署名、公開の方法は [`docs/releasing.md`](docs/リリース.md) で説明しています。

> [!IMPORTANT]
> **ステータス:** PhotoCraft は初期アルファ段階です。現状を率直にお伝えすると、Photoshop の機能の多くは何らかの形で存在していますが、**日常のプロの仕事で Photoshop の代わりになるにはまだ至っていません**。最も大きな不足は、AI／生成系機能、約 20 の未実装ツール、タイポグラフィとプロ向けワークフローの深さ、プラグイン互換性です。Photoshop のメニュー項目はすべてコマンドに接続されていますが（[`docs/parity.md`](docs/parity.md)）、これは接続状況を測るものであり、挙動を測るものではありません。観点ごとの率直な評価と今後の方向性は、[ロードマップのパリティ評価](docs/ロードマップ.md#honest-parity-assessment-2026-10-05) にまとめています。荒削りな部分があることを前提に、ぜひ Issue を登録してください（OS、ドキュメントサイズ、レイヤー数、スクリーンショットを添えてください）。何が壊れたかを [Discord](https://discord.gg/artcraft) で教えていただくこともできます。

## ドキュメント

開発者向け、アーキテクチャ、自動化、フォーマット、セキュリティのドキュメントは [PhotoCraft ドキュメントブック](book/) で管理しています。

## セキュリティ

セキュリティアーキテクチャ、脅威モデリング、パーサーの堅牢化、ファジング、脆弱性の報告については、[セキュリティドキュメント](book/src/security/) とリポジトリの [セキュリティポリシー](セキュリティ.md) で扱っています。

<a id="the-crafting-apps"></a>
## Crafting Apps

PhotoCraft は **Crafting Apps** のひとつです。Crafting Apps は [ArtCraft](https://getartcraft.com/) チームによる無料でオープンソースのクリエイティブツール群で、
それぞれが Rust でゼロから書かれ、単独でも使えます。

| | アプリ | 用途 | コード | 詳細 |
|:-:|---|---|---|---|
| <img src="https://raw.githubusercontent.com/storytold/photocraft/main/assets/app-icon/hicolor/64x64/apps/ai.storyteller.photocraft.png" alt="" width="32" height="32"> | **PhotoCraft** | **画像編集: レイヤー、マスク、テキスト、本物の PSD ファイル · 現在地** | [GitHub](https://github.com/storytold/photocraft) | [Web サイト](https://getartcraft.com/apps/photocraft) |
| <img src="https://raw.githubusercontent.com/storytold/vectorcraft/main/assets/app-icon/hicolor/64x64/apps/ai.storyteller.vectorcraft.png" alt="" width="32" height="32"> | **VectorCraft** | ベクターイラストレーション | [GitHub](https://github.com/storytold/vectorcraft) | [Web サイト](https://getartcraft.com/apps/vectorcraft) |
| <img src="https://raw.githubusercontent.com/storytold/filmcraft/main/assets/app-icon/hicolor/64x64/apps/ai.storyteller.filmcraft.png" alt="" width="32" height="32"> | **FilmCraft** | 動画編集、カラー、サウンド | [GitHub](https://github.com/storytold/filmcraft) | [Web サイト](https://getartcraft.com/apps/filmcraft) |
| <img src="https://raw.githubusercontent.com/storytold/lightcraft/main/assets/app-icon/hicolor/64x64/apps/ai.storyteller.lightcraft.png" alt="" width="32" height="32"> | **LightCraft** | 写真ライブラリと RAW 現像 | [GitHub](https://github.com/storytold/lightcraft) | [Web サイト](https://getartcraft.com/apps/lightcraft) |
| <img src="https://raw.githubusercontent.com/storytold/printcraft/main/assets/app-icon/hicolor/64x64/apps/ai.storyteller.printcraft.png" alt="" width="32" height="32"> | **PrintCraft** | PDF の閲覧、整理、保護 | [GitHub](https://github.com/storytold/printcraft) | [Web サイト](https://getartcraft.com/apps/printcraft) |
| <img src="https://raw.githubusercontent.com/storytold/effectcraft/main/assets/app-icon/hicolor/64x64/apps/ai.storyteller.effectcraft.png" alt="" width="32" height="32"> | **EffectCraft** | モーショングラフィックスと視覚効果 | [GitHub](https://github.com/storytold/effectcraft) | [Web サイト](https://getartcraft.com/apps/effectcraft) |
| <img src="https://raw.githubusercontent.com/storytold/designcraft/main/assets/app-icon/hicolor/64x64/apps/ai.storyteller.designcraft.png" alt="" width="32" height="32"> | **DesignCraft** | ページレイアウトと出版 | [GitHub](https://github.com/storytold/designcraft) | [Web サイト](https://getartcraft.com/apps/designcraft) |

そして [**ArtCraft**](https://getartcraft.com/) 本体。本当のコントロールを求めるアーティストのための AI 画像・動画スタジオです。

Crafting Apps は共通の約束事を持っています。クリーンルームで純粋な Rust、macOS・Windows・Linux でネイティブに動き、WebAssembly でブラウザでも動き、エージェントから完全に操作できます。

<br>

<p align="center">
  <a href="https://discord.gg/artcraft"><img alt="Discord で ArtCraft コミュニティに参加" src="https://img.shields.io/badge/Join%20us%20on%20Discord-5865F2?style=for-the-badge&logo=discord&logoColor=white" height="40"></a>
</p>

<h3 align="center">一緒にものづくりをしましょう</h3>

<p align="center">
  私たちの Discord には、あらゆる種類のアーティストが集まっています。絵を描く人、写真を撮る人、イラストを描く人、映画を編集する人、
  文字を組む人、そして自分が何をつくりたいのかまだ探している人。取り組んでいる作品を共有したり、
  助けを求めたり、壊れているところを教えてくれたり、これらのツールにできてほしいことを伝えてください。
  どんな表現手段でも、どれだけの経験があっても、歓迎します。
</p>

<p align="center">
  <a href="https://discord.gg/artcraft"><b>discord.gg/artcraft</b></a> ·
  <a href="https://getartcraft.com/">getartcraft.com</a> ·
  <a href="https://getartcraft.com/apps">The Crafting Apps</a> ·
  <a href="https://getartcraft.com/apps/photocraft">PhotoCraft</a>
</p>

---

## ライセンスとクレジット

PhotoCraft は [MIT](LICENSE-MIT) または [Apache-2.0](LICENSE-APACHE) のデュアルライセンスで、どちらかを選択できます。
Copyright (c) 2026 ArtCraft Team and the PhotoCraft contributors. 必要な通知は [NOTICE](NOTICE) にあります。

同梱のフォント、アイコン、画像その他のアセットは、それぞれ独自のオープンライセンスを保持しています。各アセットの
作者、入手元、ライセンスは [ATTRIBUTION.md](ATTRIBUTION.md) に記載しています。

掲載している作品はすべてパブリックドメイン（Wikimedia Commons、NASA、米国国立公文書館）です。出典は [`docs/images/SOURCES.md`](docs/images/SOURCES.md) に記載しています。

[`docs/brand/`](docs/brand/) にある ArtCraft の名称、ワードマーク、ロゴは ArtCraft Team の商標であり、
本ライセンスの対象外です。[`docs/brand/LICENSE-brand.txt`](docs/brand/LICENSE-brand.txt) に従い、改変せず、
このリポジトリおよび PhotoCraft の一部としてのみ使用できます。
フォークや改変版ではこれらを削除する必要があります。

<sub>Adobe、Photoshop、Illustrator、Premiere Pro、Lightroom、Acrobat、After Effects、InDesign は、米国およびその他の国における Adobe Inc. の商標または登録商標です。PhotoCraft は独立したオープンソースプロジェクトであり、Adobe Inc. とは提携しておらず、後援や承認も受けていません。これらの名称は、互換性のあるワークフローを説明する目的でのみ使用しています。</sub>

<p align="center">
  <a href="https://getartcraft.com/"><img alt="ArtCraft" src="docs/brand/artcraft-mark.svg" width="28"></a><br>
  <sub><a href="https://getartcraft.com/">ArtCraft</a> チームとコミュニティによって作られています。</sub>
</p>
