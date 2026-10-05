# AGENTS.md: AI エージェントとコントリビューター向けガイド

PhotoCraft は、**Rust のみ**（JavaScript や TypeScript は使わない）で書かれた、オープンソースのネイティブな Photoshop 相当の画像エディタです。**Tauri、Electron、webview シェルは使いません。** デスクトップアプリは wgpu 上のネイティブな egui/eframe であり、Web ビルドは同じ Rust を WebAssembly にコンパイルしたもの（trunk + wasm-bindgen）です。Tauri（およびあらゆる webview/JS UI フレームワーク）を依存関係、ビルドステップ、パッケージングターゲットとして追加してはいけません。ユーザー向けテキスト（UI、ウィンドウタイトル、About、インストーラー、リリース名、ドキュメントの本文）では、製品名は常に **PhotoCraft** と表記します（兄弟アプリの ArtCraft、ArtCraftX、DesignCraft、DrawCraft、EffectCraft、FilmCraft、LightCraft、PrintCraft と同様に、PascalCase の `{Function}Craft`）。機械向けの名前は小文字のままにします：クレート（`photocraft-*`）、バイナリ、ファイル名、ID（`ai.storyteller.photocraft`）。Craft 系アプリ間で共有される標準と知見は `../craftrules` にあります（その `README.md` を読んでください）。再利用可能な知見はそこへ還元し、コードは還元しないでください。リポジトリ間でコードは共有しません。目標は Photoshop との 1:1 のパリティ（同じメニュー、ショートカット、挙動、ファイル忠実度）をより高い性能で実現し、すべての機能をエージェントから操作可能にすることです。まずこのファイルを読み、次に `docs/` を読んでください。

## 1. オリエンテーション（5 分）

| 読むもの | 理由 |
|---|---|
| `docs/architecture.md` | クレート構成、依存レイヤー、エンジンと UI の境界、ドキュメントモデル |
| `docs/development.md` | ビルド、テスト、実行、アプリのプログラム操作、デバッグのコツ |
| `docs/contributing.md` | ルール：クリーンルーム、テスト、レイヤリング、スタイル、コミット、「コマンドを追加する」チェックリスト |
| `docs/control-protocol.md` | JSON コントロールチャネル：エージェントが実行中のアプリを操作しスクリーンショットを撮る方法 |
| `docs/ui-design.md` | デザイントークン、テーマ、ウィジェット、Photoshop の見た目に合わせる方法 |
| `docs/roadmap.md` | 率直なパリティ評価（どこが不足しているか、どこへ向かうか）、マイルストーン、**現在のフォーカス** |
| `docs/parity.md` | 生成された、Photoshop の全メニュー項目の実装済み／未実装一覧 |
| `crates/<name>/README.md`（存在する場合） | そのクレートの公開 API |

## 2. ワークスペース構成

```text
crates/
  geom cms color raster      L0 foundation (geometry, ICC colour management, pixel formats + blend math, COW tiles)
  psd codecs                 L0 standalone format crates (no workspace deps; publishable)
  doc                        L1 document model (layers, masks, adjustments, effects, smart objects: pure data)
  ops paint algo text vector L2 history, brush engine, imaging algorithms, type engine, paths/shapes
  compose gpu format         L3 CPU compositor (the oracle), wgpu compositor, .pcraft native format
  io plugins                 L4 document <-> PSD / flat formats; sandboxed WebAssembly plug-ins
  engine                     L5 Session + command registry (every action is a command)
  ui-egui automation         L6 egui shell (thin: all actions go through the engine); MCP server
  testkit                    test helpers
apps/
  photocraft                 desktop app (eframe/wgpu), TCP control server
  photocraft-cli             headless CLI (convert/info/run/batch/commands/mcp)
  photocraft-web             the same app in the browser (trunk + wasm-bindgen)
xtask/                       cargo xtask layers | wasm | ci | stats | corpus | parity
```

