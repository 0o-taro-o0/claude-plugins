# 設計

bet-edge-finder で実証済みの運用一式を、Claude Code プラグインとして配布するための設計である。
配布物の構成と移植の方針を規定する。

## 移植原則

移植であって再構築ではない。現に機能している bet-edge-finder の実物
(`CLAUDE.md`、`CONTRIBUTING.md`、`docs/templates/` の5書式、運用の引き継ぎメモ、
文書整備の方針)を正とする。
汎用化は「プロジェクト固有部分を、適用先プロジェクトが埋める欄に置き換える」ことだけに留め、
要件文書から新しく書き起こさない。現行からの変更は次の4点のみとする。

1. `docs/backlog.csv` の廃止と GitHub Issue への一本化
2. 人間への判断依頼の GitHub Issue 起票と、決定の記録の判断依頼 Issue への一本化
3. 「判断を仰ぐこと」の、「後から取り消せない変更」「作業を制約する側を変える変更」
   「費用や所要時間が大きく変わる変更」の3分類への再構成
4. ラベル体系のアーティファクト種別での分離(後述の「運用の規定」)

## リポジトリとプラグインの構成

claude-plugins リポジトリをプラグインマーケットプレイスとし、その中にプラグインを1つ置く。
マーケットプレイス名は `taro-plugins` とする(リポジトリ名と同じ `claude-plugins` は、
CLI の検証で公式を騙る名前として拒否されるため使えない)。
プラグイン名は `workflow` とする。スキルは `/workflow:work-report` のように
プラグイン名を接頭辞として呼び出される。

構成は Claude Code 公式ドキュメント(code.claude.com/docs の Create plugins、
Create and distribute a plugin marketplace)で確認した形式に従う。

    claude-plugins/
    ├── .claude-plugin/
    │   └── marketplace.json        # マーケットプレイスの目録(リポジトリ直下に置く)
    └── plugins/
        └── workflow/               # プラグインルート
            ├── .claude-plugin/
            │   └── plugin.json     # プラグインの名前・説明・版
            ├── README.md           # プラグインの説明と導入手順
            ├── skills/             # スキル。<名前>/SKILL.md の形式
            │   └── .../SKILL.md
            └── templates/          # プロジェクトへ配置する雛形の置き場
                └── ...

形式の要点(公式ドキュメントで確認済み)。

- `marketplace.json` はリポジトリ直下の `.claude-plugin/` に置く。必須項目は
  `name`(マーケットプレイス識別子)、`owner`、`plugins`(各項目に `name` と `source`)。
  同一リポジトリ内のプラグインは `"source": "./plugins/workflow"` の相対パスで指す。
- `plugin.json` はプラグインルートの `.claude-plugin/` に置く。`.claude-plugin/` に
  入れてよいのは `plugin.json` だけで、`skills/` などの部品はプラグインルート直下に置く。
- スキルは `skills/<名前>/SKILL.md`。YAML frontmatter の `description` で用途を示し、
  自動発動させない呼び出し式のスキルには `disable-model-invocation: true` を付ける。
  `$ARGUMENTS` で引数を受けられる。`commands/` は旧形式であり使わない。
- 利用者は `/plugin marketplace add 0o-taro-o0/claude-plugins` で登録し、
  `/plugin install workflow@taro-plugins` で導入する。

導入手順(雛形と Issue テンプレートの複写、ラベルの作成)は配布物に含めず、
プラグインの説明文書 `plugins/workflow/README.md` に記す。導入時にプラグイン側で
読むものであり、適用先プロジェクトには置かない。

## 配布物一覧と移植対応表

移植元は `C:\Users\TaroTSUCHIYA\develop\ai\bet-edge-finder` の実物である。

### 共通の置換規則

移植原則の変更点に伴い、移植した配布物へ次の置換を一律に適用する。
対応表の「原文のまま」は、この規則の適用後を指す。

