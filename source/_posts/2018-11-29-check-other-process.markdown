---
layout: post
title: "シェルスクリプトで別プロセスの終了ステータスを取得する"
date: 2018-11-29 14:55:51 +0900
comments: true
categories: linux
---

シェルスクリプトで別プロセスの終了ステータスを取得するTipを紹介します。

### ユースケース

時間のかかる処理などを並列で行い、その実行結果(終了ステータス)を取得したい。

### サンプルコード

Bashのサンプルコードです。スクリプトに解説を記入しています。

```
#!/bin/bash

command1() {
  echo "executing commnad1"
  sleep 3
}

command2() {
  echo "executing commnad2"
  sleep 3
  # エラーとして終了させる
  exit 1
}

echo "start"

# バックグラウンドで実行
command1 &
# $!で直前に実行されたコマンドのプロセスIDを取得し、変数に保存している
pid1=$!

command2 &
pid2=$!

# waitは指定されたプロセスIDの処理が終わるまで待つ
wait $pid1

# $?でwaitで指定しているプロセスの終了ステータスを取得することができる
if [ $? != 0 ]; then
  echo "command1 error"
else
  echo "commnad1 success"
fi

wait $pid2
if [ $? != 0 ]; then
  echo "command2 error"
else
  echo "command2 success"
fi

echo "end"
```

gistにもコードをアップしています。

* <a href="https://gist.github.com/shoyan/6f45b6a005fd34a44264fa1dcb4d56e6" target="_blank">wait_process.sh</a>
