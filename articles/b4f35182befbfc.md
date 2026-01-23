---
title: "Git pushで「RPC failed; HTTP 400」エラーが出たときの解決法"
emoji: "🔥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["git", "github","ファイルサイズ", "エラー"]
published: true
---

## 発生したエラー

大きめのファイルをGitHubにプッシュしようとしたら、こんなエラーが出ちゃいました。

```bash
$ git push
Enumerating objects: 34, done.
Counting objects: 100% (34/34), done.
Delta compression using up to 8 threads
Compressing objects: 100% (32/32), done.
error: RPC failed; HTTP 400 curl 22 The requested URL returned error: 400
send-pack: unexpected disconnect while reading sideband packet
Writing objects: 100% (33/33), 14.05 MiB | 15.16 MiB/s, done.
Total 33 (delta 5), reused 0 (delta 0), pack-reused 0
fatal: the remote end hung up unexpectedly
Everything up-to-date
```

約14MBのデータをプッシュしようとして失敗してるみたいですね。

## 原因

原因はシンプルで、GitのデフォルトのHTTP postBufferサイズ（だいたい1MB程度）を超えちゃったからです。画像ファイルとかバイナリファイルが入ってると、よくこのエラーに遭遇します。

今回のケースでは、以下のような1MB以上のファイルが複数含まれていました：

- 画像ファイル（JPG、PNG）：最大1.9MB
- Blendファイル：最大1.1MB

## 解決方法

HTTPバッファサイズを増やせば解決します。簡単！

```bash
# HTTPバッファサイズを500MBに設定
$ git config http.postBuffer 524288000

# 再度プッシュ
$ git push
```

これで無事プッシュできるようになりました。

### 設定の確認

ちゃんと設定できたか確認するには：

```bash
$ git config --get http.postBuffer
524288000
```

### グローバル設定にする場合

全部のリポジトリで使いたいなら、`--global` オプションを付ければOKです：

```bash
$ git config --global http.postBuffer 524288000
```

## より良い解決策：Git LFS

大きいファイルを頻繁に扱うなら、**Git LFS（Large File Storage）**を使うのがおすすめです。

### Git LFSのメリット

- 大きなファイルを効率的に管理
- リポジトリのクローン速度が向上
- 履歴が軽量に保たれる

### Git LFSの基本的な使い方

```bash
# Git LFSのインストール（macOS）
$ brew install git-lfs

# リポジトリでGit LFSを有効化
$ git lfs install

# 特定の拡張子をLFS管理下に置く
$ git lfs track "*.jpg"
$ git lfs track "*.png"
$ git lfs track "*.blend"

# .gitattributesをコミット
$ git add .gitattributes
$ git commit -m "Configure Git LFS"

# 通常通りプッシュ
$ git push
```

## まとめ

- `git push` で HTTP 400 エラーが出たら、`http.postBuffer` の設定を増やす
- とりあえず対処するなら `git config http.postBuffer 524288000`
- 大きいファイルをよく使うなら Git LFS を入れておくと便利

これで大きなファイルがあっても気にせずプッシュできますね！
