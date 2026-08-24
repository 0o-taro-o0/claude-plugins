# 導入手順

プラグイン workflow の雛形から、この開発体制をプロジェクトへ導入するときに一度だけ行う。

1. 雛形2枚(`templates/CLAUDE.md`、`templates/CONTRIBUTING.md`)をプロジェクトルートへ複写し、`<>` の欄を埋める
2. Issue テンプレート2種(`templates/ISSUE_TEMPLATE/task.md`、同 `decision-request.md`)を `.github/ISSUE_TEMPLATE/` へ複写する
3. ラベルを作る(プルリクエストの状態を示す4種と、課題の起票用の1種)

ラベルは次のコマンドで作る。

    gh label create 実装中 --color FBCA04 --description "実装役が作業している"
    gh label create レビュー中 --color 1D76DB --description "レビュー役が読んでいる、または指摘に対応している"
    gh label create 判断待ち --color D93F0B --description "保留した指摘があり、オーナーの判断を待っている"
    gh label create マージ可 --color 0E8A16 --description "指摘がすべて解消し、検証も通っている"
    gh label create 課題 --color C5DEF5 --description "作業中に気づいた課題"

ラベルの色は、このコマンドの `--color` が唯一の定義箇所である。複数のリポジトリで
色を揃える場合も、この手順の色をそのまま使う。既存のラベルと色が異なる場合は
`gh label edit <ラベル名> --color <色>` で合わせる。
