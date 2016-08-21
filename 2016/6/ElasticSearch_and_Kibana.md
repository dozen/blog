title: ElasticSearch + Kibanaでサーバのリソース監視
date: 2016-06-26 00:00:00
author: me
preview: fluentdの設定ファイルを他所から丸パクリしたら動かなかった
tags:
   - elasticsearch
   - kibana
   - fluentd

---

dstatでCPU使用率, メモリ, スワップ, ネットワークI/O, ディスクI/Oなどを取得してfluentdでElasticSearchに投げつけるというものです.

それをKibanaで視覚化します. ElasticSearch + Kibana + fluentd + dstatのやり方については他に紹介してるWebサイトがたくさんあるので割愛します.

dstatだかfluentdのdstatプラグインの仕様が変わっているだかで, 参考にしたWebサイトの設定ファイルでは動きませんでした. ということで修正した `fluentd.conf` を紹介します.

## fluent.conf

``` text
<source>
  type config_expander
  <config>
    type dstat
    tag host.dstat.__HOSTNAME__
    option  -tclmsgr -dD sda,sdb --disk-util -nN eth0
    delay 10
  </config>
</source>

<match host.dstat.**>
  type copy
  <store>
    type map
    map '["map." + tag + ".cpu_usr", time , "cpu_usr" => record["dstat"]["total_cpu_usage"]["usr"]]'
  </store>
  <store>
    type map
    map '["map." + tag + ".cpu_sys", time , "cpu_sys" => record["dstat"]["total_cpu_usage"]["sys"]]'
  </store>
  <store>
    type map
    map '["map." + tag + ".cpu_idl", time , "cpu_idl" => record["dstat"]["total_cpu_usage"]["idl"]]'
  </store>
  <store>
    type map
    map '["map." + tag + ".cpu_wai", time , "cpu_wai" => record["dstat"]["total_cpu_usage"]["wai"]]'
  </store>
   <store>
    type map
    map '["map." + tag + ".cpu_hiq", time , "cpu_hiq" => record["dstat"]["total_cpu_usage"]["hiq"]]'
  </store>
  <store>
    type map
    map '["map." + tag + ".cpu_siq", time , "cpu_siq" => record["dstat"]["total_cpu_usage"]["siq"]]'
  </store>
  <store>
    type map
    map '["map." + tag + ".loadavg_1m", time , "loadavg_1m" => record["dstat"]["load_avg"]["1m"]]'
  </store>
  <store>
    type map
    map '["map." + tag + ".mem_used", time , "mem_used" => record["dstat"]["memory_usage"]["used"]]'
  </store>
  <store>
    type map
    map '["map." + tag + ".mem_buff", time , "mem_buff" => record["dstat"]["memory_usage"]["buff"]]'
  </store>
  <store>
    type map
    map '["map." + tag + ".mem_cache", time , "mem_cache" => record["dstat"]["memory_usage"]["cach"]]'
  </store>
  <store>
    type map
    map '["map." + tag + ".mem_free", time , "mem_free" => record["dstat"]["memory_usage"]["free"]]'
  </store>
  <store>
    type map
    map '["map." + tag + ".io_sda_read", time , "io_sda_read" => record["dstat"]["io/sda"]["read"]]'
  </store>
  <store>
    type map
    map '["map." + tag + ".io_sda_write", time , "io_sda_write" => record["dstat"]["io/sda"]["writ"]]'
  </store>
  <store>
    type map
    map '["map." + tag + ".io_sdb_read", time , "io_sdb_read" => record["dstat"]["io/sdb"]["read"]]'
  </store>
  <store>
    type map
    map '["map." + tag + ".io_sdb_write", time , "io_sdb_write" => record["dstat"]["io/sdb"]["writ"]]'
  </store>
  <store>
    type map
    map '["map." + tag + ".dsk_sda_read", time , "dsk_sda_read" => record["dstat"]["dsk/sda"]["read"]]'
  </store>
  <store>
    type map
    map '["map." + tag + ".dsk_sda_write", time , "dsk_sda_write" => record["dstat"]["dsk/sda"]["writ"]]'
  </store>
  <store>
    type map
    map '["map." + tag + ".dsk_sdb_read", time , "dsk_sdb_read" => record["dstat"]["dsk/sdb"]["read"]]'
  </store>
  <store>
    type map
    map '["map." + tag + ".dsk_sdb_write", time , "dsk_sdb_write" => record["dstat"]["dsk/sdb"]["writ"]]'
  </store>
  <store>
    type map
    map '["map." + tag + ".sda_util", time , "sda_util" => record["dstat"]["sda"]["util"]]'
  </store>
  <store>
    type map
    map '["map." + tag + ".paging_in", time, "paging_in" => record["dstat"]["paging"]["in"]]'
  </store>
  <store>
    type map
    map '["map." + tag + ".paging_out", time, "paging_out" => record["dstat"]["paging"]["out"]]'
  </store>
  <store>
    type map
    map '["map." + tag + ".swap_used", time, "swap_used" => record["dstat"]["swap"]["used"]]'
  </store>
  <store>
    type map
    map '["map." + tag + ".net_eth0_recv", time , "net_eth0_recv" => record["dstat"]["net/eth0"]["recv"]]'
  </store>
  <store>
    type map
    map '["map." + tag + ".net_eth0_send", time , "net_eth0_send" => record["dstat"]["net/eth0"]["send"]]'
  </store>
</match>

<match map.host.dstat.**>
  type record_reformer
  enable_ruby true
  tag  timeadd.map.host.dstat
  <record>
    hostname ${tag_parts[3]}
  </record>
</match>

<match timeadd.map.host.dstat.**>
  type typecast
  item_types cpu_usr:float,cpu_sys:float,cpu_idl:float,cpu_wai:float,cpu_hiq:float,cpu_siq:float,loadavg_1m:float,mem_used:integer,mem_free:integer,mem_cache:integer,mem_buff:integer,io_sda_read:float,io_sda_write:float,io_sdb_read:float,io_sdb_write:float,dsk_sda_read:float,dsk_sda_write:float,dsk_sdb_read:float,dsk_sdb_write:float,sda_util:float,paging_in:float,paging_out:float,swap_used:integer,net_eth0_recv:float,net_eth0_send:float
  tag  typecast.timeadd.map.host.dstat
</match>

<match typecast.timeadd.map.host.dstat.**>
  type elasticsearch
  type_name       dstat
  host            localhost
  port            9200
  logstash_format true
  logstash_prefix logstash
  flush_interval  10s
</match>
```

## ダッシュボード
良い感じですがElasticSearchとKibanaはどちらもめちゃくちゃメモリを食います. 🐘

![Kibana](/images/2016-06/26-kibana_screen_shot.png "Kibana")

