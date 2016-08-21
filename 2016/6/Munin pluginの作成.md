title: Munin pluginの作成
date: 2016-06-25 21:00:00
author: me
preview: Plugin::Munin, Plugin::Munin::Frameworkを使ってMunin pluginを自作する
tags:
   - perl
   - github
   - munin
   - softether

---

Softetherの転送量をMuninでグラフ化する<a href="https://github.com/dozen/softether-munin" target="_blank">softether-plugin</a>を作りました.

今回は `Plugin::Munin`, `Plugin::Munin::Framework` を使いましたが, これらのモジュールを使った自作プラグインの作り方について解説しているサイトが全然見つからなかったので, 紹介します.

情報少ないしこのモジュールはMunin v2.1以上じゃないと動かないし, 後述する `negative` を指定したグラフがたまにうまく描画できなかったりと, プラグインの作成方法としては結構微妙であることを先に書いておきます.

## Plugin::Munin::Frameworkについて
softether-pluginはPlugin::Munin::Frameworkというモジュールを使っていますが, これについての詳細はよくわかりません.
というのも, このプラグインを書いたのは1年前で, 当時参考にしていたWebサイトが見つからなかったからです.

Munin公式のプラグインで使われているようですがとにかくWeb上に情報が載ってないです.
munin 2.1.9-1では少なくともMunin公式のプラグインで `Munin::Plugin` , `Munin::Plugin::Framework` が使われています.
しかし munin 2.0.25 ではそのようなプラグインは使用されておらず, またそれらを使ったプラグインは動きません. CPANにはこの2つのモジュールは無かったので, これらのモジュールを使いたい場合は対応したバージョンのmuninをインストールするしかなさそうです.

そんなよくわからないものを使って大丈夫なのかよという感じですが, 動くし, 簡単にプラグインが書けるっぽいのでよしとします.

## 自作プラグインで実現したいこと
プラグインを自作する上で実現したいのは以下の3つです.

1. softether_myhub のようなシンボリックリンクを張るとmyhubのグラフを作ってくれる
1. Softetherのホスト名とかHubのパスワードはmunin-node.conf.d/munin-nodeにかける
1. Virtual HUB単位で転送量, パケット数をグラフ化

グラフ化できることはもちろんですが, Muninのプラグインらしく, シンボリックリンクで監視対象のデバイスを指定したり, munin-node.conf.d/munin-node に書いたconfigが読めるようにしたいです.

## プラグインの作成
softether-muninのソースコードを使って解説していきます.

### シンボリックリンク名でVirtual Hubを指定
Muninのプラグインでありがちな, 実体が `if_` というファイルで `if_eth0` というシンボリックリンクを張るとeth0についてのグラフを作成する, というのを真似したい.

プラグイン `softether_` のシンボリックリンク名を読み取る部分はこうなります.

``` perl
my $hub;

if ($0 =~ /softether_([\w\d#-_.&]+)$/) {
  $hub = $1;
} else {
  die ("Can't run config without a symlinked name");
}
```

これで `softether_myhub` というシンボリックリンクを張れば `$hub` に `myhub` が入り, あとで好きに使えます.

### munin-node.conf.dにあるConfigを読み込む
ホスト名, パスワード, vpncmdのパスを設定ファイルから読み込めるようにします.

ソースコードはこんな感じです.

``` perl
unless (defined $ENV{host}) {
  $ENV{host} = "localhost";
}

unless (defined $ENV{password}) {
  die ("Please set password");
}

unless (defined $ENV{vpncmdpath}) {
  $ENV{vpncmdpath} = "/usr/local/vpnserver/vpncmd";
}
```

対応するmunin-node.conf.dの設定は以下のようになります

```
[softether_*]
env.password myvpnpassword
env.host localhost
env.vpncmdpath /usr/local/vpnserver/vpncmd
```

ホスト名とvpncmdのパスはデフォルト値をプラグインのソースコードで指定しているので, 設定ファイルには書かなくても動きます.
設定ファイルに `env.password` と書いた場合はPerlの場合だと `$ENV{password}` みたいな感じでアクセスできるみたいです.

### 値の取得とか, 描画したいグラフの設定

``` perl
my @item_key = (
"Outgoing Unicast Packets",
"Outgoing Unicast Total Size",
"Outgoing Broadcast Packets",
"Outgoing Broadcast Total Size",
"Incoming Unicast Packets",
"Incoming Unicast Total Size",
"Incoming Broadcast Packets",
"Incoming Broadcast Total Size",
);

my $result = `$ENV{vpncmdpath} $ENV{host} /SERVER /password:$ENV{password} /hub:$hub /cmd StatusGet`;

my %data;

sub get_value {
  my $tmp;
  my $key = shift;
  if ($result =~ /$key\s*\|([0-9,]+)/) {
    $tmp = $1;
    $tmp =~ s/,//g;
    return $tmp;
  }
}

for my $item (@item_key) {
  $data{$item} = get_value($item);
}
```

