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
git logで指定する、集合の引き算(..)と、対称差(...)
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


-----------------------------------------------------------
切り離されたHEADのブランチと任意のコミットからのブランチ
-----------------------------------------------------------

ブランチの先頭でないコミットをチェックアウトすると、detached HEADと
呼ばれる無名ブランチができる。::

  mkdir ex4_detached_head
  cd ex4_detached_head/
  git init
  git checkout -b master
  hode.sh a
  node.sh b
  node.sh c

こんな感じのグラフになる。::

  miyakz@lily:/tmp/ex4_detached_head$ git log --graph --oneline --decorate --all
  * fd54607 (HEAD, master) c
  * fe69331 b
  * d1cfe80 a
  miyakz@lily:/tmp/ex4_detached_head$ 

"b"のところからcheckoutしてみる。::

  miyakz@lily:/tmp/ex4_detached_head$ git checkout fe69331
  Note: checking out 'fe69331'.
  
  You are in 'detached HEAD' state. You can look around, make experimental
  changes and commit them, and you can discard any commits you make in this
  state without impacting any branches by performing another checkout.
  
  If you want to create a new branch to retain commits you create, you may
  do so (now or later) by using -b with the checkout command again. Example:
  
    git checkout -b new_branch_name
  
  HEAD is now at fe69331... b
  miyakz@lily:/tmp/ex4_detached_head$ 

HEADの様子はこんな感じ::

  miyakz@lily:/tmp/ex4_detached_head$ git branch
  * (detached from fe69331)
    master
  miyakz@lily:/tmp/ex4_detached_head$ git log --graph --oneline --decorate --all
  * fd54607 (master) c
  * fe69331 (HEAD) b
  * d1cfe80 a
  miyakz@lily:/tmp/ex4_detached_head$ 

名前をつけてみる::

  miyakz@lily:/tmp/ex4_detached_head$ git checkout -b from_b
  Switched to a new branch 'from_b'
  miyakz@lily:/tmp/ex4_detached_head$ git branch
  * from_b
    master
  miyakz@lily:/tmp/ex4_detached_head$ 
  miyakz@lily:/tmp/ex4_detached_head$ git log --graph --oneline --decorate --all
  * fd54607 (master) c
  * fe69331 (HEAD, from_b) b
  * d1cfe80 a
  miyakz@lily:/tmp/ex4_detached_head$ 

dというnodeを増やしてみた。::

  miyakz@lily:/tmp/ex4_detached_head$ node.sh d
  [from_b 678e533] d
   1 file changed, 1 insertion(+)
   create mode 100644 d
  miyakz@lily:/tmp/ex4_detached_head$ git log --graph --oneline --decorate --all
  * 678e533 (HEAD, from_b) d
  | * fd54607 (master) c
  |/  
  * fe69331 b
  * d1cfe80 a
  miyakz@lily:/tmp/ex4_detached_head$ 


-----------------------------------------------------------
git diffとコミット範囲
-----------------------------------------------------------

実験の構造を作ってみる。::

  mkdir ex5_git_diff
  cd ex5_git_diff 
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

  miyakz@lily:/tmp/ex5_git_diff$ git log --graph --oneline --decorate --all
  * debb7b4 (HEAD, master) z
  * 69da2b0 y
  * 6e3faa2 x
  * 75ef0e0 w
  | * 1406d85 (topic) d
  | * d517715 c
  | *   b54782a Merge branch 'master' into topic
  | |\  
  | |/  
  |/|   
  * | 8d81df9 v
  | * 3ec124b a
  |/  
  * 5e9bce5 u
  miyakz@lily:/tmp/ex5_git_diff$ 

