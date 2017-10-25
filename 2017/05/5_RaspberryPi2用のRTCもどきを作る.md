title: RaspberryPi2用のRTCもどきを作る
date: 2017-05-05 06:00:00
author: me
preview: 再起動したときntpで同期できないのが辛いので
tags:
   - linux
   - systemd
   - Shell Script
   - Raspberry Pi2

---

Raspberry Pi2にはRTCがありませんが、これによって再起動したときに時刻が `2017年1月1日` とかになります。

システム時刻が現在時刻とかけ離れている場合、NTPによる同期が失敗することがあり、ラズパイを再起動するといつまでたっても正しい時刻に戻ってくれず大変困ります。

RTCモジュール買えば良いんですが、再起動したときにササッとNTPに同期できるようにしたいだけなので、タダで済ませたいところです。

そこで、再起動する直前の時刻時刻をファイルに書き出しておき、再起動後にセットすることでNTPと同期できる程度の差に抑えることにしました。

<a href="https://github.com/dozen/myrtc" target="_blank">dozen/myrtc - Github</a>

Raspberry Pi2にはArchlinuxを入れているので、systemdで使えるようにしました。

`myrtc` を `/usr/local/myrtc/` に置いて、 `myrtc.service` を `/etc/systemd/system/` に置きます。

`/usr/local/myrtc/myrtc store` で現在時刻をファイルに保存し、 `/usr/local/myrtc/myrtc restore` でファイルに書き出した時刻をシステム時刻にセットします。

`restore` にはroot権限が必要です。

`myrtc.service` はonshotタイプのサービスで、start時に`restore`コマンドを実行し、stop時に`store`コマンドを実行することで、起動時に`restore`、シャットダウン時に`store`を実行できるようにしています。

再起動にかかる時間は考慮されていないので、myrtcでセットし直した時刻は、私のRaspberry Pi2でだいたい30秒くらい遅れます。storeかrestoreどちらかでその間に進む時刻を大体で加算すればもっと誤差が小さくなりそうですが、NTPで同期できればそれでいいので、ややこしいことはしていません。

もし不慮のシャットダウンなども考慮したい場合は、cronで`myrtc store`を定期的に実行するか、 myrtcコマンドを書き換えて無限ループの中でスリープを挟みながら時刻をファイルに書き出し続ける、みたいにすればいい感じになりそうです。

もし後者のmyrtcを書き換えるパターンを実践する場合は、 `myrtc.service` を書き換える必要がありますね。

ともかくこれでRaspberry Pi2を再起動したとき、毎回 `date -s …` と打ち込まずに済むようになりました。


追記

`tinker panic 0`とすればどんなに時刻がずれていても同期してくれるので、こんな物はいらないですね…🐘
