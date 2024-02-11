---
title: VSCodeの拡張機能「GitLens」で快適なGitライフを手に入れよう！
tags:
  - Git
  - VSCode
  - GitLens
private: false
updated_at: '2024-02-11T12:04:59+09:00'
id: e7f010dd13f99e61beba
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに

- 以前の記事である[GitHubとVSCode、vscode-revealを使って、知識のストック→アウトプットをシームレスに行える環境を整えた話](https://qiita.com/yoyoyo_pg/items/413729f47854af2e644b)の続きです
- 前回の記事ではあまり触れていなかった、VSCode拡張機能の`GitLens`についての使い方解説です

---

### 対象

- VSCode上でラクにGit操作をしたい方

---

### 前提

- Gitの基本的な使い方が分かること
- Git、VSCodeがインストール済であること

---

### VSCodeデフォルトの場合

![2022-06-18-15-22-34.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/ced4b4bb-a85d-ddfa-807a-d0c5bc9d3e1b.png)
![2022-06-18-15-20-31.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/b69c535e-d244-2a29-91fe-03a2e42ee4f7.png)

- 基本的なGit操作はできる
  - 変更の追跡、差分確認、コミット、プッシュなど

---

### GitLensとは

![2022-06-18-15-31-12.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/a8fefd4b-56e8-6b62-b1d5-331b170572ef.png)

- VSCode内でGitの各種操作をしやすくするための拡張機能
- リポジトリの状態、ブランチ管理を視覚的に行える

---

### GitLensを入れてみると

- 先ほどの「ソース管理」内に以下のビューが追加
- 今回はその中でもよく使うビューについて紹介します
![2022-06-18-15-49-46.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/31eb7c00-dd0b-7439-761e-b3e1da43dc73.png)

---

#### COMMITS（コミットビュー）

現在のブランチのコミットが一覧表示されます  
![2022-06-18-15-53-19.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/9fda5dbf-34cf-d547-ed07-524df64cb578.png)

--

また、過去のコミットの詳細も確認できます
![2022-06-18-15-54-57.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/b35aab68-1fac-25a9-a344-a7b322278b36.png)

---

#### FILE HISTORY（ファイル履歴ビュー）

現在開いているファイルの更新履歴を可視化してくれるビューです  
こちらも、コミットビューと同様にコミットの詳細を確認できます
![2022-06-18-16-01-55.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/2d675a9c-8cb2-a70c-6305-fcdfd0e0b2a3.png)

---

#### BRANCHES（ブランチビュー）

すべてのローカルブランチが一覧表示されます
右側に「✓」がついているのが現在のブランチです
視覚的にブランチの状態が分かる点が便利です
![2022-06-18-16-06-20.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/32ee888f-f5af-00f5-2222-73f1547997e2.png)

--

**プルされていない変更がある場合**  
「↓」アイコンクリックでプルができます
![2022-06-18-16-08-21.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/8c2b1a24-232f-52d8-5930-d7a1d5143819.png)

**プッシュしていない変更がある場合**  
「↑」アイコンクリックでプッシュができます
![2022-06-18-16-12-06.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/72629c0b-aadf-5156-7c47-739cdfa62c7b.png)

--

**ブランチ作成**  
ブランチを選択し右クリック > Create Branchを選択  
![2022-06-18-16-16-20.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/f2fb04a9-e908-70ce-b507-07fb77c835d4.png)

上部メニューに新規ブランチ名を入力してEnterで、新しいブランチが作成できます
![2022-06-18-16-17-52.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/a9a9ba31-9073-8adf-67f8-9209acea38ad.png)
![2022-06-18-16-19-07.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/c818d763-22f1-b042-a64a-faf8cdc5819e.png)

---

#### REMOTES（リモートビュー）

リモートの状態が一覧表示されます  
fetchやpruneがアイコンクリックでできます
![2022-06-18-16-26-21.png).png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/eebcd04f-ff1a-c562-2031-30605896358e.png)

リモートがない場合、以下のように表示されます
![2022-06-18-16-29-53.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/b22e66be-a62a-8a31-1ae6-03b65c292de6.png)

--

**新規リモートの作成**
「+」アイコンクリック後、リモート名、リポジトリURIを入力しEnter  
すると、リモートに追加されます
![2022-06-18-16-38-33.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/4c956cde-c228-1b12-6979-a523304cf464.png)
![2022-06-18-16-36-32.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/369861ac-a9af-53c5-1f69-ed85427b35e8.png)
![2022-06-18-16-36-51.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/00ebb888-0057-ebd3-f3b1-f3f494558ed3.png)

---

### まとめ

- 手軽にGit操作ができるので非常に便利です！
- 他にも便利な機能があれば是非教えてください！

---

### 参考文献

- [gitkraken/vscode-gitlens](https://github.com/gitkraken/vscode-gitlens#repositories-view-)
- [GitLens --- Git supercharged](https://gitlens.amod.io/)
