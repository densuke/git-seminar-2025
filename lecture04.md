---
marp: true
theme: default
paginate: true
mermaid: true
---
<script type="module">
import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@11.4.1/dist/mermaid.esm.min.mjs';
mermaid.initialize({ startOnLoad: true });
</script>

# 4時間目: 変更の統合とコンフリクト解決

## 異なるブランチの変更を統合し、競合を解決する

---

## 学習目標

*   マージの種類を理解します。
*   マージコミットの役割を理解します。
*   マージコンフリクトを解決します。

---

## このセッションで学ぶこと

*   Fast-Forwardを学びます。
*   3-wayマージを学びます。
*   マージコンフリクトの発生を体験します。
*   マージコンフリクトの解決手順を習得します。

---

## マージとは？

*   マージは、異なるブランチの変更を統合する操作です。
*   開発ラインを一つにまとめるために使います。
*   Gitは自動的に変更を統合しようとします。

---

## マージの種類: Fast-Forward

* mainブランチで作業していました
* 途中でfeatureブランチを切ってそちらで作業しました
* mainに戻ってきて、featureブランチの変更を統合しようとしました
* もしこの時、mainを触っていた(コミットした)人がいなかったら? という問題

---

## Fast-Forwardとは? 操作例

<pre class="mermaid">
gitGraph
    commit id: "A"
    commit id: "B" tag: "main"
    branch feature
    checkout feature
    commit id: "C" tag: "feature"
    checkout main
    merge feature tag: "main" tag: "HEAD" tag: "feature"
</pre>
でもこれ、なんかおかしくない? マージ後のコミット(図中だと白抜きの○)って必要ですか?

---

## Fast-Forward

実際、こういうことじゃない?

<pre class="mermaid">
gitGraph
    commit id: "A"
    commit id: "B"
    commit id: "C" tag: "feature" tag: "main" tag: "HEAD"
</pre>

---

## FFになる条件(雑)

FF(Fast-Forwardの略です)はこうなっています。

*   **条件**: 統合先のブランチが、統合元ブランチの直接の祖先である場合です。
*   **挙動**: Gitは単にブランチのポインタを前進させます。
*   **特徴**: 新しいコミットは作成されません。履歴が線形に保たれます。

雑に言えば

* ブランチ切って作業しました
* 完了して枝分かれ元ブランチに戻ってきました
* マージしようとしたけど **誰も途中でコミットしていない** 場合

この条件を満たすとFFとなる。

---

## 演習: Fast-Forwardを体験しよう

3時間目に使っていたリポジトリをそのまま使います。
指示通りにやっているなら、そのまま継続利用できます。

```powershell
PS> cd ~/git-branch-demo
PS> git branch # mainとfeature/new-featureがあるはず、最後にmainに戻ってる?
PS> git switch main # 強制的に戻っておく
```

---

## 新規ブランチでちょっと遊ぶ

```powershell
PS> git switch -c bugfix/newfile # 作成・切り替え
PS> new-item G; git add G; git commit -m "Add file G" # ブランチ上でコミット
PS> git switch main # mainに戻る
PS> git branch # ブランチ一覧確認、bugfix/newfileがあるはず
```

---

## マージ実行(Fast-Forward)

```powershell
PS> git merge bugfix/newfile # マージ実行
# マージコミットは作成されないはず(Fast-Forwardと出力される)
PS> git log --oneline # 履歴を確認
```

他のブランチやmain上でのコミットが作業中に起きていなければFFとなる

---

## マージの種類: 3-wayマージ

*   **条件**: 履歴が分岐している場合です。
*   **挙動**: Gitは共通の祖先を見つけます。
    *   共通の祖先、統合元、統合先の3つのスナップショットを比較します。
    *   新しい「マージコミット」を作成します。
*   **特徴**: マージコミットは複数の親を持ちます。履歴が非線形になります。

<pre class="mermaid">
gitGraph
    commit id: "A"
    commit id: "B" 
    branch feature
    checkout feature
    commit id: "C" tag: "feature"
    checkout main
    commit id: "D" 
    merge feature tag: "main" tag: "HEAD"
</pre>

---

## マージコミットの役割

*   3-wayマージで作成される特別なコミットです。
*   複数の親コミットを持ちます。
*   異なる開発ラインの統合点を示します。
*   履歴の分岐と合流を明確に記録します。

---

## 演習: マージコミットの実施

* 実は現在のリポジトリでは、3時間目にfeature/new-featureブランチを作成しています。
* しかも先程FやGを作っています。

条件を満たしているのでマージコミットが可能です。

```powershell
PS> git switch main # mainに戻る
PS> git merge feature/new-feature # マージ実行
# マージコミットが作成されるはず
PS> git log --oneline # 履歴を確認(マージコミットが生成されている)
```
---

### 演習: マージコミットを確認する

最後のマージコミットのコミットオブジェクトを眺めてみよう

```powershell
PS> git log --oneline -1 #最後のコミット(マージコミット) IDを確認
PS> git cat-file -p <コミットID> # コミットオブジェクトの内容を確認
```

---

## マージコンフリクトとは？

*   Gitが自動的に変更を統合できない場合に発生します。
*   同じファイルの同じ行を、異なるブランチで変更した場合などです。
*   Gitはマージを一時停止し、解決を求めます。
*   コンフリクトはエラーではありません。Gitの通常のワークフローの一部です。

---

## コンフリクトの確認: `git status`

*   コンフリクトが発生すると、`git status`が教えてくれます。
*   どのファイルが未マージ状態かを示します。
*   解決すべきファイルを確認できます。

