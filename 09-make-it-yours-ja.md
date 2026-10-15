# 自分用にする

poteto-mode は、ある 1 人の作業スタイルを反映したものです。その下にある仕組み（プレイブック、振り分け、モデルのロール）は、開発者自身のスタイルに合わせても同じようにうまく動きます。このページでは、個人用のモードの生成、セッションから得た教訓の記録、エージェントが同じミスを繰り返さないためのリポジトリの修正、焦点を絞ったスキルの作成、そして信頼する前のスキルの変更のテストについて説明します。

思っているよりも小さく始めてください。初日から多くのスキルは必要ありませんし、このプラグイン全体すら必要ありません。平易にプロンプトを書き、エージェントがどこで失敗するかを観察し、同じ失敗が 2 回現れたらスキルや確認を追加します。

## `/automate-me` で自分のモードを生成する

```text
/automate-me
```

開発者が自分のスタイルを説明する必要はありません。[`/automate-me`](https://github.com/cursor/plugins/blob/main/pstack/skills/automate-me/SKILL.md) が履歴からそれを読み取るからです。このスキルは、アクティブなワークスペースでの最近のトランスクリプトを調べ、返信、委任、検証、コード、文章、進め方について繰り返し現れる好みを洗い出し、どのパターンが本当に開発者らしいものかを尋ねます。そのうえで、Cursor 組み込みの `create-skill` フローを通じて `.cursor/skills/<your-name>-mode/SKILL.md` の草稿を作り、草稿に [`/unslop`](https://github.com/cursor/plugins/blob/main/pstack/skills/unslop/SKILL.md) をかけ、worktree から PR を開きます。そのため、ほかの変更と同じようにレビューできます。

習慣が変わってきたら、いつでも再実行してください。

```text
/automate-me update my mode skill with everything since its last edit
```

更新モードでは、スキルが最後に変更されてからの履歴だけを調べます。矛盾する証拠のない規則はそのまま残し、新しい証拠がある規則は修正し、本当に新しいパターンについてだけ節を追加します。

## `/reflect` でセッションの教訓を記録する

何かを学んだタスクの直後に、次を実行します。

```text
/reflect that took way too long. capture what we learned so the next run doesn't repeat it.
```

[`/reflect`](https://github.com/cursor/plugins/blob/main/pstack/skills/reflect/SKILL.md) は、トランスクリプトを 3 つのレビュアーに並行して送り、統合役がその提案を `Accepted`、`Rejected`、`Backlog` に分類します。スキルを変更する前には、開発者の承認を待ちます。提案を承認するのは、それが将来の判断を変える場合だけにしてください。1 回の奇妙なセッションは逸話であり、規則にはなりません。

<a id="fix-the-environment-with-correct"></a>
## `/correct` で環境を直す

同じミスについて何度もエージェントを訂正しているなら、その修正は次のプロンプトではなくリポジトリに入れるべきです。選択肢を、効果が持続する順に並べると次のようになります。

1. アーキテクチャや、より良いデータ構造によって、そのミスを起こせないようにする。
2. 型、または修正方法をエラーメッセージで示す lint や CI のチェックによって防ぐ。
3. テストで検出する。
4. ドキュメントやエージェント向けのルールとして書き残す。エージェントがルールを無視しても何も失敗しないので、これは最後の手段にする。

人間によるレビューはこのリストに含まれません。PR のたびに同じミスを見つけなければならないレビュアーこそが、この仕組みで解決したい問題だからです。この作業は [`/correct`](https://github.com/cursor/plugins/blob/main/pstack/skills/correct/SKILL.md) が行います。

```text
/correct agents keep calling the database client directly instead of going through the repository layer
```

このスキルは、最近のコミット、リバート、レビューコメント、回避策を説明するコメントを読み、ミスを種類ごとにまとめます。ある種類のミスは、2 回起きた時点で数えます。最も頻度の高い種類から順に、うまくいく最も高い段階で、1 種類につき 1 コミットで修正し、新しいチェックが過去の実際のミスで失敗することを証明します。また、エージェント向けの指示ファイルに、各ルールとそれを強制する仕組みを対にした表を保持します。そのため、何も強制していないルールは、繰り返し発生するミスとして表面化します。返信には、ミスの種類ごとに、その証拠、選んだ段階、より高い段階が使えなかった理由が列挙されます。

引数なしで実行すると、履歴からミスの種類を自分で見つけます。`/reflect` と `/correct` は役割を分担しています。`/reflect` は 1 回のセッションからスキルを改善します。`/correct` は、ある種類のミスが再発しないようにリポジトリを変えます。修正が新しい境界になる場合は、`/architect` と組み合わせてください。リポジトリ全体でその両方を行うプロンプトを、[多数のプロジェクトを並行して動かす](./07-overnight-ja.md#run-many-projects-in-parallel)で紹介しています。

## 焦点を絞ったスキルを作る

記録したいワークフローがすでに分かっている場合は、次のように依頼します。

```text
/poteto-mode write a skill for verifying database migrations in this repo
```

スキルの作成は、[Authoring or modifying a skill プレイブック](https://github.com/cursor/plugins/blob/main/pstack/skills/poteto-mode/playbooks/authoring-a-skill.md)に振り分けられます。このプレイブックは Cursor 組み込みの `create-skill` を経由し、フロントマターとリンクを検証し、Opening a PR プレイブックを通じて結果を出荷します。エージェント向けの文章には、人間向けの文章よりも高い水準が求められます。役に立たない 1 文が、将来どこかのエージェントが従う指示になってしまうからです。`SKILL.md` を自由に書くのではなく、その水準の維持はプレイブックに任せてください。

特別なケースが 1 つあり、それには専用の生成器があります。アプリを操作して振る舞いを証明しなければならないスキルは検証スキルなので、代わりに [`/create-verification-skill`](https://github.com/cursor/plugins/blob/main/pstack/skills/create-verification-skill/SKILL.md) と [`/maintain-verification-skill`](https://github.com/cursor/plugins/blob/main/pstack/skills/maintain-verification-skill/SKILL.md) を使います。両方とも[検証して出荷する](./06-verify-and-ship-ja.md#create-a-project-verification-skill)で説明しています。

## `/technical-writing` で基準に沿ったドキュメントを書く

出荷する文章はスキルだけではありません。ドキュメント、RFC、README、PR の説明、コミットメッセージには、次のように使います。

```text
/technical-writing review the readme changes
```

[`/technical-writing`](https://github.com/cursor/plugins/blob/main/pstack/skills/technical-writing/SKILL.md) は、疲れたエンジニアが一読で理解できる文章という 1 つの目標に向けて、階層化された基準を適用します。まず文書の様式（チュートリアル、ハウツー、リファレンス、解説）を決め、次に 1 文ずつ、誰が何をするのか、1 文に 1 つの考え、2 通りに読める箇所がないか、を確認します。開発者やエージェントが書いたばかりのものをレビューするときに使うか、ドキュメントを依頼するときに最初から指定してください。

## スキルの変更を盲検でテストする

スキルの編集は将来のすべてのセッションに影響するので、実験としてテストします。他人のスキルを取り入れる場合も同じです。残す前に、そのスキルによって開発者自身の作業でエージェントが良くなるかを確認してください。

```text
/poteto-mode run the eval playbook on this skill change. same task for both variants, candidates stay blind.
```

スキルが失敗を続けていて、どう動くべきか分かっている場合は、変更とテストを 1 つのタスクで行います。

```text
/poteto-mode update the review skill so it flags missing migrations, and eval the change.
```

最初から評価を求めておくと、編集が正直に保たれます。1 回の悪いセッションをもとに書いた修正は、そのセッションに過剰適合しがちであり、編集を重ねるうちにスキルは本来の姿からずれていきます。評価によって、そのずれを出荷前に検出できます。

[Eval プレイブック](https://github.com/cursor/plugins/blob/main/pstack/skills/poteto-mode/playbooks/eval.md)は、観察者効果という 1 つの失敗の型を中心に作られています。評価されていると知っているエージェントは、普段と違う振る舞いをします。そのため、候補のエージェントには、整理されたディレクトリで自然に見えるタスクを与え、「eval」や「candidate」という言葉を決して見せず、ほかの候補の存在も知らせません。1 つの審査役が、中立的なラベルのもとですべての出力を採点します。参照の連鎖をたどれたかどうかは、候補が主張する内容ではなく、各候補が実際に読んだファイルから採点します。

判定を受け入れる前に、すべての出力を自分で読んでください。審査役と意見が合わない場合は、自分の判断を疑う前にルーブリックを疑ってください。

## `/make-bot-ui` でボットの UI を作る

状況に応じて使うスキルの 1 つに、Grok Bot のユーザー向けのものがあります。[`/make-bot-ui`](https://github.com/cursor/plugins/blob/main/pstack/skills/make-bot-ui/SKILL.md) は、ボタンを押すと Webhook のルーチン経由でボットを起動する小さなページを作ります。たとえば、レビューのキューをスワイプしていき、スワイプのたびにその項目への対応をボットに依頼する、といった使い方ができます。Webhook の送信用キーは開発者のマシン上のサーバーが保持するので、キーがブラウザやチャットに渡ることはありません。このスキルは、ページを Tailscale で公開する方法も扱います。

**落とし穴：** スキルの挙動がおかしいからといって、タスクの途中でスキルを編集しないでください。スキルは専用の PR で修正し、タスクはそのまま進めます。機能開発に紛れ込んで出荷されたスキルの編集は、レビューで見えず、評価もできません。

次へ: [レシピと落とし穴](./10-recipes-and-pitfalls-ja.md)