こんな感じです.  `$result` には以下のような結果が入るので, 欲しい値を `@item_key` に登録しておいて取って来ています

```
StatusGet
vpncmd command - SoftEther VPN Command Line Management Utility
SoftEther VPN Command Line Management Utility (vpncmd command)
Version 4.19 Build 9605   (English)
Compiled 2016/03/06 21:09:57 by yagi at pc30
Copyright (c) SoftEther VPN Project. All Rights Reserved.

Connection has been established with VPN Server "localhost" (port 443).

You have administrator privileges for Virtual Hub 'vpn' on the VPN Server.

VPN Server/vpn>StatusGet
StatusGet command - Get Current Status of Virtual Hub
Item                         |Value
-----------------------------+---------------------
Virtual Hub Name             |vpn
Status                       |Online
Type                         |Standalone
SecureNAT                    |Enabled
Sessions                     |2
Sessions (Client)            |1
Sessions (Bridge)            |0
Access Lists                 |0
Users                        |1
Groups                       |0
MAC Tables                   |3
IP Tables                    |3
Num Logins                   |54
Last Login                   |2016-06-25 19:12:43
Last Communication           |2016-06-25 21:22:15
Created at                   |2016-04-04 00:12:46
Outgoing Unicast Packets     |157,138,736 packets
Outgoing Unicast Total Size  |133,078,817,603 bytes
Outgoing Broadcast Packets   |477,735 packets
Outgoing Broadcast Total Size|31,355,040 bytes
Incoming Unicast Packets     |157,166,519 packets
Incoming Unicast Total Size  |133,081,898,141 bytes
Incoming Broadcast Packets   |2,833,614 packets
Incoming Broadcast Total Size|174,223,220 bytes
The command completed successfully.
```

### Munin::Plugin::Framework でグラフ作成
Muninのプラグインは引数にconfを渡した時はグラフの設定を吐かないといけなかったりとか色々あるんですが, そのへんをよしなにやってくれるのが `Munin::Plugin::Framework` です. だと思います.

これを使えば設定ファイルを書くイメージでサクサクっとプラグインを作れます.

``` perl
my $plugin = Munin::Plugin::Framework->new;

$plugin->add_graphs(
  packets => {
    args => "",
    category => $category,
    info => "$hub Packets",
    title => "$hub Packets",
    vlabel => "in (-) | out (+) packets / second",
    fields => {
      unicast_out => {
        label => "Unicast",
	type => "DERIVE",
	min => 0,
	negative => "unicast_in",
	value => $data{"Outgoing Unicast Packets"}
      },
      unicast_in => {
        type => "DERIVE",
	min => 0,
	value =>  $data{"Incoming Unicast Packets"}
      }
    }
  },
  unicast_total_packets => {
    args => "",
    category => $category,
    info => "$hub Unicast Toatl Packets",
    title => "$hub Unicast Total Packets",
    vlabel => "packets",
    fields => {
      incoming => {
        label => "Incoming",
        type => "GAUGE",
        min => 0,
        value => $data{"Incoming Unicast Packets"}
      },
      outgoing => {
        label => "Outgoing",
        type => "GAUGE",
        draw => "AREA",
        min => 0,
        value => $data{"Outgoing Unicast Packets"}
      }
    }
  }
);

$plugin->run();
```

こんな感じで `add_graphs()` でどんどん追加していくだけ.
カテゴリは `softether` とかにしてます.

- args: グラフの下限値, 上限値, 対数スケールでプロット などを指定するところ
- category: [ disk munin network processes system time ]みたいなの, あれです. すでにあるものに追加したりとか, `softether` みたいに新しいカテゴリを作ることも出来ます
- vlabel: グラフの縦軸の表示名.
- fields: 描画したいデータについて記述する
  - type: `DERIVE` だと前の値との差, `GAUGE` だと値をそのまま出力
  - negative: NICのスループットとかのグラフでoutが上方向, inが下方向にグラフが伸びてますが, あれを実現するためのものです. `negative` に指定したキーのフィールドがマイナス方向に描画されるようになります.

だいたいこんな感じなんですが, `Munin::Plugin::Framework` で `negative` を指定するとたまーにグラフの生成に失敗します.
RRDToolがエラーを吐いてますが, 原因はよくわかりません. この辺はちょっと微妙な感じですね.
`negative` を使わなければいいんですがSoftetherプラグインでグラフ化したいものはおおかたIn/OutがあるのでOutを上方向, Inを下方向に伸ばすグラフのほうがしっくり来ますし.

### プラグインの全体図, 生成されるグラフ
<a href="https://github.com/dozen/softether-munin" target="_blank">github</a>を見てください. 🐘
