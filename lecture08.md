marp: true
<style>
  .mermaid {
    font-size: 2em;
  }
</style>

theme: default
mermaid: true
---
<script type="module">
import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@11.4.1/dist/mermaid.esm.min.mjs';
mermaid.initialize({ startOnLoad: true });
</script>

# 8時間目: 落ち穂拾い

## 高度な履歴操作と便利なツール（`rebase`, `stash`, `.gitignore`）

---

## 学習目標

*   より高度な履歴操作である`rebase`を理解します。
*   日常的に役立つGitツールを学びます。
*   よりクリーンな履歴を保つためのテクニック

---

## リベースについて
### `rebase`とは？

`rebase`は、あるブランチの変更を別のブランチに適用する操作です。
これにより、履歴をより直線的に保つことができます。  
`rebase`は、特にフィーチャーブランチを`main`に統合する際に有用です。

---

### `merge` と `rebase` の違い

まずは、`main`ブランチの変更を`feature-branch`に取り込む2つの方法、`merge`と`rebase`の違いを見てみましょう。

---

### 方法1: `merge` を使う場合

`main`での変更 (`D`) を `feature-branch` にマージします。

<pre class="mermaid">
gitGraph
  commit id: "A"
  commit id: "B"
  branch feature-branch
  commit id: "C"
  checkout main
  commit id: "D"
  checkout feature-branch
  merge main id: "E"
</pre>

*   履歴が分岐し、マージコミット（`E`）が作られます。
*   「`main`の変更を取り込んだ」という事実が履歴に残ります。

---

### 方法2: `rebase` を使う場合

`feature-branch` を `main` の最新状態の上に「付け替え」ます。

<div style="display: flex; justify-content: space-between;">
<div style="width: 45%;">

**Before (rebase前)**
<pre class="mermaid">
gitGraph
  commit id: "A"
  commit id: "B"
  branch feature-branch
  checkout feature-branch
  commit id: "C"
  checkout main
  commit id: "D"
</pre>

</div>
<div style="width: 45%;">

**After (rebase後)**
<pre class="mermaid">
gitGraph
  commit id: "A"
  commit id: "B"
  commit id: "D"
  branch feature-branch
  checkout feature-branch
  commit id: "C'"
</pre>

</div>
</div>

*   `feature-branch`のベース（分岐点）が`B`から`D`に変更されます。
*   コミット`C`は、新しいベース`D`の上で`C'`として作り直されます。
*   履歴が一直線になり、きれいに見えます。

---

### `rebase` のメリット

*   **履歴がクリーンになる**
    *   マージコミットが不要なため、履歴が一直線になり、プロジェクトの歴史を追いやすくなります。
*   **Fast-Forwardマージができる**
    *   `rebase`後のブランチを`main`に統合する際、`main`はただポインタを進めるだけで済みます。



---

### リベースの注意点・デメリット

*   **元のコミットはどこへ？**
    *   `rebase`すると元のコミット(例:`C`)は履歴から見えなくなります。
    *   これらは参照されなくなっただけで、すぐには消えません(reflogで対応可)。

*   **コンフリクト解決が大変な場合がある**
    *   `rebase`では、移動したコミット1つ1つを適用していきます。
    *   そのため、複数のコミットでコンフリクトが発生すると、その回数分、コンフリクトを解決する必要があります。
    *   `rebase`中にコンフリクトが発生した場合：
        - `git add <解決済みファイル>` で解決をマーク
        - `git rebase --continue` で続行
        - `git rebase --abort` で中止も可能

---

### 演習: `rebase`を試す(準備1)

同じコマンドを繰り替えるので、一時的に関数を作って楽をしましょう
```pwsh
PS> function add { param($n); new-item $n -Value $n; git add $n; git commit -m "add $n" } 
```

これで `add A`などでファイルを作って即時コミットできます。

---

### 演習: `rebase`を試す(準備2)

ベースの構造を作っていきましょう。
```pwsh
PS> mkdir ~/rebase-test; cd ~/rebase-test
PS> git init
PS> add A; add B; git switch -c feature-branch; add C
PS> git switch main; add D
PS> git log main --graph --oneline
PS> git log feature-branch --graph --oneline
```

<pre class="mermaid">
gitGraph
  commit id: "A"
  commit id: "B"
  branch feature-branch
  commit id: "C"
  checkout main
  commit id: "D"
</pre>
A,Bは共通祖先になっていることをIDで確認してください。

---

### 演習: `rebase`を試す(実行)
`feature-branch`を`main`の最新状態に合わせてリベースします
```pwsh
PS> git switch feature-branch # リベースしたいブランチに切り替えて、
PS> git rebase main # つなぎ替えたいブランチを指定してリベース実行
```
<pre class="mermaid"> 
gitGraph
  commit id: "A"
  commit id: "B"
  commit id: "D"
  branch feature-branch
  checkout feature-branch
  commit id: "C'"
</pre>

- mainブランチに戻り、feature-branchをマージするとFast-Forwardになります
  - 興味ある方はお試しを

---

### `rebase`の黄金律 (絶対的なルール)

**絶対に、複数人で共有しているブランチで `rebase` を実行してはいけません。**
(例: `origin/main` や `origin/develop` など)

