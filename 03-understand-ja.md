# 変更する前にコードを理解する

理解していないコードを編集すると、気付きにくいリグレッションが出荷されます。これは開発者にもエージェントにも当てはまります。エージェントの失敗には、たいてい 2 つの型があります。開発者の望みを誤読するか、作業を正しく行うためのコンテキストを持っていないかです。1 つ目には[プロンプトに何を書くか](./02-poteto-mode-ja.md#what-goes-in-a-prompt)で対処しました。このページでは 2 つ目に対処します。

pstack には、コードを理解するための入口が 4 つあります。`/how` はコードが現在何をしているのかを説明します。`/why` はコードがその形になった理由を掘り起こします。`/teach` はその両方を 1 つの説明にまとめます。`/recall` は、あるトピックについての開発者自身の最近のコンテキストを再構築します。どれも、開発者が確認できる言葉でエージェントに自分の理解を説明させます。開発者よりもコードに詳しいかもしれないエージェントは、こうして監督します。

![A detective studies a machine blueprint with a magnifying glass while robots fetch case files; the evidence board behind her links clues under /how and /why.](./images/understanding.jpg)

## 読み取り専用の調査から始める

原因がはっきりしないときは、修正ではなく調査結果を求めます。

```text
/poteto-mode investigate why background jobs time out every few hours. give me what we know, what data you used, and your best hypotheses. don't change any code yet.
```

「don't change any code yet」によって、このプロンプトは [Investigation プレイブック](https://github.com/cursor/plugins/blob/main/pstack/skills/poteto-mode/playbooks/investigation.md)に振り分けられます。このプレイブックは `/how` を実行し、動機に関する問いには `/why` を加え、出典付きの説明を返します。複数の選択肢から選ぶ問いには、トレードオフの表を添えた推奨案を返します。「what data you used」と尋ねると、エージェントは証拠と推測を分けて示します。調査結果が修正を示唆していれば、修正は新しいタスクとして始めます。

## `/how` で動作を追う

```text
/how do we dedupe notifications? is there an n+1 when we look up subscribers?
```

実際に抱いている疑問をそのまま尋ねてください。[`/how`](https://github.com/cursor/plugins/blob/main/pstack/skills/how/SKILL.md) はコードを読み、シニアエンジニアがそのサブシステムを新しいメンバーに説明するときの水準で、実行時の流れ、主要な型、分かりにくい部分を含めて答えます。大きなサブシステムでは、まず読み取り専用の探索役を 2〜4 つ並行して動かします。範囲の狭い問いであれば、そのまま読んで説明します。

## `/why` で経緯を掘り起こす

```text
/why was the retry limit set to five? does the reason still hold?
```

[`/why`](https://github.com/cursor/plugins/blob/main/pstack/skills/why/SKILL.md) は、未解決事件を調べる探偵のように動きます。まずソース管理から始め、続いて MCP が提供する証拠の種類（課題管理、長文のドキュメント、チームのチャット、オブザーバビリティ、エラー追跡、分析など）をすべて並行して照会します。レポートはすべての出典を示し、直接の証拠と推論を区別し、記録が乏しいときは「appears to」と書きます。何も見つからなかった場合もそのことを報告します。「誰も理由を書き残していない」こと自体が 1 つの答えだからです。

2 つのスキルは自然に組み合わせられます。経緯が混乱の原因だと疑われるときは、`do why first then how` というプロンプトで十分です。

## `/teach` で本当に理解する

```text
/teach me how this PR changes retries. convince me it fixes the cause and not the symptom.
```

[`/teach`](https://github.com/cursor/plugins/blob/main/pstack/skills/teach/SKILL.md) は、要約だけでは足りないときに使います。`/how` と `/why`（小さな変更ならどちらか一方だけの場合もあります）を実行し、その結果を、図を 1 枚ずつ積み上げていく平易な説明にまとめます。「convince me」という言い回しは、ぜひ真似してください。説明が単なる案内ではなく、開発者が突っ込みを入れられる論証になります。

このスキルは、エージェント自身の選択にも使えます。

```text
/teach me why you implemented it this way and not with a queue. what did you trade off, and why?
```

説明させることは、開発者だけでなくエージェントにも役立ちます。自分の作業を説明しなければならないエージェントは、自信ありげに述べて先に進むのではなく、コードを読み、主張の一つひとつを証拠で裏付ける必要があるからです。

## `/recall` で自分のコンテキストを再構築する

```text
/recall catch me up on the export work from last week
```

[`/recall`](https://github.com/cursor/plugins/blob/main/pstack/skills/recall/SKILL.md) は、開発者自身の最近のチャットと共有の記録（課題、過去の修正、まだ発生しているエラー）を調べ、現状と次にやることをまとめた概要を返します。古いチャットには、新しいエージェントが持っていないコンテキストが含まれています。そのため、古いトピックについて新しい作業を始めるときは、まずそのコンテキストを読み込み、それから新しい入力を渡します。

```text
/recall my work on the virtualized list from yesterday, then read this bug report.
```

特定のチャットを再開したい場合は、`/recall` ではなく、次に説明する Session pickup プレイブックを使います。

## Session pickup で以前の作業を引き継ぐ

別のエージェント（または先週の開発者自身）がブランチを作業途中のまま残している場合は、次のように依頼します。

```text
/poteto-mode take over this branch. read the decision log, figure out what's done, and continue from there. don't redo finished work.
```

[Session pickup プレイブック](https://github.com/cursor/plugins/blob/main/pstack/skills/poteto-mode/playbooks/session-pickup.md)は、以前の作業の記録を信頼できる情報源として扱います。ブランチの状態と判断を再構築し、再開地点を特定し、すべてを一から導き直すのではなく、引き継いだ主張を元の目標に照らして検証します。

**落とし穴：** 「どうせエージェントがコードを読むから」という理由で、このページのスキルを省略しないでください。動作を追った理解なしに編集を始めたエージェントは、最初に見つけたそれらしい箇所で症状だけを直しがちです。最初に `/how` を使うほうが、2 つ目のバグに対処するよりも安く済みます。

次へ: [変更を設計する](./04-design-ja.md)
