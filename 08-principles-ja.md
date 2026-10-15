# 原則の名前で舵を取る

pstack には、24 個の原則がそれぞれ独立したスキルとして含まれています。`/poteto-mode` は、複数ステップのタスクを始めるたびに原則の索引を読み、タスクに該当するものを適用し、適用した原則の名前と、それによって変わった判断を返信に記します。

開発者が原則を呼び出す必要はありません。原則の名前は、作業の向きを変えるために使います。それぞれの名前は、エージェントがすでに読んでいる完結した規則を指しているので、1 つの言い回しで、段落 1 つ分の指示よりも正確に作業の向きを変えられます。

## 実際の舵取り

エージェントが、既存の 3 つのアダプターに新しいアダプターを継ぎ足そうとしている場合は、次のように伝えます。

```text
use subtract before you add. delete the obsolete adapters first, then design what's left.
```

ビルドが通ったことを理由にエージェントが成功を主張している場合は、次のように伝えます。

```text
apply prove it works. run the real import flow and show me the written records.
```

並行する 2 つの試行が同じブランチに書き込もうとしている場合は、次のように伝えます。

```text
separate before serializing shared state. give each attempt its own worktree, no locks.
```

どの言い回しも意図どおりに伝わるのは、その背後にある規則が具体的だからです。それでもエージェントは、その規則によってどの判断が変わったのかを返信に書く必要があります。判断の変化を伴わない原則の引用は、適用せずに名前だけを持ち出したことを示す兆候です。

## 24 個の原則の概要

中核の原則は、どれだけ作るか、いつ設計を考え直すかを決めます。

