# git-operation
## operation of git rebase

## はじめに
### 本書の位置づけ
- gitの操作を安全にできるように
  - 特にしくじりやすい rebaseを安全にできるように
    - rebaseをしながらgitになれるように

### 学習者の前提
- git add, commit, push, checkout等基本操作を身に着けていること

### 学習者のゴール
- 必修
  - gitの状態を知り、次になにをすればいいか判断できるようになる
    - git status
    - git log
  - rebaseをスムーズにできるようになる
    - git rebase --onto
    - 学習していくうちに他の必須操作も身につきます
- 任意
  - コミットをまとめれるようになる
    git rebase -i

## 本章
### gitの状態を知り、次に何をすればいいか判断できるようになる
- git status
  - gitになれるまではコマンドの直後に実施
  - rebase等、状態確認が特に重要なコマンド実施後はいちいち必ず実施
- git log
  - rebase等 conflictが発生しそうな作業の前にかならず実施
    - rebaseする前にhashキーを控えておくことでいつでももとに戻れる（後述）
    - HEADがわかる（HEADがどこのbranchも指していないとやばい）
  - git log時にやってほしいこと
    - メモ帳等にガツンとコピペ（エンターキー押下でどんどん履歴が出てくるが、自身がした過去のコミットより一つ前まであればまず安心。最低でも直近のはコピペること）
  - git logのhashがあればrebaseしようがmergeしようが以下のコマンドで必ず戻れる
    - git reset --hard [hash]

### rebaseをスムーズにできるようになる
base    ◯-◯-◯
            ＼
topic         ◎-◎
       ↓ イメージ
base    ◯-◯-◯
               ＼
topic            ◎-◎

- 子枝の付け根から先をちょん切って、親幹の先頭に付け替えることが多い

- rebaseを勧める理由
  - グラフの履歴が見やすい
  - 一度mergeをすると、rebase時にコンフリクトの解消が面倒臭い
   - 他者のrebaseを邪魔しないですむ

#### rebase実践（コンフリクトが発生しない場合）
##### gitの状態は以下として説明する
base  ◯-◯-◯
          ＼
topic       ◎-◎

##### rebaseする
- baseのgit log 採取
  - git checkout base
  - git pull
  - git log
commit aaaaabbbbbcccc12 (HEAD -> base ・・・・)
(略)
commit xxxyyyyzzzz11233
(略)

- topicのgit log 採取
  - git checkout topic
  - git pull
  - git log
commit aaaaaaaaaaaaaaaa (HEAD -> topic ・・・・)
(略)
commit bbbbbbbbbbbbbbbb
(略)
commit cccccccccccccccc
(略)
commit xxxyyyyzzzz11233
(略)

- 上記で一致している xxxyyyyzzzz11233 が切る場所

- topic側にチェックアウト
  - git checkout topic
- rebase実施
  - git rebase --onto base xxxyyyyzzzz11233
- originとのdiffを確認して、自分の変更分以外のみ変更されていることを確認
  - git diff --stat topic origin/topic 等
- リポジトリにpush
  - git push --force-with-lease
- gitgraph等で以下のようになっていることを確認
base  ◯-◯-◯
              ＼
topic           ◎-◎-◎

#### rebase実践（コンフリクトが発生した場合）
##### gitの状態は以下として説明する
base  ◯-◯-◯
          ＼
topic       ◎-◎-◎

- git logを採取して切り取るhashを取得するまではコンフリクトが発生しないパターンと同じ
- topic側にチェックアウト
  - git checkout topic
- rebase実施（ここでコンフリクトが発生すると。。。）
  - git rebase --onto base xxxyyyyzzzz11233
    - CONFLICTとかでるので、、、、
- 落ち着いて巻き戻し
  - git rebase --abort
  - git status
- 対話式rebaseにしてsquashする（コンフリクト解消がコミット回数分になるため）
  - git rebase -i --onto base xxxyyyyzzzz11233
    - vim状態になるので、一番上以外のpickをsに修正
    - :wq
  - squashは終わったがまだコンフリクト状態
    - VS Code等でコンフリクトを解消する
    - git status
    - 問題なければ、git add コンフリクトしているファイル
    - git status
    - git commit -m 'メッセージ'
    - git status
    - 多分 rebaseを終わらせろと出ているので
      - git rebase --contine
    - git status
  - originとのdiffを確認して、自分の変更分以外のみ変更されていることを確認
    - git diff --stat topic origin/topic 等
  - リポジトリにpush
    - git push --force-with-lease
  - gitgraph等で以下のようになっていることを確認
base  ◯-◯-◯
              ＼
topic           ◎'

### コミットを纏めれるようになる
- コミットが増えすぎた場合に纏める
- 複数人で作業している際はコミットを纏めて問題ないか確認すること
  - force pushする為。他のメンバはrebaseしないといけなくなる。（これは通常rebaseにも言える）
base  ◯
        ＼
topic     ◎-◎-◎-◎-◎
       ↓ イメージ
base  ◯
        ＼
topic     ◎'

##### gitの状態は以下として説明する
base  ◯
        ＼
topic     ◎-◎-◎-◎-◎

- topicのgit log 採取
  - git checkout topic
  - git pull
  - git log
commit aaaaaaaaaaaaaaaa (HEAD -> topic ・・・・)
(略)
commit bbbbbbbbbbbbbbbb
(略)
commit cccccccccccccccc
(略)
commit dddddddddddddddd
(略)
commit eeeeeeeeeeeeeeee
(略)
commit xxxyyyyzzzz11233  <= こいつが切るとこ（自分のコミットの１個前）
(略)

- git rebase -i xxxyyyyzzzz11233
- 対話モードでvimになるので一番上以外, pick を s にする(終わったら:wq)

pick aaaaaaa コミットメッセージA
pick bbbbbbb コミットメッセージB
pick ccccccc コミットメッセージC
pick ddddddd コミットメッセージD
pick eeeeeee コミットメッセージE
       ↓ 
pick aaaaaaa コミットメッセージA
s bbbbbbb コミットメッセージB
s ccccccc コミットメッセージC
s ddddddd コミットメッセージD
s eeeeeee コミットメッセージE
- コメントを編集できますが今回は割愛します

- git status
  - 正常確認
- git log 
  - 実施前に比べてコミットが少なくなっているはず

- gitgraph等で以下のようになっていることを確認
base  ◯
        ＼
topic     ◎'