**レイヤリングは `cargo xtask layers` によって強制されます。** クレートは自身より下位のレイヤーにのみ依存できます。`psd`、`codecs`、`cms` はワークスペース内の何にも依存しません。`ui-egui` より下のレイヤーでは egui、eframe、winit、rfd を使ってはいけません。新しいクレートは `xtask/src/layers.rs` に登録する必要があります。

## 3. 黄金律

### 決してクラッシュさせない（機能開発より優先）

ユーザーは PhotoCraft に自分の作品を託しており、クラッシュはその作品を失わせます。不正なファイル、誤ったコマンドや MCP パラメータ、壊れた設定ファイル、変わったキー入力、ディスクフルは、パニックではなく、ユーザーやエージェントが対処できるエラーを生じさせなければなりません。パニック経路を追加して機能を出荷してはいけません。クラッシュは、その上に何かを作る前に修正してください。共通の標準は `../craftrules/standards/never-crash.md` です。

- **テスト以外のコードは決してパニックしない。** `unwrap()`、`expect()`、`panic!`、`unreachable!`、`todo!`、`unimplemented!` は使いません。クレートのエラー型を返し、`?` で伝播させます。`ok_or(..)?`、`let .. else { return Err(..) }`、`if let`、あるいはフォールバックが本当に正しい場合（ドキュメントを黙って破損させるものは不可）は `unwrap_or*` を使います。未完成の機能は "unsupported" エラーを返します。唯一の例外は、証明可能に失敗しないリテラルです：`#[allow(clippy::expect_used)]` に加えて `.expect("why it can't fail")`。
- **`unsafe` 禁止。** ワークスペースで `unsafe_code = "forbid"` を設定しています。
- **入力由来の数値は敵対的とみなす。** ファイル、パラメータ、選択範囲、またはそれらに対する演算から得たインデックスには、`[i]`/`[a..b]` ではなく `get()` を使います。文字列のスライスは文字境界でのみ行います。長さ、オフセット、個数には `checked_*`/`saturating_*` を使います。ゼロ除算や NaN/inf のキャストを防ぎ、入力によってサイズが決まる確保には上限を設けます。
- **再帰に上限を設ける。** 深さ制限や訪問済み集合を使います（ドキュメントは深くなったり循環したりし得ます）。
- **連鎖させない。** ロックのポイズニングを処理し（`lock().unwrap_or_else(PoisonError::into_inner)`）、スレッドの join は `Result` として扱います。
- **最後の砦。** アプリシェルは、コマンドディスパッチとファイルのインポート／エクスポートの周囲で漏れたパニックを捕捉し、エラーとして報告してドキュメントを保持しなければなりません。これはセーフティネットであって、パニックしてよいという許可ではありません。`panic = "unwind"` を維持してください。
- **証明する。** すべてのクラッシュ修正には、修正前にパニックしていた小さな合成回帰テストを付けます。
- **clippy で強制。** `clippy.toml` は `unwrap`/`expect`/`panic`/インデックス参照をテスト内でのみ許可します。クリーンなクレートは `#![deny(clippy::unwrap_used, clippy::expect_used, clippy::panic, clippy::unimplemented, clippy::todo, clippy::unreachable)]` を持ちます。新しいクレートはこれを付けて始めます。

