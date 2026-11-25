---
name: openfoam-fvoptions
description: Write, choose or check OpenFOAM (ESI/openfoam.com) fvOptions entries in constant/fvOptions or system/fvOptions — e.g. "add a heat source in a cellZone", "porous zone", "fix the bulk velocity of a channel", "limit temperature to avoid divergence", "actuator disk". OpenFOAM（ESI版）の fvOptions の設定を書く・選ぶ・確認するときに使う。「cellZone に発熱を入れたい」「多孔質（ポーラス）領域を設定したい」「チャネル流の平均流速を一定にしたい」「発散対策で温度を制限したい」「アクチュエータディスクを入れたい」のように、ソース項・拘束・補正の目的を自然文で受け取り、ソースのヘッダと tutorials に基づいて正しい fvOptions の記述を作る。
---

# OpenFOAM fvOptions 設定支援

`foamFV` は、fv::option として登録されている全クラスのヘッダ（Description/Usage/パラメータ表）と、tutorials の fvOptions ファイルから作ったデータベースを検索するツールです。
このスキルでは、**記述をすべて foamFV が返す内容（＝ソースの記述）に基づいて書く**ことを原則とします。記憶や推測で type 名やキーワード名を書かないでください。

ツールの場所:
- `foamFV` が PATH にあればそれを使います（`command -v foamFV` で確認）。
- PATH に無い場合は、このスキルのベースディレクトリから見て `../../foamFV` を使います（リポジトリ同梱）。

データベース:
- 同梱の DB は OpenFOAM-v2606 から作ったものです。
- ユーザーの OpenFOAM の版が違い、「bundled database is for ...」という警告が出た場合は、OpenFOAM 環境を source した状態で `foamFV build` を実行します（数秒）。作られた DB はその版で自動的に使われます。
- `example` コマンドで tutorials の本文を表示するには、OpenFOAM 環境（`$WM_PROJECT_DIR`）が必要です。環境が無い場合は `--src <OpenFOAMのルート>` を指定します。

回答はユーザーの言語で行います。

## 手順

1. **目的を fvOption に対応付ける**
   - `foamFV search <キーワード...>` で候補を探します（英語でも日本語でも可。例: `heat source`, `porous`, `pressure gradient`, `limit`, `damping`, `発熱`, `多孔質`）。
   - 候補が複数ある場合は summary を比較して選びます。`foamFV list -g <group>`（sources / general / constraints / corrections / interRegion / atmospheric ...）も参考にします。
   - よく使う対応:
     - 体積発熱・質量/運動量/スカラーの湧き出し → `scalarSemiImplicitSource` / `vectorSemiImplicitSource`
     - 多孔質（Darcy-Forchheimer） → `explicitPorositySource`
     - チャネル流・周期流の平均流速の維持 → `meanVelocityForce`
     - 温度・速度・乱流粘性の上下限（発散対策） → `limitTemperature` / `limitVelocity` / `limitTurbulenceViscosity`
     - 速度の減衰・吸収層 → `velocityDampingConstraint` / `acousticDampingSource`
     - 領域内の値の固定 → `<Type>FixedValueConstraint` / `fixedTemperatureConstraint`
     - 風車・プロペラ → `actuationDiskSource` / `radialActuationDiskSource` / `rotorDisk`
     - ファン → `fanMomentumSource`
     - 熱交換器 → `heatExchangerSource`、領域間熱伝達 → `constantHeatTransfer` など（interRegion）
     - 凝固・融解 → `solidificationMeltingSource`
     - 輻射 → `radiation`、粘性発熱 → `viscousDissipation`、浮力 → `buoyancyForce` / `buoyancyEnergy`
     - 任意のコード → `<Type>CodedSource`（例 `scalarCodedSource`）
   - この対応は手がかりにすぎません。必ず次の手順で裏付けを取ってください。
   - 回転領域（MRF）は fvOptions ではなく `constant/MRFProperties` です。境界条件で済む場合は fvOptions を使いません。