ここでmasterとtopicのdiffをとってみる。topicになるためには、masterにどのような変更を加えればよいかという観点での出力でもある。::

  miyakz@lily:/tmp/ex5_git_diff$ git diff master..topic
  diff --git a/a b/a
  new file mode 100644
  index 0000000..7898192
  --- /dev/null
  +++ b/a
  @@ -0,0 +1 @@
  +a
  diff --git a/c b/c
  new file mode 100644
  index 0000000..f2ad6c7
  --- /dev/null
  +++ b/c
  @@ -0,0 +1 @@
  +c
  diff --git a/d b/d
  new file mode 100644
  index 0000000..4bcfe98
  --- /dev/null
  +++ b/d
  @@ -0,0 +1 @@
  +d
  diff --git a/w b/w
  deleted file mode 100644
  index e556b83..0000000
  --- a/w
  +++ /dev/null
  @@ -1 +0,0 @@
  -w
  diff --git a/x b/x
  deleted file mode 100644
  index 587be6b..0000000
  --- a/x
  +++ /dev/null
  @@ -1 +0,0 @@
  -x
  diff --git a/y b/y
  deleted file mode 100644
  index 975fbec..0000000
  --- a/y
  +++ /dev/null
  @@ -1 +0,0 @@
  -y
  diff --git a/z b/z
  deleted file mode 100644
  index b680253..0000000
  --- a/z
  +++ /dev/null
  @@ -1 +0,0 @@
  -z
  miyakz@lily:/tmp/ex5_git_diff$ 

たしかに、a,c,d,を加え、w,x,y,zを除けばtopicブランチになることができる。
指定の順番を逆にすると、予想通り、反対の結果が出てくる。::

  miyakz@lily:/tmp/ex5_git_diff$ git diff topic..master
  diff --git a/a b/a
  deleted file mode 100644
  index 7898192..0000000
  --- a/a
  +++ /dev/null
  @@ -1 +0,0 @@
  -a
  diff --git a/c b/c
  deleted file mode 100644
  index f2ad6c7..0000000
  --- a/c
  +++ /dev/null
  @@ -1 +0,0 @@
  -c
  diff --git a/d b/d
  deleted file mode 100644
  index 4bcfe98..0000000
  --- a/d
  +++ /dev/null
  @@ -1 +0,0 @@
  -d
  diff --git a/w b/w
  new file mode 100644
  index 0000000..e556b83
  --- /dev/null
  +++ b/w
  @@ -0,0 +1 @@
  +w
  diff --git a/x b/x
  new file mode 100644
  index 0000000..587be6b
  --- /dev/null
  +++ b/x
  @@ -0,0 +1 @@
  +x
  diff --git a/y b/y
  new file mode 100644
  index 0000000..975fbec
  --- /dev/null
  +++ b/y
  @@ -0,0 +1 @@
  +y
  diff --git a/z b/z
  new file mode 100644
  index 0000000..b680253
  --- /dev/null
  +++ b/z
  @@ -0,0 +1 @@
  +z
  miyakz@lily:/tmp/ex5_git_diff$ 

対称差では以下のようになる。git diff(git diff commit1...commit2)の対称差ではcommit2とcommit1とcommit2の共通の祖先との差分を表示する。この例では、topicと、masterとtopicの共通の祖先(この場合は8d81df9)の差分を表示する。8d81df9はuとvを持っており、topicはu,a,v,c,dを持っている。8d81df9がtopicになるためには、8d81df9に単純にa,c,dを加えればよい::

  miyakz@lily:/tmp/ex5_git_diff$ git diff master...topic
  diff --git a/a b/a
  new file mode 100644
  index 0000000..7898192
  --- /dev/null
  +++ b/a
  @@ -0,0 +1 @@
  +a
  diff --git a/c b/c
  new file mode 100644
  index 0000000..f2ad6c7
  --- /dev/null
  +++ b/c
  @@ -0,0 +1 @@
  +c
  diff --git a/d b/d
  new file mode 100644
  index 0000000..4bcfe98
  --- /dev/null
  +++ b/d
  @@ -0,0 +1 @@
  +d
  miyakz@lily:/tmp/ex5_git_diff$ 

-----------------------------------------------------------
コミットの変更
-----------------------------------------------------------

この演習で使う構成の作成
-------------------------

  mkdir ex6_modify_commits
  cd ex6_modify_commits
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


  * debb7b4 (HEAD, master) z
  * 69da2b0 y
  * 6e3faa2 x
  * 75ef0e0 w
  | * 1406d85 (topic) d
  | * d517715 c
  | *   b54782a Merge branch 'master' into topic
  | |\  
  | |/  
  |/|   
  * | 8d81df9 v
  | * 3ec124b a
  |/  
  * 5e9bce5 u