---

## コンフリクトマーカーの読み方

*   コンフリクトしたファイルには特別なマーカーが挿入されます。

```
<<<<<<< HEAD
現在のブランチでの変更内容
========
マージしようとしているブランチでの変更内容
>>>>>>> <ブランチ名>
```
※ 実際のものとマーカーが違いますが、スライド作成上の理由ですごめんなさい。

*   `<<<<<<< HEAD`: 現在のブランチの変更の始まりです。
*   `=======`: 変更の区切りです。
*   `>>>>>>> <ブランチ名>`: マージしようとしているブランチの変更の終わりです。

---

## コンフリクトの解決手順

1.  `git status`でコンフリクトしたファイルを特定します。
2.  各コンフリクトファイルをテキストエディタで開きます。
3.  `<<<<<<<`, `=======`, `>>>>>>>`マーカーを削除します。
    *   望ましい変更を保持するように手動で編集します。
    *   マーカー自体を消さない人が過去からよく見ます! 注意しましょう。
4.  `git add <解決したファイル名>`で、解決済みとしてステージします。
5.  `git commit`でマージコミットを作成し、マージを完了させます。

---

## 演習: 意図的なコンフリクト

*   **目的**: 管理された環境でマージコンフリクトを経験し、解決します。
*   **手順**:
    1.  `main`ブランチから開始します。
    2.  `poem.txt`を作成し、「The rose is red.」と書いてコミットします。
    3.  `git switch -c feature/violets`で新しいブランチを作成し、切り替えます。
    4.  `poem.txt`を「The violet is blue.」に変更してコミットします。
    5.  `git switch main`で`main`ブランチに戻ります。
    6.  `poem.txt`を「The poppy is red.」に変更してコミットします。
    7.  `git merge feature/violets`を実行します。コンフリクトが発生します。
    8.  コンフリクトを解決し、マージを完了させます。

---

## `poem.txt`を作成し、「The rose is red.」と書いてコミットします。

```pwsh
PS> new-item poem.txt -Value "The rose is red."
PS> git add poem.txt; git commit -m "Add poem.txt with initial line"
```

<pre class="mermaid">
gitGraph
    commit id: "A"
    commit id: "B" tag: "main" tag: "HEAD"
</pre>

---

## `git switch -c feature/violets`で新しいブランチを作成し、切り替えます。

```pwsh
PS> git branch feature/violets
PS> git switch feature/violets
```

<pre class="mermaid">
gitGraph
    commit id: "A"
    commit id: "B" tag: "main" tag: "feature/violets"
    branch feature/violets
    checkout feature/violets
</pre>

---

## `poem.txt`を「The violet is blue.」に変更してコミットします。

```pwsh
PS> set-content poem.txt "The violet is blue."
PS> git add poem.txt; git commit -m "Change poem to violets"
```

<pre class="mermaid">
gitGraph
    commit id: "A"
    commit id: "B" tag: "main"
    branch feature/violets
    checkout feature/violets
    commit id: "C" tag: "feature/violets" tag: "HEAD"
</pre>

---

## `git switch main`で`main`ブランチに戻ります。

```pwsh
PS> git switch main
```

## `poem.txt`を「The poppy is red.」に変更してコミットします。

```pwsh
PS> set-content poem.txt "The poppy is red."
PS> git add poem.txt; git commit -m "Change poem to poppy"
```

<pre class="mermaid">
gitGraph
    commit id: "A"
    commit id: "B" 
    branch feature/violets
    checkout feature/violets
    commit id: "C" tag: "feature/violets"
    checkout main
    commit id: "D" tag: "main" tag: "HEAD"
</pre>

---

## `git merge feature/violets`を実行します。

```pwsh
PS> git merge feature/violets
```
<pre class="mermaid">
gitGraph
    commit id: "A"
    commit id: "B" 
    branch feature/violets
    checkout feature/violets
    commit id: "C" tag: "feature/violets"
    checkout main
    commit id: "D" tag: "main"  tag: "HEAD"
    merge feature/violets type: REVERSE
</pre>

---

## 手直しをしてみよう

`poem.txt`を開いてみると、以下のようなコンフリクトマーカーが表示されます。

```
<<<<<< HEAD
The poppy is red.
======
The violet is blue.
>>>>>> feature/violets
```

ここでは`feature/violets`ブランチの内容を保持してみましょう。

```plaintext
The violet is blue.
```
その後、コンフリクトマーカーを削除します(addすればOK)。
```pwsh
PS> git add poem.txt
PS> git commit -m "Resolve merge conflict in poem.txt"
```

---

## まとめ

*   マージにはファストフォワードと3-wayマージがあります。
*   マージコンフリクトはGitの通常の機能です。
*   コンフリクトマーカーを理解し、手動で解決できます。
*   `git status`でコンフリクト状態を確認します。
*   `git add`と`git commit`で解決を完了します。

コンフリクトの解決は、発生した原因の方とコミュニケーションを取る必要があります。
勝手に行うと相手の意図が反映されない可能性が出るため、注意が必要です。

---

## おまけ: 3-wayってなにをしているのか

- base: 共通の祖先コミット
- ours: 現在のブランチの最新コミット
- theirs: マージしようとしているブランチの最新コミット

* base→oursの差分を取得する --- (A)
* base→theirsの差分を取得する ---(B)
* (A)と(B)の混合コミットを計算して適用する
  * 両方の差分が重なっているとき、コンフリクトが発生する
