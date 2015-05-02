===========================================================
gitの練習帳
===========================================================

-----------------------------------------------------------
HEADとは？
-----------------------------------------------------------

実用Gitに記載されていなかったので、以下から::

  http://git-scm.com/book/ja/v1/Git-%E3%81%AE%E3%83%96%E3%83%A9%E3%83%B3%E3%83%81%E6%A9%9F%E8%83%BD-%E3%83%96%E3%83%A9%E3%83%B3%E3%83%81%E3%81%A8%E3%81%AF

抜粋：Git では、HEAD はあなたが作業しているローカルブランチへのポインタとなります。

別のブログでは、最新のコミットに対するハッシュ値の別名です。とも。

-----------------------------------------------------------
git log
-----------------------------------------------------------

git logの使いかた例::

 miyakz@lily:~/github_repos/documents$ git log -1 --pretty=oneline HEAD
 e605ecaf08b4d584ce82c32aa9b13426d225a047 update linux tips
 miyakz@lily:~/github_repos/documents$ 
 
 miyakz@lily:~/github_repos/documents$ git log -1 --pretty=oneline 
 e605ecaf08b4d584ce82c32aa9b13426d225a047 update linux tips
 miyakz@lily:~/github_repos/documents$ 

コミット番号の上４桁を指定してもＯＫ::

 miyakz@lily:~/github_repos/documents$ git log -1 --pretty=oneline fa9e
 fa9e702d680e41a364ce2d20756143f0d00f6675 updated
 miyakz@lily:~/github_repos/documents$ 

コミットの履歴をマージグラフとともにいい感じで見たい場合::

  git log --graph --oneline --decorate

-----------------------------------------------------------
ORIG_HEADとその効用
-----------------------------------------------------------

mergeやリセットのようなオペレーションでは、HEADが新しい値にされるまえに、
前のHEADの値をORIG_HEADに書き込んでいる。以下、間違ってmergeしたものを
元に戻す例::

  cd /tmp/
  mkdir ex1_orig_head
  cd ex1_orig_head 
  git init 
  git branch
  #masterブランチを作る。ここでdoc1というファイルを作成
  git checkout -b master
  git branch
  touch doc1
  echo "first doc1" > doc1 
  git add doc1 
  git commit -m "first doc1" 
  #topicブランチを作って、そこでdoc1を更新する。
  git checkout -b topic-x
  echo "second" >> doc1 
  git commit -m "second" -a
  #masterブランチに戻ってtopicをマージする。あ！間違った！
  git checkout master
  cat doc1 
  git merge topic-x

この時、HEADとORIG_HEADの値はそれぞれ、以下のようになっている::

  miyakz@lily:/tmp/ex1_orig_head$ cat .git/ORIG_HEAD 
  29c00303d0e91e94bc8d74bf3d39c7ec0b8798c0
  miyakz@lily:/tmp/ex1_orig_head$ cat .git/HEAD 
  ref: refs/heads/master
  miyakz@lily:/tmp/ex1_orig_head$ 

29c0はこんな感じ。つまり、ORIG_HEADはmerge前のコミットを指しているね::

  miyakz@lily:/tmp/ex1_orig_head$ git show 29c0 
  commit 29c00303d0e91e94bc8d74bf3d39c7ec0b8798c0
  Author: kazuhiro MIYASHITA <miyakz1192@gmail.com>
  Date:   Sat May 2 11:43:44 2015 +0900
  
      first doc1
  
      diff --git a/doc1 b/doc1
      new file mode 100644
      index 0000000..e743584
      --- /dev/null
      +++ b/doc1
      @@ -0,0 +1 @@
      +first doc1
      miyakz@lily:/tmp/ex1_orig_head$ 

doc1ファイルの内容はmergeした状態になっているので、::

 miyakz@lily:/tmp/ex1_orig_head$ cat doc1 
 first doc1
 second
 miyakz@lily:/tmp/ex1_orig_head$ 

ファイルも含めて元に戻したい場合は以下を実行すれば良い::

 miyakz@lily:/tmp/ex1_orig_head$ git reset --hard ORIG_HEAD
 HEAD is now at 29c0030 first doc1
 miyakz@lily:/tmp/ex1_orig_head$ 


参考ＵＲＬ::

  http://labs.timedia.co.jp/2011/01/git-undo-merge.html

-----------------------------------------------------------
MERGE_HEAD
-----------------------------------------------------------

