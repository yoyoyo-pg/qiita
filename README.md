# Qiita記事管理用リポジトリ

- [Qiitaの記事をGitHubリポジトリで管理する方法](https://qiita.com/Qiita/items/32c79014509987541130)
- [QiitaCLI を使って Qiita の記事を GitHub 管理する方法](https://qiita.com/ryocha12/items/e412306f9e8339d7cffe)

## コマンド

```powershell
# QIita執筆前に念の為pull
npx qiita pull
# Qiita記事作成
npx qiita preview
```

### Qiita CLIセットアップ

```powershell
# Qiita CLIのインストール
npm install @qiita/qiita-cli --save-dev
# Qiita上でトークンを発行しログイン
npx qiita login
```

## 記事作成方法

- `新規記事作成`をクリック
- 以下、編集内容

```md
title: # 記事のタイトル
tags: # 記事のタグ
  - xxx
  - yyy
private: # true: 限定共有記事 / false: 公開記事
updated_at: # 記事を投稿した際に自動的に記事の更新日時に変わります
id: # 記事を投稿した際に自動的に記事のUUIDに変わります
slide: # true: スライドモードON / false: スライドモードOFF
ignorePublish: # true: `publish`コマンドにおいて無視されます（Qiitaに投稿されません） / false: `publish`コマンドで処理されます（Qiitaに投稿されます）
```

- `main`ブランチにマージされると、GitHubActionsでQiitaに反映