**`rebase`はコミットを作り直す(履歴を書き換える)操作です(すごく重要)。**
もし共有ブランチの履歴を書き換えて`push`してしまうと、
他のチームメンバーのリポジトリと深刻な食い違いを生み出し、
チーム全体を大混乱に陥れてしまいます。

**「`push`済みのブランチは`rebase`しない」** と覚えておきましょう。

---

## `stash`について
### `stash`とは？

`stash`は、現在の作業内容を一時的に保存し、後で再開できるようにするGitの機能です。
これにより、作業中の変更を一時的に退避させ、他のブランチに切り替えたり、別の作業を行ったりすることができます。

なおstashは構造上いわゆる『スタック』の挙動を持っていることから、明示的に指定しない場合は操作がpush/popのような扱いとなります。
そのため、以降の説明ではstashに入れる操作はpush操作、取り出す操作はpop操作として表現します。

---

### `stash`のユースケース(1)

* ある機能を追加していました
  * 「あれ? ここバグじゃね?」
  * 先にバグを潰す必要があるので、作業中の内容を一旦引っ込めておきます(クリーンな状況にしておく)
    * stash(push操作)
  * バグ修正コミットを行ってコミットをします
  * 「よし、さっきの続きをしよう」と引っ込めたものを取り出してきます
    * stash(pop操作)

---

### `stash`のユースケース(2)

ブランチ切り替え前に待避するケース

* 『ちょっと悪いけどこっちのブランチの作業を手伝って!』
* とはいえこちらも作業中かつコミットするには中途半端という状態です
* 一度stash(push操作)してクリーンな状態の戻してからブランチを切り替えます
* お手伝いが終わり、戻ってきたらstash(pop操作)で戻します

---

### `stash`のユースケース(3)

`pull`や`rebase`前の待避

- `pull`や`rebase`を行う前に、現在の変更を一時的に保存しておきたい場合
- これにより、`pull`や`rebase`中にコンフリクトが発生した場合でも、作業内容を失うことなく対応できます

終了後にstash(pop操作)で戻すことで、最新の状態に反映させることができます。
※ 副作用でコンフリクトが起きることはあります

---

### `stash`の基本コマンド

地味にサブコマンドが多いので、基本のみにします。

```pwsh
PS> git stash push -m "作業内容の説明" # 作業中の変更をstashに保存
PS> git stash -m "作業内容の説明"      # pushは書かなくてもよい
PS> git stash list                   # 保存したstashの一覧を表示  
PS> git stash pop                    # 最新のstashを適用して削除    
PS> git stash pop stash@{0}          # 特定のstashを適用して削除
```

これ以外に「適用するけど削除しない」操作や、「削除せずに捨てる」ためのサブコマンドもあります。

---

## `.gitignore`
### `.gitignore`とは？

`.gitignore`は、Gitが追跡しないファイルやディレクトリを指定するためのファイルです。
リポジトリに含めたくないファイルを指定し、うっかりコミットに含めることを防ぎます。

---

### どんなものを含めてはいけないのか?(1)

具体的にはいろいろあると思いますが、例えば以下のようなものが該当します。

* ビルド成果物(プログラム本体など)
  * ソースから生成できるはずのものはコミットに含めないのが基本です
  * 逆にソースから生成できないものはコミットに含めましょう
* IDEの設定ファイルなど、ローカル環境固有の設定
  * VSCodeやIntelliJなどの設定ファイルは、プロジェクトごとに異なるため、通常はコミットしません
  * ただし開発メンバー全員で共有すべき設定がある場合はワークスペース設定として切り離して含めるべきという場合もあります
* 一時ファイルやキャッシュ
  * ビルド中に生成される一時ファイルやキャッシュファイルもコミットに含めないのが基本です

---

### どんなものを含めてはいけないのか?(2)

* センシティブ情報
  * APIキーやパスワードなどのセンシティブな情報は、`.gitignore`に追加してコミットしないようにします
  * `.env` ファイルなども含まれます
    * 『Webアプリケーション開発』では敢えて行っていますが、あれは授業進行のための例外中の例外です
    * AS構築では **『許さない』** でしたよね(Actions Secret)
* ログファイル
  * デバッグの都合で検証用のデータとしてコミットすることは稀にある

**超重要**
センシティブ情報のコミットおよび(GitHubなど)公開は、**情報漏洩事案ということでセキュリティ上の大きなリスク**となります。

---

### `.gitignore`の書き方

`.gitignore`ファイルには、無視したいファイルやディレクトリの名前を1行に1つずつ記述します。

```
# これはコメントです

# "secret.txt" という名前のファイルを無視
secret.txt

# "logs" という名前のディレクトリ全体を無視
logs/

# ".log" で終わるすべてのファイルを無視
*.log

# 特定のファイルだけは無視しない (!)
!important.log
```

ワイルドカード(`*`)や、特定のパターンを否定する(`!`)などの記法も使えます。

---

### `.gitignore`自体はコミットしよう

`.gitignore`ファイルは、プロジェクトのルールを定義する重要な設定ファイルです。

*   チームメンバー全員が同じルールでファイルを無視できるように、
    `.gitignore`ファイル自体はリポジトリにコミットして共有するのが一般的です。

これにより、「自分の環境では問題なかったのに、他の人の環境では不要なファイルがコミットされてしまった」といった事態を防ぎます。
