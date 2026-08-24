# 設計

bet-edge-finder で実証済みの運用一式を、Claude Code プラグインとして配布するための設計である。
次の実装プルリクエストの入力とする。

## 移植原則

移植であって再構築ではない。現に機能している bet-edge-finder の実物ファイル
(`CLAUDE.md`、`CONTRIBUTING.md`、`docs/templates/` の5書式)を正とする。
汎用化は「プロジェクト固有部分を、適用先プロジェクトが埋める欄に置き換える」ことだけに留め、
要件文書から新しく書き起こさない。現行からの変更は次の2点のみとする。

1. `docs/backlog.csv` の廃止と GitHub Issue への一本化
2. 人間への判断依頼の GitHub Issue 起票(Issue を正とする)

## リポジトリとプラグインの構成

claude-plugins リポジトリをプラグインマーケットプレイスとし、その中にプラグインを1つ置く。
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
  自動発動させない書式呼び出しには `disable-model-invocation: true` を付ける。
  `$ARGUMENTS` で引数を受けられる。`commands/` は旧形式であり使わない。
- 利用者は `/plugin marketplace add 0o-taro-o0/claude-plugins` で登録し、
  `/plugin install workflow@<マーケットプレイス名>` で導入する。

## 配布物一覧と移植対応表

移植元は `C:\Users\TaroTSUCHIYA\develop\ai\bet-edge-finder` の実物である。

| 移植元 | 配布物 | 形 |
|---|---|---|
| `CLAUDE.md` | `templates/CLAUDE.md` | 雛形。固有部分を欄に置換し、`docs/backlog.csv` への言及を GitHub Issue に置換 |
| `CONTRIBUTING.md` | `templates/CONTRIBUTING.md` | 雛形。同上。作業の流れ・役割3分担・ラベル4状態・コミット規約は原文のまま移す |
| `docs/templates/implementation-request.md` | `skills/implementation-request/SKILL.md` | 呼び出し用スキル。書式と注意事項を原文のまま含める |
| `docs/templates/review-request.md` | `skills/review-request/SKILL.md` | 同上 |
| `docs/templates/review-comment.md` | `skills/review-comment/SKILL.md` | 同上。「推奨」の退避先を backlog.csv から Issue 起票に置換 |
| `docs/templates/decision-request.md` | `skills/decision-request/SKILL.md` | 同上。書式を埋めたうえで GitHub Issue として起票する手順を加える(変更点2) |
| `docs/templates/work-report.md` | `skills/work-report/SKILL.md` | 同上。成果物欄の backlog id を Issue 番号に置換 |
| `docs/backlog.csv`(廃止) | `templates/ISSUE_TEMPLATE/task.md` | 課題起票用の Issue テンプレート。backlog.csv の列(概要・詳細・方針)を項目として引き継ぐ |
| `docs/templates/decision-request.md`(併用) | `templates/ISSUE_TEMPLATE/decision-request.md` | 判断依頼用の Issue テンプレート。書式の項目は decision-request と同一 |
| `CONTRIBUTING.md`「プルリクエストの状態」 | ラベル作成手順 | `実装中` `レビュー中` `判断待ち` `マージ可` の4種を `gh label create` で作る手順。`templates/CONTRIBUTING.md` の導入手順に記す |

Issue テンプレートは、適用先プロジェクトの `.github/ISSUE_TEMPLATE/` へ複写して使う。
雛形2枚(`CLAUDE.md`、`CONTRIBUTING.md`)も同様に、プロジェクトルートへ複写して欄を埋める。
複写を自動化するブートストラップは本設計の範囲外とする(後述の候補)。

## 雛形の欄

移植元の実物を読み、プロジェクト固有と判別した記述を欄に置き換える。汎用部分は原文のまま残す。

| 欄 | 移植元での中身(固有の実例) |
|---|---|
| 絶対ルールの中身 | 時系列リークの禁止、テスト期間は一度だけ |
| 判断を仰ぐことの固有例 | 収集の停止、`experiments/` の改変、封印、評価指標の変更 |
| レビュー役を2人にする条件 | 絶対ルール関連、封印の監査証跡、本番データの削除 |
| 検証コマンド | `uv run ruff check .`、`uv run pytest -q`、cfn-lint ほか |
| コミット範囲の候補 | `collect` `infra` `features` |
| 正本文書の構成(第4層の一覧と参照先表) | DESIGN、INFRA、DATA_INVENTORY、MILESTONES ほか |
| 作業別の手引きの一覧 | `docs/guides/` の3枚 |
| 環境と環境固有の注意 | Python 3.11・uv、`PYTHONUTF8=1`、worktree の `PYTHONPATH` ほか |
| 日本語校正の設定 | textlint、`.textlintrc.json`、抑止コメントの規約 |
| プロジェクトの目的(冒頭の説明) | 競艇の着順確率予測と回収率検証 |

役割3分担、ラベル4状態、最大2往復のレビュー、Conventional Commits(日本語1行)、
文書の4層構成、「上の層に下の層の内容を書かない」原則は汎用であり、欄にしない。

## 段階導入

最初の実装プルリクエスト(スライス1)に含めるのは、上記の配布物一覧の全てである。
すなわち `marketplace.json`、`plugin.json`、スキル5つ、雛形2枚、Issue テンプレート2種、
ラベル作成手順。

以降の候補(本設計には含めない)。

- 適用ブートストラップのスキル(雛形の複写・欄埋め・ラベル作成までを一括で行う)
- hooks による機械的強制(検証コマンドの実行漏れ検知など)
- 振り返りスキル(運用してみた結果を雛形へ還元する)

## 未確定事項

実装プルリクエストで検証して確定する。

- Web 版セッションへのプラグイン配布の挙動(マーケットプレイス登録とスキル呼び出しが同様に働くか)
- スキルから同プラグイン内 `templates/` のファイルを参照させる方法(パス解決の実際の挙動)
- Issue テンプレートの形式(Markdown 形式と YAML form 形式のどちらが運用に合うか)
