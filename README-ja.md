> **非公式の日本語訳です。** このリポジトリの `*-ja.md` は、[cursor/plugins の pstack ガイド](https://github.com/cursor/plugins/tree/main/pstack/docs/guide)（英語原文）を有志が翻訳したもので、Cursor や原著者による公式の翻訳ではありません。原文は MIT License（Copyright (c) 2026 Lauren Tan）で公開されており、本リポジトリにも同じ [LICENSE](./LICENSE) を同梱しています。内容に食い違いがある場合は原文を優先してください。

# pstack ガイド

pstack は、エージェントを細かく管理するのをやめたときにいちばん力を発揮します。開発者は、何を望むのかと、完了をどう判断するのかを伝えます。`/poteto-mode` がプレイブックを選び、各ステップで必要になったほかのスキルを実行し、その証拠を示します。このガイドでは、その習慣を現実的なプロンプトを使って身につけます。

このガイドで学ぶ内容は次のとおりです。

1. [pstack をセットアップする](./01-setup-ja.md)：プラグインをインストールし、使うモデルを選びます。
2. [作業を `/poteto-mode` に任せる](./02-poteto-mode-ja.md)：目標を渡し、プレイブックが選ばれる様子を確認します。
3. [コードを理解する](./03-understand-ja.md)：何かを編集する前に、読み取り専用の調査を行い、続けて `/how`、`/why`、`/teach`、`/recall` を使います。
4. [変更を設計する](./04-design-ja.md)：コードの形が固まる前に、`/architect`、`/arena`、`/swarm`、`/interrogate`、プロトタイプ、計画を使います。
5. [変更を実装して整える](./05-build-and-clean-ja.md)：実装系のプレイブック、`/tdd`、`/unslop`、`/no-comments` を扱います。
6. [検証して出荷する](./06-verify-and-ship-ja.md)：実際のアプリで動作を証明し、`/benchmark-checklist` で数値を吟味してから、焦点を絞った PR を開いてマージまで進めます。
7. [寝ているあいだに作業を進める](./07-overnight-ja.md)：ループの前に信頼を確立し、夜間作業の契約を結び、監査できる判断ログを残し、1 つのエージェントの規模を超えるために Projects とオートメーションを使います。
8. [原則の名前で舵を取る](./08-principles-ja.md)：作業の途中でエージェントの向きを変える 24 個の名前を紹介します。
9. [自分用にする](./09-make-it-yours-ja.md)：自分専用のモード、繰り返されるミスに対する `/correct`、スキルの変更をテストする方法を扱います。
10. [レシピと落とし穴](./10-recipes-and-pitfalls-ja.md)：そのまま使えるプロンプトと、避けるべきミスをまとめています。

初回はページを順番に読んでください。2 回目以降は、どのページも単独で読めます。

行き詰まったときや、どのスキルが合うのか判断できないときは、質問を添えて [`/poteto-help`](https://github.com/cursor/plugins/blob/main/pstack/skills/poteto-help/SKILL.md) と入力します。

```text
/poteto-help which skill should i use to review this branch?
```

このスキルは質問に答え、送信用のプロンプトを示し、回答の根拠になったスキルやガイドのページへのリンクを添えます。ただし作業は開始しません。pstack の実行は実際にトークンを消費するので、準備ができた時点で開発者がプロンプトを送る仕組みになっています。`/poteto-help` は、開発者が入力したときにだけ動きます。

## 1 つだけ覚えるとしたら

エージェントには、目標とその確認方法を自分の言葉で渡してください。

```text
/poteto-mode the export writes duplicate rows when a retry lands mid-run. repro first, then fix and verify.
```

プレイブックの名前を挙げたり、スキルを列挙したりする必要はありません。`/poteto-mode` が振り分けに必要とする手がかりは、「repro first」と確認可能な結果だけで足ります。この例では Bug fix プレイブックが選ばれ、その手順が todo リストにコピーされ、各ステップの実行時に適切なスキルが呼び出されます。

次へ: [pstack をセットアップする](./01-setup-ja.md)