cherry-pick
--------------

masterにtopicの修正(c d517715)をcherry pickする。::

  miyakz@lily:/tmp/ex5_git_diff$ ls
  u  v  w  x  y  z
  miyakz@lily:/tmp/ex5_git_diff$ git log --graph --oneline --decorate --all
  * debb7b4 (HEAD, master) z
  * 69da2b0 y
  * 6e3faa2 x
  * 75ef0e0 w
  | * 1406d85 (topic) d
  | * d517715 c
  | *   b54782a Merge branch 'master' into topic
  | |\  
  | |/  
  |/|   
  * | 8d81df9 v
  | * 3ec124b a
  |/  
  * 5e9bce5 u
  miyakz@lily:/tmp/ex5_git_diff$ git cherry-pick d517715
  [master e535492] c
   Date: Sat May 2 22:28:13 2015 +0900
   1 file changed, 1 insertion(+)
   create mode 100644 c
  miyakz@lily:/tmp/ex5_git_diff$ ls
  c  u  v  w  x  y  z
  miyakz@lily:/tmp/ex5_git_diff$ git log --graph --oneline --decorate --all
  * e535492 (HEAD, master) c
  * debb7b4 z
  * 69da2b0 y
  * 6e3faa2 x
  * 75ef0e0 w
  | * 1406d85 (topic) d
  | * d517715 c
  | *   b54782a Merge branch 'master' into topic
  | |\  
  | |/  
  |/|   
  * | 8d81df9 v
  | * 3ec124b a
  |/  
  * 5e9bce5 u
  miyakz@lily:/tmp/ex5_git_diff$ 

さて、このレポジトリに全く関係のないレポジトリのコミットをcherry pickできるのだろうか、実験してみる。

まず、neutronレポジトリをremoteとして追加する。::

  miyakz@lily:/tmp/ex5_git_diff$ git remote add my_neutron https://github.com/miyakz1192/neutron.git
  miyakz@lily:/tmp/ex5_git_diff$ git remote -v
  my_neutron  https://github.com/miyakz1192/neutron.git (fetch)
  my_neutron  https://github.com/miyakz1192/neutron.git (push)
  miyakz@lily:/tmp/ex5_git_diff$ 

次にmy_neutronからfetchしてくる。::

  miyakz@lily:/tmp/ex5_git_diff$ git fetch my_neutron
  miyakz@lily:/tmp/ex5_git_diff$ git checkout stable/juno

checkoutしただけで、ブランチが結合したように見える。。。が？::

  * e535492 (HEAD, master) c
  * debb7b4 z
  * 69da2b0 y
  * 6e3faa2 x
  * 75ef0e0 w
  | * 1406d85 (topic) d
  | * d517715 c
  | *   b54782a Merge branch 'master' into topic
  | |\  
  | |/  
  |/|   
  * | 8d81df9 v
  | * 3ec124b a
  |/  
  * 5e9bce5 u
  * a5ffaaa (my_neutron/feature_audit) neutron audit log added
  * e6d7c45 (my_neutron/stable/icehouse) Updated from global requirements

ただ、そう見えただけであった。::

  miyakz@lily:/tmp/ex5_git_diff$ git show HEAD~6
  commit 5e9bce58e4f1198a23be42dd439a4e30340defcd
  Author: kazuhiro MIYASHITA <miyakz1192@gmail.com>
  Date:   Sat May 2 22:27:40 2015 +0900
  
      u
  
  diff --git a/u b/u
  new file mode 100644
  index 0000000..4ae8ef0
  --- /dev/null
  +++ b/u
  @@ -0,0 +1 @@
  +u
  miyakz@lily:/tmp/ex5_git_diff$ git show HEAD~7
  fatal: ambiguous argument 'HEAD~7': unknown revision or path not in the working tree.
  Use '--' to separate paths from revisions, like this:
  'git <command> [<revision>...] -- [<file>...]'
  miyakz@lily:/tmp/ex5_git_diff$ 