ちょっとわからない。以下の参照ＵＲＬではmegrgeしてきた時に
MERGE_HEADが作られる（書き込まれる）はずだが、私の環境では存在しなかった::

 MERGE_HEAD
 $ git checkout master
 $ git merge <branch-name>
 このようなコマンドを実行して、ローカルブランチ<branch-name>の変更をmasterブランチにマージしたとします。
 このときのローカルブランチ<branch-name>の最新のコミットに対するハッシュ値の別名です。

参考URL:http://qiita.com/t-mochizuki/items/347cba461fd570bca03c

-----------------------------------------------------------
FETCH_HEAD
-----------------------------------------------------------

要するに、リモートレポジトリからfetchした時に、フェッチしたブランチの先頭を
保持している変数。フェッチ操作の直後だけ有効。

以下の参照URLの説明が大変わかりやすい。以下抜粋::

pullを実行すると、リモートリポジトリの内容のマージが自動的に行われてしまいます。しかし、単にリモートリポジトリの内容を確認したいだけの時はマージをしたくない場合もあります。そのような時はfetchを使用します。

fetchを実行すると、リモートリポジトリの最新の履歴の取得だけを行うことができます。取得したコミットは、名前の無いブランチとして取り込まれます。このブランチはFETCH_HEADという名前でチェックアウトすることができます。

例えば、ローカルリポジトリとリモートリポジトリのoriginのそれぞれに、Bから進んだコミットがある状態でfetchを行うと、下の図のような履歴になります。

ローカルリポジトリとリモートリポジトリのoriginのそれぞれに、Bから進んだコミットがある状態でfetch

この状態から、リモートリポジトリの内容をローカルリポジトリのmasterに統合する場合は、FETCH_HEADをマージするか、改めてpullを実行します。

FETCH_HEADをマージ

マージを行うと、pullの時と同じ履歴になります。
実は、pullというのは内部でfetch + mergeをしているからです 

参考URL:http://www.backlog.jp/git-guide/stepup/stepup3_2.html

-----------------------------------------------------------
相対的なコミット名
-----------------------------------------------------------

実用gitのP74のような３つブランチが存在して、それが１つに
マージされる様子を作ってみる。::

   branch1    /---- c4 -- c5 \
   master   c1  --  c2 -- c3 +-
   branch2    \ --  c6 -- c7 /

スクリプトは以下の通り::

  mkdir ex2_relative_commit_name
  cd ex2_relative_commit_name/
  git init
  #masterブランチの作成
  git checkout -b master
  git branch
  echo c1 > c1
  git add c1
  git commit -m "c1" -a
  git branch
  #branch1の作成と関連するコミット
  git checkout -b branch1
  echo c4 > c4
  git add c4
  git commit -m "c4" -a
  echo c5 > c5
  git add c5
  git commit -m "c5" -a
  git checkout master
  #branc2をmasterから作る
  git checkout -b branch2
  echo c6 > c6
  git add c6
  git commit -m "c6" -a
  echo c7 > c7
  git add c7
  git commit -m "c7" -a
  #masterに戻り、続きのコミットを作成する
  git checkout master
  echo c2 > c2
  git add c2
  git commit -m "c2" -a
  echo c3 > c3
  git add c3
  git commit -m "c3" -a

ファイルの構成は以下のようになる::

  miyakz@lily:/tmp/ex2_relative_commit_name$ git checkout branch1
  Switched to branch 'branch1'
  miyakz@lily:/tmp/ex2_relative_commit_name$ ls
  c1  c4  c5
  miyakz@lily:/tmp/ex2_relative_commit_name$ git checkout branch2
  Switched to branch 'branch2'
  miyakz@lily:/tmp/ex2_relative_commit_name$ ls
  c1  c6  c7
  miyakz@lily:/tmp/ex2_relative_commit_name$ git checkout master
  Switched to branch 'master'
  miyakz@lily:/tmp/ex2_relative_commit_name$ ls
  c1  c2  c3
  miyakz@lily:/tmp/ex2_relative_commit_name$ 

次にmaster1にbranch1とbranch2をmergeする。::

  miyakz@lily:/tmp/ex2_relative_commit_name$ git checkout master^C
  miyakz@lily:/tmp/ex2_relative_commit_name$ git merge branch1 branch2
  Trying simple merge with branch1
  Trying simple merge with branch2
  Merge made by the 'octopus' strategy.
   c4 | 1 +
   c5 | 1 +
   c6 | 1 +
   c7 | 1 +
   4 files changed, 4 insertions(+)
   create mode 100644 c4
   create mode 100644 c5
   create mode 100644 c6
   create mode 100644 c7
  miyakz@lily:/tmp/ex2_relative_commit_name$ 

