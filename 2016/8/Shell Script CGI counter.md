title: Shell ScriptでCGIカウンター
date: 2016-08-22 00:05:00
author: me
preview: CGIカウンターとか懐かしくないですか?
tags:
   - CGI
   - Shell Script

---

CGIカウンターとか懐かしくないですか?

```
#!/bin/sh

file="count.log"
lockfile="counter.lock"
sleeptime=0

while :
do
  #ロック
  mktemp $lockfile 1> /dev/null 2>/dev/null
  result=$?

  #ロック出来てたらカウンタを進める
  if [ $result -eq 0 ]; then
    count=$((`cat $file` + 1))
    echo $count > count.log
    #ロック開放
    rm $lockfile
    break
  fi

  sleeptime=$(($sleeptime + 1))
  sleep "0.00"$sleeptime
done

echo $count
```

`count.log` というファイルを置いておけばカウンタとして動作します．ロックには `mktemp` を使いましたが `mkdir` とかでもいいはず．

ロックされてたら一定時間待つようにしていますが，ちょっと手抜きすぎるかも．
