# foamFV — OpenFOAM fvOptions 検索・雛形生成ツール

[English](README.md) | 日本語

OpenFOAM（ESI 版、openfoam.com）には約50クラス、型の展開を含めると約70種類の fvOptions（ソース項・拘束・補正）があります。使うたびに「type 名は何か」「必須項目は何か」「selectionMode はどう書くか」をソースや tutorials から探すのは手間がかかります。
**foamFV** は、fv::option として登録されている全クラスのヘッダの doc コメント（`Description`、`Usage`、`\verbatim` の記述例、`\table` のパラメータ表）と、tutorials の `fvOptions` ファイルを解析してデータベースを作ります。そのデータベースを使って、検索、表示、雛形の生成、ケースへの書き込みができます。

functionObject 用の姉妹ツール [foamFO](../functionObjects) と同じ使い方です。

| ファイル | 内容 |
|---|---|
| `foamFV` | CLI（Python 3.8 以上、標準ライブラリのみ） |
| `catalog.html` | 検索できるカタログ。ブラウザで開くだけで使えます（オフライン、単一ファイル） |
| `fvDatabase.json` | 同梱のデータベース（OpenFOAM-v2606 から生成） |
| `catalog_template.html` | カタログの雛形 |
| `skill/openfoam-fvoptions/` | Claude Code 用スキル |
| `install.sh` | インストール補助（スキルのリンク、PATH の設定） |

## インストール

```bash
cd fvOptions            # このディレクトリ
./install.sh            # ~/.claude/skills にスキルをリンクし、PATH 設定の1行を表示
./install.sh --bashrc   # ~/.bashrc に PATH を追記する場合（バックアップあり）
./install.sh --uninstall
```

使う OpenFOAM が v2606 以外の場合は、その環境を読み込んでからデータベースを作ってください（数秒で終わります）。

```bash
source /usr/lib/openfoam/openfoam2412/etc/bashrc
foamFV build            # -> ~/.local/share/foamFV/fvDatabase-openfoam2412.json
```

`$WM_PROJECT_DIR` の名前と一致するデータベースが自動的に選ばれます。

- **探索順**: `$FOAMFV_DB` → `~/.local/share/foamFV/fvDatabase-<版>.json` → 同梱の `fvDatabase.json`
- **版が違う場合**: 同梱のデータベースの版と環境の版が違うと警告が出ます（`FOAMFV_QUIET=1` で抑止できます）。

## 使い方

| コマンド | 説明 |
|---|---|
| `foamFV list [-g sources]` | 一覧を表示します（T=tutorials に使用例あり、Z=selectionMode によるセル選択、R=領域間） |
| `foamFV list --groups` | グループ別の件数を表示します |
| `foamFV search 発熱` | 名前、説明、パラメータ名、同義語を全文検索します（`porous`、`多孔質`、`発散`、`ファン` なども可） |
| `foamFV show limitTemperature [--inherited]` | 説明、記述例、パラメータ表（必須は赤、条件付きは紫）、継承した共通項目、Note、tutorials の場所を表示します |
| `foamFV example explicitPorositySource -m 3` | tutorials 内の実際の記述を抜き出して表示します（OpenFOAM 環境または `--src` が必要） |
| `foamFV add scalarSemiImplicitSource --zone heater --fields h` | 雛形を表示します（書き込みはしません） |
| `foamFV add limitTemperature -n limT --zone all --insert` | `constant/fvOptions` に追記します（無ければ FoamFile ヘッダ付きで作成。元のファイルは `fvOptions.bak`） |
| `foamFV add ... --full` | オプション項目もコメントとして含めます |
| `foamFV add ... --system` / `-region solid` | `system/fvOptions` / `constant/solid/fvOptions` を対象にします |
| `foamFV build [--src DIR] [-o FILE]` | データベースを作ります |
| `foamFV html` | `catalog.html` を作り直します |