- [Laziness Protocol](https://github.com/cursor/plugins/blob/main/pstack/skills/principle-laziness-protocol/SKILL.md) は、削除と、問題を解決する最小の変更を優先します。
- [Foundational Thinking](https://github.com/cursor/plugins/blob/main/pstack/skills/principle-foundational-thinking/SKILL.md) は、ロジックを書く前に中核のデータ構造を選びます。
- [Redesign from First Principles](https://github.com/cursor/plugins/blob/main/pstack/skills/principle-redesign-from-first-principles/SKILL.md) は、新しい要件を、初日からあったかのように統合します。
- [Attack the Premise](https://github.com/cursor/plugins/blob/main/pstack/skills/principle-attack-the-premise/SKILL.md) は、どの関係者が不均衡を抱えているかを洗い出したうえで、2 回以上失敗した修正に共通する前提を問い直します。
- [Subtract Before You Add](https://github.com/cursor/plugins/blob/main/pstack/skills/principle-subtract-before-you-add/SKILL.md) は、上に積み上げる前に無駄を取り除きます。
- [Minimize Reader Load](https://github.com/cursor/plugins/blob/main/pstack/skills/principle-minimize-reader-load/SKILL.md) は、読み手が頭の中に保持しなければならないレイヤーや隠れた状態を減らします。
- [Outcome-Oriented Execution](https://github.com/cursor/plugins/blob/main/pstack/skills/principle-outcome-oriented-execution/SKILL.md) は、使い捨ての互換性のための状態を残すのではなく、書き直しを目標の設計へと収束させます。
- [Experience First](https://github.com/cursor/plugins/blob/main/pstack/skills/principle-experience-first/SKILL.md) は、実装の都合よりもユーザーが得る結果を選びます。
- [Exhaust the Design Space](https://github.com/cursor/plugins/blob/main/pstack/skills/principle-exhaust-the-design-space/SKILL.md) は、前例がないときに、競合するプロトタイプを 2〜3 個作ります。
- [Build the Lever](https://github.com/cursor/plugins/blob/main/pstack/skills/principle-build-the-lever/SKILL.md) は、作業を行う、または作業を証明するスクリプトを作り、レビュアーが再実行できるようにします。エージェントが同じことを手作業で繰り返しているなら、欲しいと思っている道具やスキルを書かせます。スクリプトでいつも同じように実行できるステップにはスクリプトを使い、エージェントは判断が必要な場面のために残しておきます。

アーキテクチャの原則は、状態、検証、互換性をどこに置くかを決めます。

- [Model the Domain](https://github.com/cursor/plugins/blob/main/pstack/skills/principle-model-the-domain/SKILL.md) は、繰り返し現れる規則を、散らばった条件分岐ではなく 1 つの構造で表現します。
- [Boundary Discipline](https://github.com/cursor/plugins/blob/main/pstack/skills/principle-boundary-discipline/SKILL.md) は、境界で検証し、内部の型は信頼します。
- [Type System Discipline](https://github.com/cursor/plugins/blob/main/pstack/skills/principle-type-system-discipline/SKILL.md) は、不正な状態を表現できないようにします。
- [Make Operations Idempotent](https://github.com/cursor/plugins/blob/main/pstack/skills/principle-make-operations-idempotent/SKILL.md) は、再試行しても同じ最終状態に収束するようにします。
- [Migrate Callers Then Delete Legacy APIs](https://github.com/cursor/plugins/blob/main/pstack/skills/principle-migrate-callers-then-delete-legacy-apis/SKILL.md) は、移行と削除を 1 回の波で行います。
- [Separate Before Serializing Shared State](https://github.com/cursor/plugins/blob/main/pstack/skills/principle-separate-before-serializing-shared-state/SKILL.md) は、調停の仕組みを足す前に、共有そのものをなくします。

検証の原則は、何を証明と見なすかを定めます。

- [Prove It Works](https://github.com/cursor/plugins/blob/main/pstack/skills/principle-prove-it-works/SKILL.md) は、代理の指標ではなく実際の成果物を検証します。
- [Fix Root Causes](https://github.com/cursor/plugins/blob/main/pstack/skills/principle-fix-root-causes/SKILL.md) は、コードを変更する前に再現し、原因までたどります。
- [Sequence Work into Verifiable Units](https://github.com/cursor/plugins/blob/main/pstack/skills/principle-sequence-verifiable-units/SKILL.md) は、小さな単位の作業をそれぞれ確認で締めくくってから、次の単位を始めます。
- [Test Behavior, Not Implementation](https://github.com/cursor/plugins/blob/main/pstack/skills/principle-test-behavior-not-implementation/SKILL.md) は、利用者と同じ方法でコードを呼び出して期待値をリテラルで検証し、インポートしたすべての関数が `undefined` を返しても通ってしまうテストを削除します。
- [Explain the Number](https://github.com/cursor/plugins/blob/main/pstack/skills/principle-explain-the-number/SKILL.md) は、誰かが計測値を信頼したり報告したりする前に、その数値を制限しているものを特定し、別のものを計測していた可能性を排除します。[`/benchmark-checklist`](https://github.com/cursor/plugins/blob/main/pstack/skills/benchmark-checklist/SKILL.md) は、これを実際の実行結果から答える 7 つの問いにしたものです。

委任の原則は、並行作業を健全に保ちます。

- [Guard the Context Window](https://github.com/cursor/plugins/blob/main/pstack/skills/principle-guard-the-context-window/SKILL.md) は、大量の読み込みをサブエージェントに回し、発見した内容をメインのチャットに残します。
- [Never Block on the Human](https://github.com/cursor/plugins/blob/main/pstack/skills/principle-never-block-on-the-human/SKILL.md) は、取り消し可能な作業は先に進め、その結果を提示します。

そして、メタな原則が 1 つあります。

- [Encode Lessons in Structure](https://github.com/cursor/plugins/blob/main/pstack/skills/principle-encode-lessons-in-structure/SKILL.md) は、2 回繰り返した助言を lint、確認、スクリプトに変えます。[`/correct`](https://github.com/cursor/plugins/blob/main/pstack/skills/correct/SKILL.md) は、これをリポジトリ全体に適用します。詳しくは[自分用にする](./09-make-it-yours-ja.md#fix-the-environment-with-correct)で説明しています。

このリストを暗記する必要はありません。いまはざっと目を通し、ここにある名前で防げたはずのことをエージェントがしているのに気付いたら、また戻ってきてください。そうすることで語彙が身につきます。

次へ: [自分用にする](./09-make-it-yours-ja.md)