1. **すべてはコマンド。** ユーザーに見える新しい挙動 = エンジン内のコマンド（`crates/engine/src/*_cmds.rs`、`commands.rs` に登録）で、id、label、メニューパス、ショートカット、パラメータのドキュメント、`enabled`、`run` を持ち、テストを伴います。UI、CLI、コントロールチャネル、MCP はいずれも id でコマンドをディスパッチします。**`crates/ui-egui/src/menu_catalog.rs` にある正確な id** を使えば、メニュー項目は自動的に有効になります。純粋なビュー／ウィンドウ状態（ズーム、パネル、スクリーンモード）だけはシェル（`menus.rs` の `UI_COMMANDS`）に属します。
2. **フォーマットやカラーを仮定しない。** ビット深度（8/16/32f）とカラーモデル（RGB/Gray/CMYK/Lab…）は実行時のデータです。公開 API に `u8` 専用のピクセル経路を導入してはいけません。sRGB を仮定してはいけません：色変換は `photocraft-cms`（`Transform`、`transform::cached`）を通します。複数の深度でテストしてください。
3. **クリーンルーム。** Photoshop やその他のプロプライエタリなエディタは、*挙動と見た目についてのみ* 研究しました。それらのコード、シェーダー、プロファイル、アセットを決してコピーしないでください。公開仕様（Adobe PSD 仕様、ICC、ISO 32000 のブレンドモード、論文）と観察から実装します。サードパーティのアセットは寛容なライセンスでなければならず、ライセンスファイルをその隣に置き、同じ変更内で `ATTRIBUTION.md` に行（パス、タイトル、作者、出典、ライセンス）を追加する必要があります。オリジナルのアセットも同様です。`docs/brand/` にある ArtCraft のロゴはオープンソースではありません（`docs/brand/LICENSE-brand.txt`）。
4. **テストが関門。** すべての変更にテストを伴います。フォーマット系クレートはラウンドトリップ、合成ジェネレーター、オラクル、ファズテストを使います。PSD コーパスの結果とパリティの下限（`crates/ui-egui/src/parity.rs`）を後退させないでください。
5. **UI は薄く、データ駆動。** UI の状態は `ui-egui/src/state.rs`（serde）に置き、コントロールチャネルから読み取り・操作できるようにします。色と角丸は `theme::Tokens` から取り、決してハードコードしません。
6. **UI の変更は目で確認する。** `cargo run -p photocraft-ui-egui --example snapshot` でオフスクリーン描画する（ウィンドウなし、フォーカスを奪わない）か、`--control` 付きで起動して `ui.screenshot` を取得します。PNG を確認してください。デモ画像はパブリックドメインの作品でなければならず、個人の写真は決して使いません。アセットを取得する際、リクエスト（User-Agent、ヘッダー、URL）に個人の名前、メールアドレス、その他の個人情報を含めてはいけません。汎用の `Photocraft-dev` User-Agent を使ってください。
7. **wasm を決して壊さない。** L0–L6 は `cargo check --target wasm32-unknown-unknown` が通らなければなりません（`cargo xtask wasm` を実行）。ファイルシステムのコードは `cfg(not(target_arch = "wasm32"))` にするか、プラットフォームサービスを経由させます。
8. **性能は機能である。** 重い処理は 24–36 MP の画像でリリースビルドでベンチマークします。タイル単位で並列に処理し（rayon）、空のタイルはスキップし、毎フレーム全サーフェスを走査せず（リビジョンごとにキャッシュ）、前後のタイミングを開発ログに記録します。

9. **入力でパニックしない**（上記の *決してクラッシュさせない* を参照）。コマンドの `run` クロージャとそこから呼ばれるものはすべて、*どんな* パラメータやドキュメント状態に対しても、パニックではなく `Err` を返さなければなりません：パラメータを検証し、インデックス参照や除算の前に境界を確認し、確保の前に不合理なサイズを拒否します。統合テスト `panic_hunt` は敵対的なパラメータで全コマンドをファズし、常にグリーンでなければなりません。

## 4. 作業の選び方

優先順位：重要なインフラが最優先、次に手軽なパリティ、最後にロングテール。

0. **まず `docs/roadmap.md` → 「Honest parity assessment」を読む。** そこには、観点ごとに
   PhotoCraft がどこで不足しているか、そしてどこへ向かうかの優先順位が書かれています。`docs/parity.md`
   （メニューの接続状況）は挙動の尺度ではありません。あなたの作業が計測値（PSD オラクル、
   ラウンドトリップ、ワークフローテスト、性能）を動かしたら、そのセクションを日付付きの数値で更新してください。
1. `docs/roadmap.md` → **Current focus**。
2. `cargo xtask parity` → `docs/parity.md` に、未実装のメニュー項目がすべてメニューごとに一覧されます。手軽な成果は、たいてい `algo`、`paint`、`vector`、`text` にアルゴリズムが既に存在する未実装コマンドです。
3. `log/devlog.md` → 最近のエントリの「Still open」の箇条書き。

