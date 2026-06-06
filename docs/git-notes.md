# Gitメモ

このプロジェクトではGitの練習も兼ねて、作業を小さく分けて記録します。

## よく使うコマンド

```bash
git status
git add .
git commit -m "初期設定を追加"
git log --oneline
```

## 基本の流れ

1. 変更状況を見る

```bash
git status
```

2. コミット対象に追加する

```bash
git add README.md .gitignore docs/git-notes.md
```

3. コミットする

```bash
git commit -m "プロジェクト初期設定を追加"
```

## コミット前に見るもの

```bash
git diff
git diff --staged
git status
```

## 迷ったら

まず`git status`を見る。