では試しに、stable/icehouseからmasterに対して、なにか、cherry pickしてくる。ためしに、自分のオリジナル修正(a5ffaaa)を持ってくる。エラーは起きたが、ファイルは存在する様子::

  miyakz@lily:/tmp/ex5_git_diff$ git cherry-pick a5ffaaa
  error: could not apply a5ffaaa... neutron audit log added
  ヒント: after resolving the conflicts, mark the corrected paths
  ヒント: with 'git add <paths>' or 'git rm <paths>'
  ヒント: and commit the result with 'git commit'
  miyakz@lily:/tmp/ex5_git_diff$ git status
  On branch master
  You are currently cherry-picking commit a5ffaaa.
    (fix conflicts and run "git cherry-pick --continue")
    (use "git cherry-pick --abort" to cancel the cherry-pick operation)
  
  Unmerged paths:
    (use "git add/rm <file>..." as appropriate to mark resolution)
  
    deleted by us:   neutron/api/v2/resource.py
  
  no changes added to commit (use "git add" and/or "git commit -a")
  miyakz@lily:/tmp/ex5_git_diff$ ls
  c  neutron  u  v  w  x  y  z
  miyakz@lily:/tmp/ex5_git_diff$ ls neutron/api/v2/resource.py 
  neutron/api/v2/resource.py
  miyakz@lily:/tmp/ex5_git_diff$ less neutron/api/v2/resource.py 
  miyakz@lily:/tmp/ex5_git_diff$ 

gitはconflictsを起こしていると言っているが、特にコンフリクトは起こしていないので、指示どおりに、git cherry-pick --continue を実行する。::

  miyakz@lily:/tmp/ex5_git_diff$ git cherry-pick --continue
  U neutron/api/v2/resource.py
  error: commit is not possible because you have unmerged files.
  ヒント: Fix them up in the work tree, and then use 'git add/rm <file>'
  ヒント: as appropriate to mark resolution and make a commit, or use
  ヒント: 'git commit -a'.
  fatal: Exiting because of an unresolved conflict.
  miyakz@lily:/tmp/ex5_git_diff$


指示通りに、git addでステージしてやる。::

  miyakz@lily:/tmp/ex5_git_diff$ git add neutron/api/v2/resource.py 
  miyakz@lily:/tmp/ex5_git_diff$ less neutron/api/v2/resource.py ^C
  miyakz@lily:/tmp/ex5_git_diff$ git status
  On branch master
  You are currently cherry-picking commit a5ffaaa.
    (all conflicts fixed: run "git cherry-pick --continue")
    (use "git cherry-pick --abort" to cancel the cherry-pick operation)
  
  Changes to be committed:
  
    new file:   neutron/api/v2/resource.py
  
  miyakz@lily:/tmp/ex5_git_diff$ 

あと一息、commit して、graphを見てやる。::

  * 7d0fccb (HEAD, master) cherry-pick from my_neutron
  * e535492 c
  * debb7b4 z
  * 69da2b0 y
  * 6e3faa2 x
  * 75ef0e0 w
  | * 1406d85 (topic) d
  | * d517715 c
  | *   b54782a Merge branch 'master' into topic
  | |\  
  | |/  
  |/|   
  * | 8d81df9 v
  | * 3ec124b a
  |/  
  * 5e9bce5 u

直前のコミットの取り消し(git commit --amend)
---------------------------------------------

上記の状態でnode qqqを追加する。::

  * 4145854 (HEAD, master) qqq
  * 7d0fccb cherry-pick from my_neutron
  * e535492 c
  * debb7b4 z
  * 69da2b0 y
  * 6e3faa2 x
  * 75ef0e0 w
  | * 1406d85 (topic) d
  | * d517715 c
  | *   b54782a Merge branch 'master' into topic
  | |\  
  | |/  
  |/|   
  * | 8d81df9 v
  | * 3ec124b a
  |/  
  * 5e9bce5 u

次に、qqqの中身をzzzに変更するcommitをする。::

  miyakz@lily:/tmp/ex5_git_diff$ cat qqq
  qqq
  miyakz@lily:/tmp/ex5_git_diff$ echo zzz > qqq
  miyakz@lily:/tmp/ex5_git_diff$ cat qqq
  zzz
  miyakz@lily:/tmp/ex5_git_diff$ git commit -m "qqq to zzz" -a
  [master e68cab4] qqq to zzz
  1 file changed, 1 insertion(+), 1 deletion(-)
  miyakz@lily:/tmp/ex5_git_diff$ 

