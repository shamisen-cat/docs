# git: 運用メモ

## `main` へのマージは `--squash` で履歴を整理

`develop` から `main` へのマージは `--squash` で履歴を整理する。

```bash
git checkout main
git merge --squash develop
git commit -m "コミットコメントを入力"
```

## リモートへのプッシュはブランチを指定

リモートへのプッシュはブランチを指定し、`main` 以外のブランチをプッシュしない。

```bash
git push origin main
```

## 安全のための設定

### デフォルトのプッシュ挙動を `simple` に変更

デフォルトのプッシュ挙動を `simple` に変更し、ブランチ指定にする。

```bash
git config --global push.default simple
git config --global --get push.default
```

実行結果

```text
simple
```

### リモート追跡を解除

誤って `develop` や `feature/*` をプッシュしないよう、ローカルブランチの `upstream` を解除しておく。

```bash
git checkout develop
git branch --unset-upstream
```

実行結果の例

```text
fatal: branch 'develop' has no upstream information
```

#### 追跡状況の確認方法

```bash
git branch --format='%(refname:short)' | while read branch
do
  echo -n "$branch: "
  git config branch.$branch.remote || echo "(no upstream)"
done
```

実行結果の例

```text
develop: (no upstream)
main: origin
```