雛形の作り方:
- ヘッダの記述例のコメント（`// Mandatory entries`、`// Optional entries`、`// Conditional ...`）とパラメータ表の Reqd 列から、必須項目を取り出します。
- cellSetOption を継承するもの（Z 印）には `selectionMode` / `cellZone` を加えます。interRegion 系には、基底クラスの必須項目（`nbrRegion`、`nbrModel` など）を加えます。
- `<Type>` 付きのテンプレート型（`<Type>SemiImplicitSource` など）は、`scalarSemiImplicitSource` のような実名で引けます。基本名で指定した場合は `scalar…` になります。
- ヘッダの記述例が雛形に向かないもの（SemiImplicitSource、explicitPorositySource）は、ヘッダと tutorials をもとに作った最小例を使います。ヘッダに記述例が無いもの（rotorDisk など）は、tutorials の記述を元にします。
- `-case` のケースから cellZone 名（`constant/polyMesh/cellZones`）と `0/` の場の名前を読み、候補として表示します。存在しない cellZone を `--zone` に指定すると警告します。
- `<…>` はプレースホルダです。残っている場合は警告が出ます。

## catalog.html

ブラウザで開くだけで使えます。OpenFOAM がインストールされていなくても見られます。

- **左側**: インクリメンタル検索（`/` キーで検索欄へ）、グループの絞り込み、「tutorials に例がある」「セル選択あり」の絞り込み
- **右側**: すぐ使える雛形（`foamFV add` と同じ内容、Copy ボタン付き）、ヘッダの記述例、パラメータ表、継承した共通項目（cellSetOption の selectionMode など）、tutorials での使用箇所
- **項目を直接開く**: `catalog.html#limitTemperature` や `catalog.html#scalarSemiImplicitSource` のように末尾に名前を付けます

## Claude Code スキル

`install.sh` で `~/.claude/skills/openfoam-fvoptions` にリンクされます。
「heater という cellZone に 500 W の発熱を入れたい」のように頼むと、foamFV でソースの記述を確認し、ソルバのエネルギー変数（T/h/e）や cellZone を確認してから fvOptions を書きます。名前で呼ぶ場合は `/openfoam-fvoptions` です。

## OpenFOAM の既存機能とファイルの扱い

### ファイルの場所
- `constant/fvOptions` を読みます。**このファイルが無い場合に限り** `system/fvOptions` を読みます（両方あると system 側は無視されます。`src/finiteVolume/cfdTools/general/fvOptions/fvOptions.C`）。
- マルチリージョンの場合は `constant/<region>/fvOptions` です。
- 実行ログの `Selecting finite volume options type …` と `selected N cell(s) with volume …` で、適用されたことと選択されたセル数を確認できます。

### cellZone の作成（topoSet）
```cpp
// system/topoSetDict
actions
(
    { name heater; type cellZoneSet; action new; source boxToCell; box (0 -0.01 -1) (0.05 0.01 1); }
);
```

### `foamDictionary` などの既存コマンド
```bash
grep -rh --include=fvOptions -E "^\s{4}type\s" $FOAM_TUTORIALS | sort | uniq -c   # types used in the tutorials
foamDictionary constant/fvOptions -expand             # syntax check (resolves #include)
cat $WM_PROJECT_DIR/etc/caseDicts/general/fvOptions/porosity/porousZone   # porosity template
```
- `foamSearch` は fvOptions の名前付きエントリ内の `type` を拾えないため、上のように grep を使います。
- `<Type>CodedSource` のコードの雛形は `etc/codeTemplates/dynamicCode/codedFvOptionTemplate.{C,H}` にあります。

## 制限と注意
- 対応しているのは **ESI 版（openfoam.com, vYYMM）** です。Foundation 版（openfoam.org）の `fvModels`/`fvConstraints` には対応していません。
- finite-area の faOptions は対象外です。
- ヘッダの記述が不完全な fvOption は、例や表を取り出せません。`foamFV build -v` で一覧を確認でき、その場合は `foamFV show` に表示されるヘッダを直接確認してください。
- ヘッダの記述そのものの誤り（例: limitVelocity の説明にある "(i.e. T)"）は、そのまま表示されます。
- 動作確認の範囲:
  - DB の生成: v2606（ソース）と v2412（パッケージ版）
  - 雛形を使った実行: v2412（simpleFoam/pitzDaily に cellZone を作成し、`vectorSemiImplicitSource` と `limitVelocity` を追加）

## ライセンス
GPL-3.0-or-later（[LICENSE](LICENSE)）。
同梱の `fvDatabase.json` と `catalog.html` には、OpenFOAM のソース（GPL-3.0-or-later、© OpenCFD Ltd. ほか）の doc コメントの文章が含まれます。

OPENFOAM® is a registered trade mark of OpenCFD Limited, producer and distributor of the OpenFOAM software via www.openfoam.com. This offering is not approved or endorsed by OpenCFD Limited.
