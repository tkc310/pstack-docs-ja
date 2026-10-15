# 結果を検証して PR を開く

「コンパイルが通る」は証拠になりません。[Prove It Works 原則](https://github.com/cursor/plugins/blob/main/pstack/skills/principle-prove-it-works/SKILL.md)は、成功を報告する前に実際の成果物を確認するようエージェントに求めます。開発者の役割は、その「実際の成果物」を確認可能にすることです。このページでは、完了条件の述べ方、計測値の吟味、アプリ用の検証スキルの生成、PR の作成、そしてマージまでの進め方を説明します。

ほとんどのエージェント作業では、検証が最も時間のかかるステップです。検証は、たいてい人間の手を待つステップだからです。エージェントが自分で検証できるようにすれば、開発者がボトルネックになることはなくなります。検証を省くと、エージェントを増やしても、レビューすべき未確認の作業が増えるだけになります。

![A prototype plane flies a real test course while she times it with a stopwatch and robots film and checklist the run; the terminal reads verify: pass, evidence: captured.](./images/verification.jpg)

## 完了条件を最初に述べる

完了が何を意味するのかを、最初のプロンプトに、合う言葉で書いておきます。

```text
/poteto-mode add json output to this command. text output stays byte-identical, the json parses, both run against the sample project. show me the evidence.
```

これでエージェントは、満たすべき漠然とした気分ではなく、実行できる 3 つの確認を手にします。返信には、実行した正確なコマンドとその出力が含まれているはずです。確認を実行できなかった場合、良い返信は「inconclusive」と書きます。証拠のない自信ありげな返信は、危険信号として扱ってください。

確認は変更の種類に合わせます。

- CLI の変更では、実際のコマンドを実行します。
- UI の変更では、動いているアプリで変更した操作の流れをたどります。参照画像とピクセル単位で一致させる必要がある場合は、[Visual parity プレイブック](https://github.com/cursor/plugins/blob/main/pstack/skills/poteto-mode/playbooks/visual-parity.md)が、目視で判断する代わりに、固定したベースラインとスクリーンショットを比較します。
- パーサーや移行処理では、保存しておいた入力を再生します。
- 性能の変更では、変更前後のプロファイルを比較します。
- ストレージの変更では、書き込んだ値を読み戻します。

証拠は、開発者自身が確認できる成果物として求めます。たとえば、失敗するテストとその後に通るテスト、変更前後の動画、トレース、スクリーンショットです。修正がすでにマージされている場合は、main で同じ確認を再度求めます。成果物は、もっともらしい説明よりも優れています。実行全体を再現しなくても、成果物に対して疑問を投げかけられるからです。

完全には信頼できない小さな diff には、[`/blast-radius`](https://github.com/cursor/plugins/blob/main/pstack/skills/blast-radius/SKILL.md) を使うと、ほかの箇所で何が壊れうるかを洗い出せます。このスキルは、変更が安全であることの根拠となる 1 つの事実を選び、それについて長文を書くのではなく、コードを実行して証明します。

<a id="vet-a-measured-number-with-benchmark-checklist"></a>
## `/benchmark-checklist` で計測値を吟味する

変更前後の数値は、うっかり誤りやすい証拠の代表です。キャッシュが温まっていた、片方だけデバッグビルドだった、計測区間の中で処理が実際には実行されていなかった、のいずれかだけで、説得力のある高速化に見えてしまいます。数値を報告したり、それに基づいて行動したりする前に、次のように入力します。

```text
/benchmark-checklist vet the export speedup before it goes in the pr
```

[`/benchmark-checklist`](https://github.com/cursor/plugins/blob/main/pstack/skills/benchmark-checklist/SKILL.md) は 7 つの問いを立て、それぞれについて実行結果からの証拠を求めます。

1. 何がこの数値を制限しているのか。なぜ 2 倍ではないのか。
2. どの側も本番と同じ調整で実行したか。
3. 結果がディスク帯域やコア数のような物理的な限界を超えていないか。
4. エラーが出たり、誤った出力が返ったりしていないか。
5. 交互に実行しても再現するか。中央値と範囲はどうか。
6. ユーザーが実際に待つ経路で、エンドツーエンドに意味のある差か。
7. 計測区間の中で処理が実際に行われたか。

判定は、速くなった、遅くなった、計測可能な差はない、判断できない（inconclusive）のいずれかで返され、実行回数、範囲、制限要因が添えられます。制限要因を特定できない場合や、片方が調整なしで実行された場合は、inconclusive と判定します。`/poteto-mode` は Perf issue と Hillclimb のプレイブックの中ですでにこのチェックリストを実行しているので、開発者が自分で入力するのは、それらの外で何かを計測したときか、他人の数値が良すぎるように見えるときです。このスキルは、[Explain the Number 原則](https://github.com/cursor/plugins/blob/main/pstack/skills/principle-explain-the-number/SKILL.md)を実際に使える形にしたものです。

<a id="create-a-project-verification-skill"></a>
## プロジェクト用の検証スキルを作る

先ほどの UI の項目には、実は重要な前提が隠れています。エージェントには、アプリを操作するためのスクリプト化された手段が必要です。プロジェクトにすでにあれば問題ありません。なければ、次を実行します。

```text
/create-verification-skill
```

[`/create-verification-skill`](https://github.com/cursor/plugins/blob/main/pstack/skills/create-verification-skill/SKILL.md) が聞き取りを行う相手は、開発者ではなくリポジトリです。ユーザーが触れる部分、ローカルでのアプリの起動方法、アプリを操作できる手段（まず既存のハーネス、なければブラウザと CDP、PTY、素の HTTP）、振る舞いを証明する証拠、2 つのインスタンスを並べて動かせるかどうかを調べます。開発者に尋ねるのは、コードから答えが得られないことだけです。

このスキルは `.cursor/skills/verify-<app>/` を書き出します。中身は、Launch、Doctor、Drive、Evidence、Cleanup の各節を正確に備えたエージェント向けの手順書と、`features/` 以下の機能マップです。機能マップは、アプリが何をするのか、そして各機能が動くことをどの結果で証明するのかを索引にしたものです。スキルには[機能マップの作例](https://github.com/cursor/plugins/tree/main/pstack/skills/create-verification-skill/references/feature-map-example/)が含まれており、README の索引と、必須の 4 つの H2 を使った機能ごとのファイルで構成されています。引き渡す前に、生成器はスキルを一度エンドツーエンドで証明します。具体的には、起動、Doctor による確認、1 つの機能の操作、証拠の取得、後片付けを行います。この証明が失敗した場合は、出力を使わないでください。

これ以降、「アプリで検証する」は、このリポジトリでどのエージェントもセットアップの対話なしに実行できるステップになります。特定の形式で証拠が欲しい場合は、プロンプトでスキルを指定します。

```text
/poteto-mode build the bulk-archive action. use /verify-<app> to verify your changes and show me a video and screenshots as proof.
```

```text
/poteto-mode repro this with /verify-<app>. if it repros on main, fix it and show me a video as proof.
```

検証スキルが動くようになれば、[`/swarm`](https://github.com/cursor/plugins/blob/main/pstack/skills/swarm/SKILL.md) で全体の確認を機能マップの項目ごとに分割し、結果を集約できます。検証役の swarm は、十分な数のサンプルで性能の改善を確かめたり、PR を出荷する前にアプリをファジングしてリグレッションを探したりするのにも使えます。

検証スキルは、使い捨てではなくインフラとして扱ってください。リポジトリにコミットすれば、チームのすべての人とエージェントが同じ方法でアプリを操作できます。そのうえで、[てこを作ります（Build the Lever）](https://github.com/cursor/plugins/blob/main/pstack/skills/principle-build-the-lever/SKILL.md)。エージェントがアプリをクリックして回るための使い捨てのスクリプトを何度も書いているなら、代わりにスキルから呼び出せる小さな制御用 CLI を依頼します。エージェントが消費するトークンが減り、毎回の実行が再現可能になります。エージェントが使いこなせる CLI には、次の特徴があります。

- 薄いコマンドを多数用意するのではなく、それぞれが実際の作業を行う、組み合わせ可能な少数のコマンドを持つ
- 破壊的な操作にはすべて `--dry-run` オプションがある
- サブコマンドによって、機能を一度にではなく段階的に見せる
- エラーメッセージが、代わりに何をすべきかを示す
- `--help` の説明が充実している
- JSON のような機械可読な出力がある

ついでに、開発環境のセットアップも再現可能にしておきます。シードデータ、テストユーザー、そして毎回同じ方法で環境を立ち上げる 1 つのコマンドを用意します。

## 検証スキルを正確に保つ

アプリは変わり、機能マップは古くなっていきます。少なくとも 1 日に 1 回、理想的には誰も覚えておかなくて済むようスケジュールされたオートメーションから、次を実行します。

```text
/maintain-verification-skill
```

[`/maintain-verification-skill`](https://github.com/cursor/plugins/blob/main/pstack/skills/maintain-verification-skill/SKILL.md) は、生成されたスキルを監査します。まず機能ごとに 1 つの読み取り専用のソース読解役を並行して動かし、続いてマップにあるすべての機能を操作するライブ確認を 1 回行います。結果は必ず 3 つのうちの 1 つになります。`clean` は、網羅に抜けがなく出荷するものもないことを意味します。`changed` は、検証スキル自身のディレクトリに限定した、証明済みの修正を含む PR が 1 つあることを意味します。`blocked` は、障害となっているものを示します。このスキルはプロダクトのコードを決して編集しません。ライブ確認でプロダクトのリグレッションが見つかった場合は、ドキュメントで取り繕うのではなく、そのリグレッションを報告します。

## PR を開く

```text
/poteto-mode open the pr. small ordered commits, evidence in the description.
```

[Opening a PR プレイブック](https://github.com/cursor/plugins/blob/main/pstack/skills/poteto-mode/playbooks/opening-a-pr.md)は、worktree を使って作業し、変更を小さく順序立ったコミットにリベースし、diff を整え、文章に unslop をかけ、PR のリンクを返します。大きな PR を 1 つ作るより、範囲の狭い PR を 5 つ作るほうが良く、ブランチを膨らませ続けるより、後続の変更を積み重ねる（stack する）ほうが良い結果になります。

## Babysit で PR をマージ可能な状態まで進める

開いた PR には、すぐに障害が溜まり始めます。チェックが失敗し、レビュアーがコメントし、トランクが進みます。こうした対応は [Babysit プレイブック](https://github.com/cursor/plugins/blob/main/pstack/skills/poteto-mode/playbooks/babysit.md)に任せます。

```text
/poteto-mode babysit this pr. get it green.
```

Babysit は同梱のウォッチャーで PR を監視し、障害を、コンフリクト、レビューのスレッド、CI の順に処理します。判明している修正はすべて 1 回のプッシュにまとめるので、チェックの再実行は修正ごとではなく 1 回で済みます。コメントの選別は懐疑的に行います。人間もボットも、本物の指摘と雑音を同じリストに並べるからです。本物の指摘には修正を加え、雑音はその反証をスレッドに投稿したうえで却下します。状況だけを知りたい場合は、小さく尋ねれば、Babysit はループを始めずに答えます。

```text
/poteto-mode check on pr 123. anything outstanding?
```

Babysit は、マージ可能な状態で止まります。すべてのチェックが通っていても、マージは別の判断なので、決して自分ではマージしません。

## Shipping でスタックをマージする

チェックが通っていることと、安全であることは同じではありません。マージする準備ができたら、そう伝えます。

```text
/poteto-mode land the stack.
```

[Shipping プレイブック](https://github.com/cursor/plugins/blob/main/pstack/skills/poteto-mode/playbooks/shipping.md)は、何かのマージを予約する前に、各 PR を個別に検証します。PR ごとに新しいエージェントが 1 つ割り当てられて振る舞いを実際に証明し、変更を判定するエージェントは、決してその変更を書いたエージェントではありません。そのうえで Shipping は、下から連続して検証済みになっている範囲だけを、1 つずつマージします。既定では GitHub を使い、Origin の CLI が使える場合は Origin を使います。そして、連続を断ち切った最初の PR を報告します。未検証の PR の上にある検証済みの PR は待機させます。それをマージすると、その下にある未検証の部分まで取り込まれてしまうからです。

次へ: [寝ているあいだに作業を進める](./07-overnight-ja.md)