グラフにすると以下のような感じ::

  miyakz@lily:/tmp/ex2_relative_commit_name$ git log --graph --oneline --decorate
  *-.   a5bf01e (HEAD, master) Merge branches 'branch1' and 'branch2'
  |\ \  
  | | * 0e97f86 (branch2) c7
  | | * 98f3092 c6
  | * | 7b0e0a2 (branch1) c5
  | * | c4c5a4d c4
  | |/  
  * | f943893 c3
  * | 7ff5106 c2
  |/  
  * 8241c85 c1
  miyakz@lily:/tmp/ex2_relative_commit_name$

a5bf01eから見た親はについてmaster^1 ... ^3で見てみる。以下のように、c3,c5,c7が見えている::

  miyakz@lily:/tmp/ex2_relative_commit_name$ git show master^1 --pretty=oneline --abbrev-commit
  f943893 c3
  diff --git a/c3 b/c3
  new file mode 100644
  index 0000000..0771aea
  --- /dev/null
  +++ b/c3
  @@ -0,0 +1 @@
  +c3
  miyakz@lily:/tmp/ex2_relative_commit_name$ git show master^2 --pretty=oneline --abbrev-commit
  7b0e0a2 c5
  diff --git a/c5 b/c5
  new file mode 100644
  index 0000000..c363571
  --- /dev/null
  +++ b/c5
  @@ -0,0 +1 @@
  +c5
  miyakz@lily:/tmp/ex2_relative_commit_name$ git show master^3 --pretty=oneline --abbrev-commit
  0e97f86 c7
  diff --git a/c7 b/c7
  new file mode 100644
  index 0000000..20be687
  --- /dev/null
  +++ b/c7
  @@ -0,0 +1 @@
  +c7
  miyakz@lily:/tmp/ex2_relative_commit_name$ 

^4を指定すると、親がないためエラーになる。::

 miyakz@lily:/tmp/ex2_relative_commit_name$ git show master^4 --pretty=oneline --abbrev-commit
 fatal: ambiguous argument 'master^4': unknown revision or path not in the working tree.
 Use '--' to separate paths from revisions, like this:
 'git <command> [<revision>...] -- [<file>...]'
 miyakz@lily:/tmp/ex2_relative_commit_name$ 

チルダを指定して、親の親を見てみる。それぞれ、c2,c4,c6が出ており、親の親が見れることがわかる::

  miyakz@lily:/tmp/ex2_relative_commit_name$ git show master^1~1 --pretty=oneline --abbrev-commit
  7ff5106 c2
  diff --git a/c2 b/c2
  new file mode 100644
  index 0000000..16f9ec0
  --- /dev/null
  +++ b/c2
  @@ -0,0 +1 @@
  +c2
  miyakz@lily:/tmp/ex2_relative_commit_name$ git show master^2~1 --pretty=oneline --abbrev-commit
  c4c5a4d c4
  diff --git a/c4 b/c4
  new file mode 100644
  index 0000000..a103f67
  --- /dev/null
  +++ b/c4
  @@ -0,0 +1 @@
  +c4
  miyakz@lily:/tmp/ex2_relative_commit_name$ git show master^3~1 --pretty=oneline --abbrev-commit
  98f3092 c6
  diff --git a/c6 b/c6
  new file mode 100644
  index 0000000..86a7165
  --- /dev/null
  +++ b/c6
  @@ -0,0 +1 @@
  +c6
  miyakz@lily:/tmp/ex2_relative_commit_name$ 

-----------------------------------------------------------
集合の引き算(..)
-----------------------------------------------------------

実用GitのP84の図6-11のようなコミット履歴を作ってみる。コミット履歴を作るために
こんなscriptを一時的に作ってみる。パスを通す::

  miyakz@lily:/tmp/ex3_symmetric_difference$ cat /home/miyakz/bin/node.sh
  #!/bin/bash
  
  node_name=$1
  
  echo ${node_name} > ${node_name}
  git add ${node_name}
  git commit -m "${node_name}" -a
  
  miyakz@lily:/tmp/ex3_symmetric_difference$ 

