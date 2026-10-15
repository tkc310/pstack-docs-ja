# レシピと落とし穴

前半はそのまま使う価値のあるプロンプトを、後半は誰もが一度はするミスを紹介します。パスや完了条件は自分のものに置き換えてください。レシピはあえてくだけた書き方にしてあります。実際にはそのように入力されるものですし、スキルはそれでも意図を正しく読み取ります。

![She tastes a finished dish while robots cook from a recipe box, with pinned cards reading /how, /tdd, and /loop above the counter.](./images/recipes.jpg)

## なじみのないサブシステムを理解する

```text
use /how first to understand how this initialization works. then use /why to figure out why it broke recently.
```

仕組みを先に、経緯を後に調べます。各スキルのレポートには調べた情報源が示されるので、答えが何に基づいているのかが分かります。

## コードに触れる前に、雑多な報告を言い直させる

```text
/poteto-mode read this thread. restate the underlying issue in your own words, in plain english. don't change any code yet.
```

誤読があれば言い直しの段階で現れ、メッセージ 1 通で訂正できます。エージェントが自分の見解を述べるまでは、自分の仮説は伏せておきます。

## 選ぶ前にプロトタイプを作る

```text
/poteto-mode prototype a few options for the settings layout. put them behind a switcher and send me screenshots of each.
```

説明からではなく、実際に動くものから選べます。その過程で、エージェントはレイアウトやタイミングに関する自分の疑問にも答えます。

## 固まった設計を計画にする

```text
/poteto-mode turn this design into a plan. small verifiable PRs, each with its own proof.
```

依頼するのは、設計が固まってからにします。成果物は計画そのものであり、計画にはそれを実行するプレイブックが示されます。

## 設計についてセカンドオピニオンを得る

```text
ask /arena for a second opinion on this thread and our approach
```

現在の設計は、複数の候補のうちの 1 つになります。統合の結果を見れば、パネルがより良いものを見つけたのか、現在の設計を裏付けたのかが分かります。大きな決定の前に打てる、安価な保険です。

## 独立した区画を並行して確認する

```text
/swarm check every package under packages/ against its check.sh. one worker per package. one report.
```

各ワーカーが 1 つのパッケージを受け持ちます。親はすべての区画の完了を待ち、ワーカーの生の出力ではなく、`PASS`、`ISSUES`、`BLOCKED` による 1 つのレポートを返します。

## ブランチを懐疑的にレビューする

```text
/interrogate the whole branch, but skeptically. don't change anything yet. no nitpicks unless it's an actual bug or regression in behavior.
```

添えた条件には実際の効果があります。「don't change anything yet」は読み取り専用に保ち、細かい指摘を除外する条件は雑音をあらかじめ取り除くので、`Act on` の指摘は時間をかけて読む価値のあるものになります。

## 失敗するテストを通じてバグを直す

```text
/poteto-mode repro the duplicate write first. if there's a cheap test path, /tdd it. then fix and rerun.
```

「if there's a cheap test path」が重要です。壊れやすいモックを使って無理にテストを書いても、実際のコマンドを実行するほどの証明にはならず、プレイブックはそのことを指摘してよいことになっています。

## 証拠付きで報告を再現して直す

```text
/poteto-mode repro this with /verify-<app>. if it repros on main, fix it and show me a video as proof.
```

「if it repros on main」と書いておけば、バグがすでに解消されている場合に実行を早めに止められます。動画があれば、diff を読む前に修正を確認できます。

## 数値を公開する前に吟味する

```text
/benchmark-checklist vet this 40% speedup before it goes in the pr description
```

速くなった、遅くなった、計測可能な差はない、判断できない、のいずれかの判定が、実行回数、範囲、数値を制限しているものと一緒に返されます。

## 同じミスの訂正をやめる

```text
/correct agents keep adding new config flags without registering them in the schema
```

修正はアーキテクチャ、型、lint、テストのいずれかとしてリポジトリに入るので、次のエージェントは同じミスをできなくなります。

## 作業を始めずにやり方を尋ねる

```text
/poteto-help how do i get poteto-mode to stay on every turn?
```

回答、送信用のプロンプト、情報源へのリンクが返されます。そのプロンプトを送るまで、何も実行されません。

## 離れているあいだも実行を正直に保つ