1. `docs/backlog.csv` への言及は、課題を GitHub Issue に起票する運用に置き換える(変更点1)。
   機械的な文言置換ではなく、起票の運用は `templates/CONTRIBUTING.md` に設ける
   「課題の起票」「判断依頼」の2節へ集約し、各所からはその運用を前提とした記述にする。
2. `docs/templates/<書式>.md` への参照は、対応するスキル呼び出し
   (`/workflow:work-report` など)に置き換える(書式のスキル化に伴う変換)。
   `CONTRIBUTING.md`「参照先」表の「依頼と報告の書式」の行は、書式のスキル5つの
   一覧に置き換える。

### 書式の形式

書式のスキルと Issue テンプレートは、名詞句見出しの Markdown 形式で統一する。

- スキルは、書式の全文を Markdown コードブロックとして含め、続けて注意事項を記す。
- Issue テンプレートは frontmatter(`name`、`about`、`labels`)を持ち、`labels` で
  種別ラベルを自動付与する。コマンド(`gh issue create`)からの起票では frontmatter が
  適用されないため、テンプレートと同じ見出し構成で本文を書き `--label` を付ける旨を
  `templates/CONTRIBUTING.md` と書式のスキルに記す。

### 対応表

| 移植元 | 配布物 | 形 |
|---|---|---|
| `CLAUDE.md` | `templates/CLAUDE.md` | 雛形。固有部分を欄に置換。「判断を仰ぐこと」は3分類(変更点3)で構成する |
| `CONTRIBUTING.md` | `templates/CONTRIBUTING.md` | 雛形。同上。作業の流れ・役割3分担・最大2往復のレビュー・コミット規約は原文のまま移す。「ラベル」「課題の起票」「判断依頼」「文書作成」の節を持つ |
| `docs/templates/implementation-request.md` | `skills/implementation-request/SKILL.md` | 呼び出し用スキル。書式と注意事項を原文のまま含める |
| `docs/templates/review-request.md` | `skills/review-request/SKILL.md` | 同上。ただし注意事項の `docs/DATA_INVENTORY.md` 固有規定は削除する。スキルは配布物でありプロジェクト側が欄を埋められないため、固有の差し戻し基準はスキルに持たせない |
| `docs/templates/review-comment.md` | `skills/review-comment/SKILL.md` | 同上。レビュー観点に「修正の跡が残っていないか」(文書修正の規範との対)を含める |
| `docs/templates/decision-request.md` | `skills/decision-request/SKILL.md` | 同上。書式を埋めたうえで GitHub Issue として起票する手順を含める(変更点2)。書式に「判断結果の記録先」の項目を持つ |
| `docs/templates/work-report.md` | `skills/work-report/SKILL.md` | 同上 |
| 運用の引き継ぎメモ | `skills/resident-director/SKILL.md` | 常駐指揮役モードのスキル。オーナーが明示的に任せたセッションが、残っている課題を横断的に消化するときの動き方を定める。発動条件「オーナーの明示委任」がスキルの起動と一致するため、雛形ではなく呼び出し式スキルとする |
| 文書整備の方針・運用の引き継ぎメモ | `templates/CONTRIBUTING.md`「文書作成」 | 規定文書と手順書の区別、現在形の規定だけを書くこと、`docs/guides/`(分野名)・`docs/runbooks/`(作業名)の命名、文書修正の規範として取り込む |
| `docs/backlog.csv`(廃止) | `templates/ISSUE_TEMPLATE/task.md` | 課題起票用の Issue テンプレート。見出しは「状況」「課題とする理由」「対応の方針案」「関連」。id と状態は Issue 番号と open/close が代替する。カテゴリと重要度は項目にせず落とす(必要になれば Issue ラベルで代替できる) |
| `docs/templates/decision-request.md`(併用) | `templates/ISSUE_TEMPLATE/decision-request.md` | 判断依頼用の Issue テンプレート。見出し構成はスキルの書式と同一(該当・状況・判断を要する理由・選択肢・推奨・放置した場合の影響・判断結果の記録先)。選択肢はタスクリストとし、オーナーが選んだ案にチェックできる |
| `CONTRIBUTING.md`「プルリクエストの状態」 | `plugins/workflow/README.md` の導入手順 | ラベル6種(プルリクエストの状態4種と Issue の種別2種)を `gh label create` で作る手順。ラベルの色は、この手順の `--color` を唯一の定義箇所とする |