パリティが上がったら、`crates/ui-egui/src/parity.rs` の `FLOOR` を引き上げてください（決して下げない）。

## 5. タスクを終える前に

```sh
cargo test -p <crates you touched>
cargo clippy -p <crates> --all-targets -- -D warnings
cargo xtask layers
cargo xtask wasm            # if you touched L0–L6
cargo xtask parity          # if you added commands; commit the regenerated docs/parity.md
cargo test -p photocraft-engine --test panic_hunt -- --ignored   # if you added/changed commands: no panic on adversarial input (Rule 9)
```

コマンドは不正な入力に対して **決してパニックしてはいけません**（ルール 9）：すべての `run` クロージャとそこから呼ばれるコードは、どんなパラメータやドキュメント状態に対してもパニックではなく `Err` を返します。新しいコマンドには、穏当に失敗することを確かめるテスト（空／範囲外／型違いのパラメータ → クラッシュではなく `Err`）を付けます。

その後、`log/devlog.md` に簡潔なエントリ（何を入れたか、数値、何が未完了か）を追記してください。セッションは突然終わることがある（クラッシュ、コンテキスト上限）ので、開発ログとグリーンなツリーが次のエージェントへの引き継ぎ手段になります。どのステップでもツリーがビルドできる状態を保ってください。

## 6. 並列エージェント

- Cargo のビルドロックを避けるため、自分専用のターゲットディレクトリ（`CARGO_TARGET_DIR=target/agent-<name>`）を使い、自分が担当するファイルだけを編集してください。共有ファイル（`engine/src/lib.rs`、`engine/src/commands.rs` 内の `v.extend(...)` のリスト、`ui-egui/src/menus.rs`、`state.rs`）には小さく局所的な編集のみを行い、編集前に読み直してください。
- 新しいコマンドは、共有ファイルを肥大化させるのではなく、**新しいモジュール**（`specs()` 関数を持つ `engine/src/<area>_cmds.rs`）に置いてください。
- 他者の作業途中の編集でビルドが壊れている場合は、待ってから再試行してください。他者のファイルを修正してはいけません。
- すべての `Cargo.toml` を常に有効な状態に保ってください：`crates/*` の glob のため、壊れたマニフェストが 1 つあるだけで全員のビルドが壊れます。**マニフェストの作成や書き換えはアトミックに行ってください**：`crates/` の外にある一時ファイルに書き込み、それを `mv` で所定の位置に移します。
- ディスク容量：ターゲットディレクトリはそれぞれ約 10 GB です。作業を終えたエージェントの `target/agent-*` ディレクトリは削除してください。

## 7. 何がどこで管理されているか


- `docs/roadmap.md`：マイルストーン M0–M12、ステータス、現在のフォーカス。
- `docs/parity.md`：生成された Photoshop メニューのカバレッジ。
- `docs/releasing.md`：リリースの作成（`cargo xtask version`、`release` ブランチ）、署名用シークレット、`packaging/` 内のパッケージングスクリプト。
- `../craftrules/release/playbook.md`：storytold の全アプリが署名済みリリースバイナリをビルドする方法（正式な手順。`docs/release-playbook.md` はここを指しているだけ）。`docs/releasing.md` は PhotoCraft 固有の事項。
- `plan/`（ローカル、gitignore 対象）：調査、パリティ計画、実行計画、見積もり。
- `log/`（ローカル、gitignore 対象）：開発ログ。
- 

## 8. ネイティブフォーマットを完全に保つ

`photocraft-format` は、`photocraft-doc` の構造体にフィールドが追加されると意図的にコンパイルに失敗するようになっており、
`.pcraft` の保存から何かが黙って欠落しないようにしています。ドキュメントにフィールドを追加したら、
`crates/format/src/manifest.rs` と `convert.rs` に `#[serde(default)]` 付きで追加し、古いファイルも読み込めるようにしてください。
そのフィールドに PSD の対応物がある場合は `crates/io` でもマッピングし、未知の PSD ブロックはそのまま保持してください。