```text
im going to bed, keep going autonomously until every fixture passes. do not stop. keep a decision log i can audit in the morning.
```

契約の全体は[夜間作業のページ](./07-overnight-ja.md)にあります。タスクと完了条件がすでに会話の中にあれば、この短い形で機能します。

## 逸れていく実行を軌道修正する

舵取りのプロンプトは 1 行で済みます。

```text
i said the goal is to repro. i did not ask for a fix yet.
```

```text
apply prove it works. show me the real output, not the build log.
```

```text
/unslop that, no emdashes
```

多くの言葉が必要になることはまれです。必要なのは適切な名前であり、その語彙は[原則のページ](./08-principles-ja.md)にあります。

## 返信を平易な言葉にしてもらう

```text
/bro
```

プロンプトはこれだけです。[`/bro`](https://github.com/cursor/plugins/blob/main/pstack/skills/bro/SKILL.md) は、直前のメッセージを、人が人に話すように、専門用語を使わず短く言い直します。返信が技術的には詳しいのに、結局何を言っているのか分からないときに使います。

## 落とし穴

- **プロンプトでスキルを列挙する**：「use /how then /architect then /arena」は、プレイブックがすでに決めている手順の順序を入れ替えてしまいます。目標と制約を述べ、スキルの名前は既定の選択を上書きしたいときだけ挙げます。
- **曖昧な完了条件**：「make it better」では、`/loop` が確認するものが何もありません。成功か失敗かを判定できるコマンドや成果物を渡します。
- **原因についての自分の仮説から始める**：エージェントは指し示された場所を探します。まず問題を言い直させてから、自分の見立てを伝えます。
- **最初の設計を採用する**：1 回の試行では、モデルが最初に思いついた形で固まります。プロトタイプか `/architect` を求め、証拠に基づいて選びます。
- **抽象的な計画を磨き続ける**：裏付けるコードのない計画に敵対的レビューをかけると、決して起こらないリスクが作り出されます。プロトタイプで疑問を解消してから、実際に作られたものをレビューします。
- **1 つの worktree で複数のエージェントを並行させる**：互いの変更を上書きし、diff は発掘調査のような状態になります。cloud agent として動かすか、「own worktree per attempt」と伝えます。
- **信頼する前にループを回す**：自分の作業を検証できないループは、未確認の作業を速く生み出すだけです。先に検証スキルを動くようにします。
- **吟味していない数値を信頼する**：温まったキャッシュや、通らなかったコードパスによって、高速化が見かけ上生じることがあります。数値をどこかに出す前に `/benchmark-checklist` を実行します。
- **同じミスを手作業で訂正する**：チャットでの訂正は 1 回の実行にしか効果がありません。`/correct` はリポジトリを直すので、以降の実行で同じミスは繰り返されません。
- **網羅のために `/arena` を使う**：`/arena` は 1 つの設計またはコードの依頼を繰り返し、土台を選んで最も良い部分を移植します。`/swarm` は区画や宣言済みのレースのアームに分割し、1 つのレポートに集約します。
- **すべてのレビューコメントを受け入れる**：ボットも人間も、本物の指摘と雑音を 1 つのリストに並べます。`/interrogate` は指摘を、対応すべきものと却下するものに理由付きで分類し、開発者はどちらの判断も覆せます。
- **`auto` をモデルのスラッグとして扱う**：`auto` と `inherit-parent` は「model フィールドを省略し、サブエージェントに親チャットのモデルを引き継がせる」ことを意味します。ロールについては[セットアップ](./01-setup-ja.md)で説明しています。
- **ビルドが通ったことを根拠に成功を報告する**：ビルドが証明するのはコンパイルできることだけです。実際のコマンド、操作の流れ、保存された値、プロファイルを求め、返信に証拠が含まれていることを期待します。
- **`SKILL.md` を自由に書く**：[Authoring or modifying a skill プレイブック](https://github.com/cursor/plugins/blob/main/pstack/skills/poteto-mode/playbooks/authoring-a-skill.md)を経由させ、検証とレビューが行われるようにします。

ガイドは以上です。途中を飛ばして読んだ場合は、[セットアップ](./01-setup-ja.md)に戻って実際のタスクを 1 つ実行してください。習慣は読むことではなく、使うことで身につきます。

[ガイドの目次](./README-ja.md)に戻る