Issue テンプレートは、適用先プロジェクトの `.github/ISSUE_TEMPLATE/` へ複写して使う。
雛形2枚(`CLAUDE.md`、`CONTRIBUTING.md`)も同様に、プロジェクトルートへ複写して欄を埋める。
複写を自動化するブートストラップは本設計の範囲外とする(後述の候補)。

## 運用の規定

配布物が定める運用のうち、設計上の決定に当たる要点を挙げる。全文は配布物にある。

### ラベル体系

ラベルはアーティファクト種別で分け、プルリクエストと Issue で共用しない。
プルリクエストのラベルは状態を、Issue のラベルは種別を表す。

| 対象 | ラベル |
|---|---|
| プルリクエスト(状態) | `実装中` `レビュー中` `保留` `マージ可` |
| Issue(種別) | `課題` `判断依頼` |

`保留` は、保留した指摘があり判断依頼 Issue の裁定を待っている状態を表す。
付けるときは対応する Issue 番号をプルリクエストへ返信し、裁定が出たら `レビュー中` へ戻す。

### 判断依頼と決定の記録

判断を求める文章はプルリクエスト本文・コメントに書かず、判断依頼 Issue を起票して
番号の参照だけを書く。決定の記録は判断依頼 Issue を正とする。専用の決定ログファイルは
持たず、クローズ済みの判断依頼 Issue の集合が決定の記録になる。

### マージ

マージは人間が行う。オーナーが明示的に許可した特定のプルリクエストだけは例外とし、
マージしてよい。

### 文書修正の規範

文書を修正するとき、過去の誤りや修正前の状態を匂わせる記述を残さない。
その瞬間に単体で読んで自然な、元からそうであったかのような文にする。
経緯や理由は文書本体ではなく、判断依頼の Issue とプルリクエスト本文に置く。

## 雛形の欄

移植元の実物を読み、プロジェクト固有と判別した記述を欄に置き換える。汎用部分は原文のまま残す。

| 欄 | 移植元での中身(固有の実例) |
|---|---|
| 絶対ルールの中身 | 時系列リークの禁止、テスト期間は一度だけ |
| 判断を仰ぐことの固有例 | 収集の停止、`experiments/` の記録の改変、評価指標の変更 |
| レビュー役を2人にする条件 | 絶対ルール関連、封印の監査証跡、本番データの削除 |
| 検証コマンド | `uv run ruff check .`、`uv run pytest -q`、cfn-lint ほか |
| コミット範囲の候補 | `collect` `infra` `features` |
| 規定文書の構成(第4層の一覧と参照先表) | DESIGN、INFRA、DATA_INVENTORY、MILESTONES ほか |
| 作業別の手引きの一覧 | `docs/guides/` の3枚 |
| 環境と環境固有の注意 | Python 3.11・uv、`PYTHONUTF8=1`、worktree の `PYTHONPATH` ほか |
| プロジェクトの目的(冒頭の説明) | 競艇の着順確率予測と回収率検証 |

役割3分担、ラベル体系、最大2往復のレビュー、Conventional Commits(日本語1行)、
文書の4層構成、「上の層に下の層の内容を書かない」原則は汎用であり、欄にしない。

## 段階導入

プラグインの初版(スライス1)に含めるのは、上記の配布物一覧の全てである。
すなわち `marketplace.json`、`plugin.json`、スキル6つ(書式5つと常駐指揮役)、
雛形2枚、Issue テンプレート2種、導入手順を含む `README.md`。

以降の候補(本設計には含めない)。

- 適用ブートストラップのスキル(雛形の複写・欄埋め・ラベル作成までを一括で行う)
- hooks による機械的強制(検証コマンドの実行漏れ検知など。Issue #7)
- 振り返りスキル(運用してみた結果を雛形へ還元する)
