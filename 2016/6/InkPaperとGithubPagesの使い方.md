title: InkPaperとGithub Pagesの使い方
date: 2016-06-05 06:14:00
author: me
preview: InkPpaerでブログを生成して, Github Pagesにアップロードするまで
tags:
   - golang
   - github

---

## InkPaperの使い方
[InkPaperのREADME](https://github.com/InkProject/ink#introduce)を読めばだいたい分かる.

### InkPaperをインストール
[http://www.inkpaper.io/](http://www.inkpaper.io/)から自分の環境に合ったものをダウンロードします. zipファイルを解凍すればインストールは完了です.

### 設定ファイルを編集する
`blog/config.yml`を編集します.

```
site:
    title: ジェット・ゾウ #ブログのタイトル
    subtitle: サブタイトル #タイトルの下にチョロンと表示されます
    # logo: -/images/avatar.jpg #トップページに表示される自分のアイコン
    limit: 10 
    theme: theme
    disqus: somebody
    lang: en
    url: http://jetzou.com/
    # root: /blog

authors:
    me:
        name: dozen
        intro: どぜんと読みます
        avatar: -/images/avatar.jpg

build:
    port: 8000
    # Copied files to public folder when build
    copy:
        - theme/bundle
        - theme/favicon.ico
        - theme/robots.txt
        - source/images
    # Excuted command when use 'ink publish'
    publish: |
        git add . -A
        git commit -m "update"
        git push origin
```

`url`と`root`だけ気をつければあとは適当で大丈夫. 今回は`http://jetzou.com/`で公開したいので`url: http://jetzou.com/"`としました. `disqus`はコメント機能を提供してくれるDISQUSのアカウントを指定するんですが, 今回はスキップ.

`lang`は今のところ中国語と英語にしか対応していないみたいなので, `en`を指定しました.

設定ファイルはこんな感じで大丈夫そうです.

### 記事を書く
記事を置く場所は`blog/source`です.

インストールした状態ではサンプルページの`blog/source/ink-blog-tool-en.md`と`blog/source/ink-blog-tool.md`があります. 記事を書くまえにこのファイルをみて, 参考にするといいです. 公開する前にこの2つのファイルは消しておきます.

```
title: Article Title
date: Year-Month-Day Hour:Minute:Second #Created Time，Support TimeZone, such as " +0800"
update: Year-Month-Day Hour:Minute:Second #Updated Time，Optional，Support TimeZone, such as " +0800"
author: AuthorID
cover: Article Cover Path #Optional
draft: false #If Draft，Optional
top: false #Place article to top, Optional
preview: Article Preview，Also use <!--more--> to split in body #Optional
tags: #Optional
    - Tag1
    - Tag2

---

Markdown Format's Body
```

`date`は投稿日時, `update`は更新日時です. `update`は不要ですが`date`は必須です. トップページなどを生成する際に`date`をもとに記事をソートするためです. フォーマットは`2016-06-01 12:00:00`みたいな感じです. ちゃんと秒まで指定しないとエラーが出ます.

`author`は`blog/config.yml`で指定した`authors`を指定します. デフォルトだと`me`です. 

`cover`はよくわかんないです. アイキャッチかな?

`draft: true`を追加すると書きかけの記事とみなされて, ページが生成されません.

`top: true`とすると, トップページの一番上に張り付く記事になるみたいです. 使ったこと無いからわからないけど.

`preview`はトップページで表示される記事の概要を指定します. これを指定せずに, 記事の本体の適当な場所に`<!--more-->`を置くとその部分までを記事の概要として表示してくれます. ただし, Markdownを処理してくれないので`<!--more-->`は使わずに, `preview`を指定することをおすすめします.

`tags`で`PC`とか`食べ物`とか指定してあげるとタグごとのアーカイブページとか作ってくれます.

以上が記事の情報です. `---`より下が本文となります. ここから下はMarkdown記法が使えます.

これを踏まえると一番簡単な書式はこうなります.

```
title: タイトル
date: 2016-06-05 7:00:00
preview: テストです

---

テスト
```

これを`blog/source`の下に置いて, ファイル名を`タイトル.md`などとして保存してもいいのですが, それだと今後記事が増えた時にひっちゃかめっちゃかになる可能性が高いです.

そこで, `blog/source/年/月/日`というふうにサブディレクトリを作るのがおすすめです. サブディレクトリを作っても記事のURIには何の影響も出ないので, 好きなように変更できます. 私は今のところ`blog/source/年/月`くらいにしてます.

### ちゃんと生成されてるかチェックする
記事を書いたら, ちゃんとページが生成されるかテストしてみます. `./ink preview`というコマンドを実行することで, ページを生成して簡易的なサーバが起動します. `http://localhost:8000`でアクセスできます. ポート番号が8000だと都合がわるい場合は`blog/config.yml`で変更します.

### テーマを修正する
ブログが生成されているかチェックすると, あることに気づきます. 投稿した時刻がおかしいんです. 設定ファイルで英語を指定したはずなのに,`分钟前`とか表示されます. どうもこの投稿時刻の表示はJavascriptで行われており, ここだけ言語の設定が反映されないみたいです.

#### まじめに直す
テーマは`blog/theme`にあり, 編集すべきファイルは`blog/theme/source/index.coffee`です.

```
timeSince = (date) ->

    seconds = Math.floor((new Date() - date) / 1000)
    interval = Math.floor(seconds / 31536000)
    if interval > 1
        return interval + "年前"
    interval = Math.floor(seconds / 2592000)
    if interval > 1
        return interval + "个月前"
    interval = Math.floor(seconds / 86400)
    if interval > 1
        return interval + "天前"
    interval = Math.floor(seconds / 3600)
    if interval > 1
        return interval + "小时前"
    interval = Math.floor(seconds / 60)
    if interval > 1
        return interval + "分钟前"
    return Math.floor(seconds) + "秒前"
```

こんな記述があるので, 以下のように変更します.

```
timeSince = (date) ->

    seconds = Math.floor((new Date() - date) / 1000)
    interval = Math.floor(seconds / 31536000)
    if interval > 1
        return interval + "年前"
    interval = Math.floor(seconds / 2592000)
    if interval > 1
        return interval + "ヶ月前"
    interval = Math.floor(seconds / 86400)
    if interval > 1
        return interval + "日前"
    interval = Math.floor(seconds / 3600)
    if interval > 1
        return interval + "時間前"
    interval = Math.floor(seconds / 60)
    if interval > 1
        return interval + "分前"
    return Math.floor(seconds) + "秒前"
```

編集が終わったら, webpackを実行します.

`blog/theme`に移動して, 以下のコマンドを実行します.

```
npm install

./node_modules/webpack/bin/webpack.js
```

次に`ink`とか`blog`があるディレクトリに戻って, `./ink preview`を実行して確認してください. `10分前`みたいなかんじになっているはずです.

#### 適当に直す
nodeとか入れてないしjsファイル直接編集するだけでいいでしょ, って言う方はこっちで.

`blog/theme/bundle/index.js`の1行目に, `个月前`とか`天前`とか書いてある部分があるはずなので, 修正します.

`./ink preview` で変更が反映されていることを確認します.

### コマンドまとめ
記事を書いたら`./ink preview`で確認して, `./ink publish`でGithubにプッシュ, という流れになります. `./ink build`はテーマからcssやjsファイルをコピーしてくるコマンドですが, `preview`や`publish`のときにこのコマンドが走っているみたいなので自分で実行する必要がなく, 余り使いません


## Github Pagesの使い方
[https://help.github.com/categories/github-pages-basics/](https://help.github.com/categories/github-pages-basics/)を読めばだいたい分かる.

### リポジトリの作成とCNAMEの設定
githubで`blog`とか適当な名前のリポジトリを作ります. そうしたら, `blog`ディレクトリに移動して先ほど作ったリポジトリをcloneします. 次に`blog/public`を削除し, cloneしたリポジトリのを`public`にリネームします. リネームしたら`git checkout -b gh-pages`を実行します.

`blog/public/CNAME`ファイルを作成し, 使用するドメインをファイルの1行目に書き込みます. `jetzou.com`とか, `blog.jetzou.com`みたいな感じです. 書き込んだら, `git add CNAME`, `git commit -m "add CNAME"`というふうにCNAMEファイルを追加し,コミットしてください. 

コミットしたら, `git push --set-upstream origin gh-pages` を実行しておきます.

Custom Domainを使わずに`dozen.github.io/blog`みたいな感じで使う人は`CNAME`ファイルの設定は不要です.

### DNSの設定
Custom Domain使う人だけが必要な設定です. [https://help.github.com/articles/setting-up-an-apex-domain/<Paste>](https://help.github.com/articles/setting-up-an-apex-domain/)ここを参考にします.

参考にするページにも書いてありますが, CNAMEレコードは使わないようにします. で, 今回はALIAS, ANAMEレコード非対応のDNSサービスを利用するため, Aレコードを使った設定をします.

DNSに`192.30.252.153`と`192.30.252.154`のAレコードを追加すればいいっぽいので, 追加します.

### InkPaperの設定ファイルを修正する
`blog/config.yml`の`utl`を変更します. 必要な場合は`root`をアンコメントします.

### 記事をGithub Pagesに反映
ここまでの設定を終えて, 記事を書いたら`./ink publish`を実行します. これで, Github Pages上のブログにアクセスできるようになっているはずです

---

InkPaperとGithub Pagesの使い方はざっとこんな感じです.

InkPaperに関する日本語の記事があんまりないのが悲しいです. 日本で使ってる人が増えたらいいな. 🐘
