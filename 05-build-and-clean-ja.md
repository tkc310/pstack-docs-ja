# 変更を実装し、diff を整える

実装系のプレイブックには共通の規律があります。開発者は観察したことを伝え、証拠の要求はプレイブックに任せます。このページでは、よくある実装タスクごとにプロンプトに何を書くかを示し、続いて diff をレビューしやすく保つための後片付けの習慣を説明します。

## 各実装プレイブックには既知の情報を渡す

バグのプロンプトでは、症状を述べ、まず再現を求めます。

```text
/poteto-mode this command emits two records after a retry. repro first, then fix and verify.
```

機能追加のプロンプトでは、求める振る舞いと、変えてはならないものを述べます。

```text
/poteto-mode add a --json flag. text output stays byte-identical. verify both forms.
```

リファクタリングのプロンプトでは、構造を動かす前に振る舞いを固定します。

```text
/poteto-mode move parsing into one module, zero behavior change. record the current output first and prove it's unchanged after.
```

性能のプロンプトでは、印象ではなく計測値を述べます。

```text
/poteto-mode startup takes 1.8s on this fixture. trace it, fix the measured cause, show me before and after.
```

これらはそれぞれのプレイブック（[Bug fix](https://github.com/cursor/plugins/blob/main/pstack/skills/poteto-mode/playbooks/bug-fix.md)、[Feature](https://github.com/cursor/plugins/blob/main/pstack/skills/poteto-mode/playbooks/feature.md)、[Refactoring](https://github.com/cursor/plugins/blob/main/pstack/skills/poteto-mode/playbooks/refactoring.md)、[Perf issue](https://github.com/cursor/plugins/blob/main/pstack/skills/poteto-mode/playbooks/perf-issue.md)）に振り分けられ、開発者が書かなかった手順はプレイブックが補います。具体的には、修正の前に再現し、実装の前にデータの形を定め、構造を変える前に振る舞いを固定し、最適化の前にプロファイルを取ります。

1 つの数値を継続的に改善したい場合は、[Hillclimb プレイブック](https://github.com/cursor/plugins/blob/main/pstack/skills/poteto-mode/playbooks/hillclimb.md)があります。指標、目標値、試行回数の下限を渡すと、固定した計測ハーネスを使って仮説を 1 つずつ試すループを回します。改善した変更は残し、それ以外はすべて元に戻します。

どちらの性能系プレイブックも、得られた数値に [`/benchmark-checklist`](https://github.com/cursor/plugins/blob/main/pstack/skills/benchmark-checklist/SKILL.md) を適用します。Perf issue は基準値とその後のすべての数値を吟味し、Hillclimb はハーネスを固定する前にそれを吟味します。自分で入力すべき場面については、[検証して出荷する](./06-verify-and-ship-ja.md#vet-a-measured-number-with-benchmark-checklist)で説明しています。

修正よりも先に原因を知りたい場合もあります。メモリリーク、アイドル時の CPU の空回り、表示の乱れのような、いま起きている症状には、[Runtime forensics プレイブック](https://github.com/cursor/plugins/blob/main/pstack/skills/poteto-mode/playbooks/runtime-forensics.md)が実行中のプロセスを計装します。すでに取得したプロファイルがある場合は、[Trace forensics プレイブック](https://github.com/cursor/plugins/blob/main/pstack/skills/poteto-mode/playbooks/trace-forensics.md)がその成果物を読み、負荷の高いフレームをソースに対応づけます。どちらも返すのは診断であり、修正は行いません。

```text
/poteto-mode here's a cpuprofile from the slow startup. tell me where the time goes and which source lines own it. no fix yet.
```

## `/tdd` で失敗するテストを先に書く

バグをローカルで安価にテストできる場合は、プロンプト全体が 2 語で済みます。

```text
/tdd implement
```

コンテキストがあれば、これで十分です。[`/tdd`](https://github.com/cursor/plugins/blob/main/pstack/skills/tdd/SKILL.md) は、意図した理由で失敗する最小のテストを書き、次に修正を書き、テストを再実行します。テストに大がかりなハーネスの準備や壊れやすいモックが必要になる場合、スキルはそのことを伝え、代わりに最も近い実行可能な確認を使います。実際のコマンドのほうが強い証拠になる場面で、無理にテストを書く必要はありません。

## TypeScript の規則は名前を指定して読み込む

[`typescript-best-practices`](https://github.com/cursor/plugins/blob/main/pstack/skills/typescript-best-practices/SKILL.md) は、型システムに関する原則を具体的な規則（判別可能なユニオン、境界での `unknown`、網羅的なバリアント、スキーマから導出した型）に落とし込みます。このスキルは自動では読み込まれないので、タスクが `.ts` や `.tsx` のファイルに関わるときは `/typescript-best-practices` と入力します。

## コミットする前に整える

[Opening a PR プレイブック](https://github.com/cursor/plugins/blob/main/pstack/skills/poteto-mode/playbooks/opening-a-pr.md)は、各コミットの前に diff に対して `/deslop` を実行し、PR の説明とコミット本文に [`/unslop`](https://github.com/cursor/plugins/blob/main/pstack/skills/unslop/SKILL.md) を適用します。`/deslop` は pstack ではなく `cursor-team-kit` プラグインに含まれています。手元にない場合は、同じ結果を平易な言葉で依頼してください。具体的には、処理を実況するだけのコメント、根拠のないガード、使われていない互換性のためのコードパス、無関係な編集を取り除くよう求めます。

文章に対しては、`/unslop` に対象と追加の規則を渡します。

```text
/unslop the readme changes, no emdashes
```

使ううちに自分なりの略記ができてきます。このスキルは、`unslop that, tighten it` のような短いプロンプトからでも意図を正しく読み取ります。

## `/no-comments` でコメントを取り除く

コメントには専用の確認が必要であり、それを書いたエージェント以外が行う必要があります。書き手は、開発者が自分のコメントを擁護するのと同じように、自分のコメントを擁護するからです。そのため、レビューの前に新しい目に渡します。

```text
/no-comments the diff
```

[`/no-comments`](https://github.com/cursor/plugins/blob/main/pstack/skills/no-comments/SKILL.md) は [Comment Sicko](https://github.com/cursor/plugins/blob/main/pstack/agents/comment-sicko.md) を起動します。これは読み取り専用のレビュアーであり、残してよいものの短いリストを持っています。具体的には、ライセンスヘッダー、公開 API のドキュメントコメント、コードでは説明できないことを説明するリンク、作り変えられない外部依存によって強制される振る舞いです。それ以外はすべて削除します。自分のコード内の意外な挙動は、この例外に含まれません。そのようなコメントはリファクタリングを促す指摘として返され、`/no-comments` は受け入れた指摘を根本原因で修正します。コメントが「do not remove」のように制約を主張している場合、スキルはその主張を型、テスト、lint として表現することを提案します。いずれの場合も、コメントは削除されます。

役割分担は明確にしておく価値があります。`/deslop` はコードから、`/unslop` は文章から質の低い記述を取り除き、`/no-comments` はコメントを、それを書いていないレビュアーに渡します。

**落とし穴：** 後片付けは任意の仕上げではありません。実況するコメントや防御的な無駄を含む diff は、レビュアーには未完成に見えますし、余分なコードは次のバグが潜む場所になります。diff が水増しされていると感じたら、レビューで指摘される前、コミットする前に `deslop it` と伝えてください。

次へ: [検証して出荷する](./06-verify-and-ship-ja.md)
