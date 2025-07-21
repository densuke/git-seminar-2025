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

# 3時間目: ブランチの基本と履歴の確認

## Gitのブランチと履歴を効果的にナビゲートする

---

## 学習目標

*   ブランチの概念を理解します。
*   `git log`で履歴を効果的に確認します。
*   ブランチ操作の基本を習得します。

---

## このセッションで学ぶこと

*   ブランチはコミットへの軽量なポインタです。
*   `HEAD`の役割を理解します。
*   `git branch`コマンドを使います。
*   `git switch`コマンドを使います。
*   `git log`の基本と主要オプションを学びます。
*   視覚ツールでブランチとログの動きを体感します。

---

## ブランチとは？

*   ブランチは「プロジェクトのコピー」ではありません。
*   ブランチはコミットへの「軽量なポインタ」です。
*   特定のコミットを指す、移動可能な参照です。
*   これにより、複数の開発ラインを並行して進められます。

---

## `HEAD`とは？

<pre class="mermaid">
gitGraph
    commit id: "A"
    commit id: "B"
    commit id: "C" tag: "main(HEAD)"
</pre>

*   `HEAD`は現在のブランチを指す特別なポインタです。
*   ほとんどの場合、`HEAD`はブランチ名（例: `main`）を指します。
*   そのブランチ名がコミットを指します。
*   「ブランチ上にいる」状態を示します。

※ この場合、A,B,Cはmainブランチ上にいて、Cが最新コミットです。HEADがCを指しています。

---

## `main`ブランチ

* mainブランチは、Gitにおけるデフォルトのブランチ名です。
  * もともとは `master` でしたが、現在は `main` が推奨されています。
* プロジェクトの「主要な」開発ラインを表します。
* 実際にこれをどう考えるかはプロジェクト次第
  * 何も考えず最新コードを入れる場所
    * 何も考えずに作ったものをとにかく放り込むケースも(危険)
  * 実際に動かせるコードを入れる場所
    * 各部の動いた部品を持ち込む
  * その他いろいろ考えられる

---

## ブランチの操作: `git branch`

*   **ブランチの作成**: `git branch <ブランチ名>`
    *   例: `git branch feature/new-feature`
*   **ブランチの一覧表示**: `git branch`
    *   現在のブランチがハイライトされます。
*   **ブランチの削除**: `git branch -d <ブランチ名>`
    *   マージ済みのブランチを削除します。

---

## ブランチの切り替え: `git switch`

*   **ブランチの切り替え**: `git switch <ブランチ名>`
    *   例: `git switch feature/new-feature`
    *   ワーキングディレクトリとインデックスが切り替わります。
*   **ブランチ作成と同時に切り替え**: `git switch -c <新しいブランチ名>`
    *   例: `git switch -c hotfix/bugfix`

---

## 履歴の確認: `git log`

ここまででも登場してますが、この後も使うので一応おさらい

-   `git log`: デフォルトの詳細なビューです。
-   `git log --oneline`: コミットごとに1行で表示します。
-   `git log --graph`: コミット履歴をASCIIアートで描画します。
-   `git log --decorate`: ブランチやタグのポインタを表示します。
-   `git log --all`: 現在のブランチだけでなく、すべてのブランチを表示します。

---

## 演習: ブランチとログの動き

*   **目的**: ブランチ操作と履歴表示の連動を体感します。
*   **手順**:
    1.  新しいブランチを作成し、切り替えます。
    2.  いくつかのコミットを作成します。
    3.  `git log --oneline --graph --decorate --all`を実行します。
    4.  ブランチのポインタがどのように動くかを確認します。

準備として、彼の練習リポジトリを作っておきましょう。

```pwsh
PS> mkdir ~/git-branch-demo; cd ~/git-branch-demo
```

---

## ブランチを作ってみる(1)

<pre class="mermaid">
gitGraph
    commit id: "A"
    commit id: "B"
    commit id: "C" tag: "main" tag: "HEAD"
</pre>

```pwsh
PS> new-item A # 空のファイルAを作成
PS> git add A
PS> git commit -m "Add file A"
PS> new-item B # 空のファイルBを作成
PS> git add B
PS> git commit -m "Add file B"
PS> new-item C # 空のファイルCを作成
PS> git add C
PS> git commit -m "Add file C"
PS> git log --oneline --graph
```

---

## ブランチを作ってみる(2)

<pre class="mermaid">
gitGraph
    commit id: "A"
    commit id: "B"
    commit id: "C" tag: "main" tag: "HEAD"
    branch feature/new-feature
</pre>

```pwsh
PS> git branch feature/new-feature # 新しいブランチを作成
PS> git switch feature/new-feature # ブランチを切り替え
```
- 別に"/"があっても問題ありません
- わかりやすい、英語での名前を付けることを推奨
    - featureとかbugfixとか

---

## ブランチを作ってみる(3)

<pre class="mermaid">
gitGraph
    commit id: "A"
    commit id: "B"
    commit id: "C" tag: "main"
    branch feature/new-feature
    commit id: "D" tag: "feature/new-feature" tag: "HEAD"
</pre>

```pwsh
PS> new-item D # 空のファイルDを作成
PS> git add D
PS> git commit -m "Add file D"
PS> git log --oneline --graph
```

* 切り替えた後のコミットがブランチに乗ってくる
* それより前の部分は`main`(分岐元)であることにも注意

---

## ブランチを作ってみる(4) やってみよう

<pre class="mermaid">
gitGraph
    commit id: "A"
    commit id: "B"
    commit id: "C" tag: "main"
    branch feature/new-feature
    commit id: "D" tag: "feature/new-feature"
    commit id: "E" tag: "feature/new-feature"
    switch main
    commit id: "F" tag: "main" tag: "HEAD"
</pre>
※ mainブランチにあることを示すため便宜上Cにも`main`タグを付けていますが、実際の場所はFです。
※ 同様にDも便宜上付けています

- このまま(feature/new-featureブランチ)でファイルEを作成してコミット
- mainブランチに戻ってファイルFを作成してコミット

---

## ブランチを作ってみる(4) 操作例

```pwsh
PS> new-item E # 空のファイルEを作成
PS> git add E; git commit -m "Add file E"
PS> git switch main # mainブランチに戻る
PS> new-item F # 空のファイルFを作成
PS> git add F; git commit -m "Add file F"
```

完了後、A〜Fファイルはなにがある? ない?

---

## まとめ

*  ブランチはコミットへの軽量なポインタであり、複数の開発ラインを並行して進めることができます。
*  `git branch`コマンドでブランチの作成ができます
*  `git switch`コマンドでブランチの切り替えができます。
*  `git log`コマンドで履歴を確認できます。
*  ブランチを移動すると、追加しているファイル達は前のブランチの状態によっては消えます(隠れます)