2. **正しい記述を確認する**
   - `foamFV show <type> --inherited` で記述例、パラメータ表（Reqd=yes が必須、cond は条件付き）、Note、継承した共通項目を確認します。
   - `<Type>` が付くもの（SemiImplicitSource、FixedValueConstraint、CodedSource など）は、場の型に合わせて `scalar…` / `vector…` などの実名を使います（`show` の Types 欄を参照）。
   - Z 印（cellSetOption を継承）のものは、`selectionMode`（all / cellZone / cellSet / points / geometric）が必須です。
   - R 印（interRegion）のものは、`nbrRegion` などの領域間の設定が必要です。
   - 実際の使い方は `foamFV example <type> -m 3` で tutorials から確認します。
   - 情報が足りない場合は、`show` に表示されるヘッダのパスのファイルを直接 Read して確認します。

3. **ケースを確認する**
   - **ソルバとエネルギー変数**: エネルギーの場は、ソルバによって `T`（scalarTransportFoam、buoyantBoussinesqSimpleFoam / buoyantBoussinesqPimpleFoam など）、`h`（多くの圧縮性・熱流体ソルバ。`constant/thermophysicalProperties` の `energy sensibleEnthalpy`）、`e`（`sensibleInternalEnergy`）のいずれかです。`fields` やソースの場の名前を合わせてください。発熱を `h` に与える場合の単位は W（absolute）または W/m3（specific）です。
   - **cellZone**: `constant/polyMesh/cellZones`（マルチリージョンの場合は `constant/<region>/polyMesh/cellZones`）に実在する名前を使います。無い場合は `topoSet`（`cellZoneSet` アクション）で作る方法を提示します。
   - **ファイルの場所**: `constant/fvOptions` か `system/fvOptions` です。`constant/fvOptions` があると `system/fvOptions` は読まれません（`src/finiteVolume/cfdTools/general/fvOptions/fvOptions.C`）。既存のファイルを確認し、使われている方に書きます。マルチリージョンの場合は `constant/<region>/fvOptions`（または `system/<region>/fvOptions`）です。
   - **libs**: `show` に `libs` が表示されるもの（例 atmosphericModels、radiationModels）は、ソルバがリンクしていない場合、エントリに `libs (...)` が必要です。

4. **設定を作る**
   - `foamFV add <type> -case <case> -n <entryName> [--zone <cellZone>|all] [--fields <names>] [--full]` で雛形を表示します（`--insert` を付けない限り表示だけです）。
   - `<…>` のプレースホルダは、ユーザーの条件（cellZone、場の名前、値）で置き換えます。foamFV が "Edit these placeholders" と警告したものは必ず埋めてください。
   - 物理量の単位（absolute か specific か、W か W/m3 か、`h` の場合は温度ではなくエネルギー）を説明に明記します。

5. **ケースに書き込む**（ユーザーが書き込みを求めた場合のみ）
   - `foamFV add <type> -case <case> -n <entryName> --zone <zone> --insert` を使います。`constant/fvOptions` が無ければ FoamFile ヘッダ付きで作られます。ある場合は末尾に追記され、`fvOptions.bak` が作られます（`--system` で `system/fvOptions`、`-region <r>` でリージョン別）。
   - 残ったプレースホルダは Edit で埋めます。
   - OpenFOAM 環境がある場合は、`foamDictionary constant/fvOptions -expand` で構文を確認します。
   - 実行時は、ログの `Selecting finite volume options type …` と `selected N cell(s)` で、適用されたことと選択セル数を確認するよう伝えます。

## 出力のしかた
- ユーザーの言語で説明し、最終的な fvOptions のエントリ（またはファイル全体）を示します。
- 必須パラメータの意味と、主要なオプションのデフォルト値を、パラメータ表に基づいて添えます。
- ファイルの場所、エネルギー変数（T/h/e）の前提、cellZone の作り方（必要な場合）を一言添えます。
