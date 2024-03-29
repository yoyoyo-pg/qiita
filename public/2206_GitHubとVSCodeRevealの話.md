---
title: GitHubとVSCode、vscode-revealを使って、知識のストック→アウトプットをシームレスに行える環境を整えた話
tags:
  - GitHub
  - 初心者
  - reveal.js
  - VSCode
private: false
updated_at: '2024-03-16T08:45:40+09:00'
id: 413729f47854af2e644b
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに

学習の記録の選択肢として「紙」「ローカルPC内」「メモアプリ」などなど、様々な選択肢がありますが、

- エンジニアらしくGitHubで管理してみた
- そのままプレゼン資料を作れる状態にした  
という話です

この記事のプレゼン資料は[こちら](https://yoyoyo-pg.github.io/vscode-reveal-sample/)

---

### 対象

- プレゼン資料作成に手間をかけたくない人
- 学習記録をどこかに統一で管理したい人
- 初心者向け記事です

---

### 前提

- Gitの基本的な使い方が分かること
- Git、VSCodeがインストール済であること

---

### 今回伝えたいこと

- VSCodeの拡張機能最強！
- mdは便利！そしてreveal.js最強！

--

最近はGithubで`.md`ファイルの形式で学習記録を残すようにしています。  
`.md`ファイルで残すことによって、情報が整理されるのもありますし、  
Reveal.jsを使うことで、その情報を殆ど加工なしにスライド作成できます。

---

### Reveal.jsとは

- HTMLでスライドを作れる
- mdでも書ける

--

ただし、reveal.jsの環境構築が面倒  
VSCodeの場合「vscode-reveal」という拡張機能をインストールすれば簡単に使えます！

---

### vscode-revealの使い方

- VSCode上で`.md`ファイルを編集し、そのプレビューのような形でプレゼン資料が作成できます。
- `"---"`の区切りで横スライド、`"--"`の区切りで縦スライドを追加できます

--

こちらのQiita記事を参考にしました。
[これからのプレゼン資料は reveal.js を使おう](https://qiita.com/Targityen/items/40ae4795e2cb77c1adc6)

---

### GitHub連携

- 今度は「GitLens」という拡張機能をインストールします
- 「GitLens」によりGitの各種操作をVSCodeのGUI上で実行可能です
- 2022/06/18追記：詳細はこちらの記事「[VSCodeの拡張機能「GitLens」で快適なGitライフを手に入れよう！」](https://qiita.com/yoyoyo_pg/items/e7f010dd13f99e61beba)をご覧ください

--

ここまでの準備で、GitHub上で管理している`.md`ファイルをそのままスライドにしたり、そのままQiitaに移すことが可能になります

---

### 拡張機能まとめ

- 最低限「GitLens」と「vscode-reveal」
- 文法チェックで「markdownlint」も入れておくと便利

---

### まとめ

- 「`.md`での情報を様々な場所で利活用できる」のがメリットだと思います
  - そのままQiitaに転用でき、スライドも作成できる
- `.md`記法に従いつつ、アウトプットを意識することで、情報を簡潔にまとめる癖がつきます

--

基本的にマークダウンの内容を書き換えなくて大丈夫なので、日々の「インプット」「アウトプット」をシームレスに行えるのが魅力的な部分です！

--

また、今回の記事は「vscode-reveal」でスライドとして使える状態にした`.md`ファイルを、そのままコピペして持ってきています（Qiita用に書き換えなくて良いので便利！）  

※唯一`.md`ファイルの上部に記載する、vscode-revealのスライド用の記載だけはQiita投稿時に消しています。

```md
---
theme: "night"
title: "GitHubとVSCodeの話"
enableChalkboard: false
slideNumber: true
---
```

---

### おまけ

GitHub＆GitHubPagesで今回の記事のファイル＆vscode-revealで生成されたプレゼンページを公開しました！

- `.md`ファイル置き場
<https://github.com/yoyoyo-pg/vscode-reveal-sample>

- プレゼンページ（reveal.jsでのプレゼン資料の使用感をお試し頂けます！）
<https://yoyoyo-pg.github.io/vscode-reveal-sample/>