この直前のコミットが間違えで、中身をhelloに変更したいとする。::

  miyakz@lily:/tmp/ex5_git_diff$ vim qqq
  miyakz@lily:/tmp/ex5_git_diff$ cat qqq
  hello
  miyakz@lily:/tmp/ex5_git_diff$ git add qqq
  miyakz@lily:/tmp/ex5_git_diff$ git commit --amend
  [master 60b60f1] qqq to zzz to hello
   Date: Sun May 3 01:26:54 2015 +0900
   1 file changed, 1 insertion(+), 1 deletion(-)
  miyakz@lily:/tmp/ex5_git_diff$ git log --graph --oneline --decorate  --all
  * 60b60f1 (HEAD, master) qqq to zzz to hello
  * 4145854 qqq
  * 7d0fccb cherry-pick from my_neutron
  * e535492 c
  * debb7b4 z
  * 69da2b0 y
  * 6e3faa2 x
  * 75ef0e0 w
  | * 1406d85 (topic) d
  | * d517715 c
  | *   b54782a Merge branch 'master' into topic
  | |\  
  | |/  
  |/|   
  * | 8d81df9 v
  | * 3ec124b a
  |/  
  * 5e9bce5 u

blobも以下のように編集されており、コミットが上書きされたことがわかる。::

  miyakz@lily:/tmp/ex5_git_diff$ git show 60b60f1
  commit 60b60f1bdb85ecd62a08059f767e1e2e5ae111b5
  Author: kazuhiro MIYASHITA <miyakz1192@gmail.com>
  Date:   Sun May 3 01:26:54 2015 +0900
  
      qqq to zzz to hello
  
  diff --git a/qqq b/qqq
  index 1b7ae83..ce01362 100644
  --- a/qqq
  +++ b/qqq
  @@ -1 +1 @@
  -qqq
  +hello
  miyakz@lily:/tmp/ex5_git_diff$ 

コミットの履歴を書き換える(rebase -i)
---------------------------------------

上記のグラフの状態から、以下のコミットを削除したいとする。::

  * 60b60f1 (HEAD, master) qqq to zzz to hello
  * 4145854 qqq

以下のコマンドを実行。::

  git rebase -i e535492

-iで指定したコミット番号はrebaseコマンドでの実行対象に含まれない。さらに、
この２つのコミットを削除したい場合は、何らかのコミットを残してあげないと行けないので、残すコミット番号を含むe535492からrebaseコマンドをスタートする必要がある。コマンドを実行すると、エディタの画面に以下のコミットが登場するので、7d0fccbを残してエディタから行ごと削除する。::

  pick 7d0fccb cherry-pick from my_neutron
  pick 4145854 qqq
  pick 60b60f1 qqq to zzz to hello

コマンドは成功する::

  miyakz@lily:/tmp/ex5_git_diff$ git rebase -i e535492
  Successfully rebased and updated refs/heads/master.
  miyakz@lily:/tmp/ex5_git_diff$

グラフは以下のようになり、コミットが綺麗に消えている。::

  * 7d0fccb (HEAD, master) cherry-pick from my_neutron
  * e535492 c
  * debb7b4 z
  * 69da2b0 y
  * 6e3faa2 x
  * 75ef0e0 w
  | * 1406d85 (topic) d
  | * d517715 c
  | *   b54782a Merge branch 'master' into topic
  | |\  
  | |/  
  |/|   
  * | 8d81df9 v
  | * 3ec124b a
  |/  
  * 5e9bce5 u

リベースする(rebase -i)
-----------------------

上記のグラフの状態から、topicブランチをmasterでリベースする。::

  miyakz@lily:/tmp/ex6_git_diff$ git branch
  * master
    stable/juno
    topic
  miyakz@lily:/tmp/ex5_git_diff$ git checkout topic
  Switched to branch 'topic'
  miyakz@lily:/tmp/ex5_git_diff$ git branch
    master
    stable/juno
  * topic
  miyakz@lily:/tmp/ex5_git_diff$ git rebase master
  First, rewinding head to replay your work on top of it...
  Applying: a
  Applying: d
  miyakz@lily:/tmp/ex5_git_diff$ 