構造を作ってみる。::

  mkdir ex3_symmetric_difference
  cd ex3_symmetric_difference
  git init
  git checkout -b master
  node.sh u
  git checkout -b topic
  ls
  node.sh a
  git checkout master
  node.sh v
  git checkout topic
  git merge master
  node.sh c
  node.sh d
  git checkout master
  node.sh w
  node.sh x
  node.sh y
  node.sh z

コミットグラフはこんな感じ::

  miyakz@lily:/tmp/ex3_symmetric_difference$ git log --graph --oneline --decorate --all
  * d3f1b8f (HEAD, master) z
  * 4bd773e y
  * 063cf41 x
  * 8b889be w
  | * 01333a1 (topic) d
  | * 003a7ef c
  | *   b6bba83 Merge branch 'master' into topic
  | |\  
  | |/  
  |/|   
  * | 2fca0db v
  | * 3db3d44 a
  |/  
  * dfa9329 u
  miyakz@lily:/tmp/ex3_symmetric_difference$

差をとってみる。これは集合の差。集合masterから、集合topicを引いたもの。w,x,y,zが出力されるはず::

  miyakz@lily:/tmp/ex3_symmetric_difference$ git log topic..master
  commit d3f1b8f36b5672159a3ca78201d4eec9cab91ef0
  Author: kazuhiro MIYASHITA <miyakz1192@gmail.com>
  Date:   Sat May 2 17:56:12 2015 +0900
  
      z
  
  commit 4bd773e5a9e3e7b8afc5812f6569050eae79418a
  Author: kazuhiro MIYASHITA <miyakz1192@gmail.com>
  Date:   Sat May 2 17:56:12 2015 +0900
  
      y
  
  commit 063cf41e19120bec8cbde3a1d2743e082c69f858
  Author: kazuhiro MIYASHITA <miyakz1192@gmail.com>
  Date:   Sat May 2 17:56:11 2015 +0900
  
      x
  
  commit 8b889be605d52720162aec16bf2a881c65178a04
  Author: kazuhiro MIYASHITA <miyakz1192@gmail.com>
  Date:   Sat May 2 17:56:10 2015 +0900
  
      w
  miyakz@lily:/tmp/ex3_symmetric_difference$ 

予想通り。次に対称差(git log topic...master)をとってみる。topicとmasterの和から、topicとmasterの積を引いたもの。つまり、a,c,d,w,x,y,zになる。::

  commit d3f1b8f36b5672159a3ca78201d4eec9cab91ef0
  Author: kazuhiro MIYASHITA <miyakz1192@gmail.com>
  Date:   Sat May 2 17:56:12 2015 +0900
  
      z
  
  commit 4bd773e5a9e3e7b8afc5812f6569050eae79418a
  Author: kazuhiro MIYASHITA <miyakz1192@gmail.com>
  Date:   Sat May 2 17:56:12 2015 +0900
  
      y
  
  commit 063cf41e19120bec8cbde3a1d2743e082c69f858
  Author: kazuhiro MIYASHITA <miyakz1192@gmail.com>
  Date:   Sat May 2 17:56:11 2015 +0900
  
      x
  
  commit 8b889be605d52720162aec16bf2a881c65178a04
  Author: kazuhiro MIYASHITA <miyakz1192@gmail.com>
  Date:   Sat May 2 17:56:10 2015 +0900
  
      w
  
  commit 01333a1c10c4cf5f34030b943d72f473ea34801c
  Author: kazuhiro MIYASHITA <miyakz1192@gmail.com>
  Date:   Sat May 2 17:55:22 2015 +0900
  
      d
  
  commit 003a7ef96d3572122d7139634d86621e6b939a6c
  Author: kazuhiro MIYASHITA <miyakz1192@gmail.com>
  Date:   Sat May 2 17:55:21 2015 +0900
  
      c
  
  commit b6bba8366b61ee1f585a3f982287c270495895e1
  Merge: 3db3d44 2fca0db
  Author: kazuhiro MIYASHITA <miyakz1192@gmail.com>
  Date:   Sat May 2 17:55:09 2015 +0900
  
      Merge branch 'master' into topic
  
  commit 3db3d440448f72da802464ef625aac7a0f2af71f
  Author: kazuhiro MIYASHITA <miyakz1192@gmail.com>
  Date:   Sat May 2 17:54:53 2015 +0900
  
      a