グラフは以下のようになる。::

  * 2df397a (HEAD, topic) d
  * bf45e71 a
  * 7d0fccb (master) cherry-pick from my_neutron
  * e535492 c
  * debb7b4 z
  * 69da2b0 y
  * 6e3faa2 x
  * 75ef0e0 w
  * 8d81df9 v
  * 5e9bce5 u

スカッシュでコミットをまとめる(rebase -i)
-----------------------------------------

上記のグラフの状態から、v(8d81df9)とw(75ef0e0)を１つのコミットにまとめたいとする。まず、それぞれのコミットについて確認する。::

  miyakz@lily:/tmp/ex5_git_diff$ git show 8d81df9
  commit 8d81df99975447a7492b33628e0d733b8d0c85fb
  Author: kazuhiro MIYASHITA <miyakz1192@gmail.com>
  Date:   Sat May 2 22:27:59 2015 +0900
  
      v
  
  diff --git a/v b/v
  new file mode 100644
  index 0000000..110ed9b
  --- /dev/null
  +++ b/v
  @@ -0,0 +1 @@
  +v
  miyakz@lily:/tmp/ex5_git_diff$ git show 75ef0e0
  commit 75ef0e095a375cf2818e769ef69659ddcca7fb33
  Author: kazuhiro MIYASHITA <miyakz1192@gmail.com>
  Date:   Sat May 2 22:28:21 2015 +0900
  
      w
  
  diff --git a/w b/w
  new file mode 100644
  index 0000000..e556b83
  --- /dev/null
  +++ b/w
  @@ -0,0 +1 @@
  +w
  miyakz@lily:/tmp/ex5_git_diff$ 

スカッシュするために、以下のコマンドを実行する。::

  git rebase -i 5e9bce5

エディタが開き、以下のような状態になっている。::

  pick 8d81df9 v
  pick 75ef0e0 w
  pick 6e3faa2 x
  pick 69da2b0 y
  pick debb7b4 z
  pick e535492 c
  pick 7d0fccb cherry-pick from my_neutron
  pick bf45e71 a
  pick 2df397a d

それを以下のように、編集する::

  pick 8d81df9 v
  squash 75ef0e0 w
  pick 6e3faa2 x
  pick 69da2b0 y
  pick debb7b4 z
  pick e535492 c
  pick 7d0fccb cherry-pick from my_neutron
  pick bf45e71 a
  pick 2df397a d

グラフは以下のような構造になった。もともとの構造が保存されたような形になった。おそらく、topicブランチでrebaseを実行したためと思われる::

  * 03ed80c (HEAD, topic) d
  * 2fc5cb7 a
  * d582962 cherry-pick from my_neutron
  * e72738b c
  * 4323583 z
  * 9de18e0 y
  * a5cef87 x
  * d8cb8d8 v
  | * 7d0fccb (master) cherry-pick from my_neutron
  | * e535492 c
  | * debb7b4 z
  | * 69da2b0 y
  | * 6e3faa2 x
  | * 75ef0e0 w
  | * 8d81df9 v
  |/  
  * 5e9bce5 u

v(d8cb8d8)を確認すると、想定通り、wを含んだコミットになっていた。::

  miyakz@lily:/tmp/ex5_git_diff$ git show d8cb8d8
  commit d8cb8d8a00461c5e77796ab2c4ac223d54b086e1
  Author: kazuhiro MIYASHITA <miyakz1192@gmail.com>
  Date:   Sat May 2 22:27:59 2015 +0900
  
      v
      
      w
  
  diff --git a/v b/v
  new file mode 100644
  index 0000000..110ed9b
  --- /dev/null
  +++ b/v
  @@ -0,0 +1 @@
  +v
  diff --git a/w b/w
  new file mode 100644
  index 0000000..e556b83
  --- /dev/null
  +++ b/w
  @@ -0,0 +1 @@
  +w
  miyakz@lily:/tmp/ex5_git_diff$ 




