title: Shift_JISのXMLをGoでパースする
date: 2016-10-18 06:24:00
author: me
preview: Shift_JISのXMLファイルをencoding/xmlで扱う方法
tags:
   - XML
   - golang

---

# Go言語におけるXML文書の扱い方

UTF-8のXML文書の扱い方についてはここを参考にしました．

http://qiita.com/ono_matope/items/70080cc33b75152c5c2a

# Shift_JISのXML文書

上のリンクを参考にしてXMLファイルをパースしようと思ったら，ファイルの文字コードがShift_JISだったのでうまくいきませんでした．


```
<?xml version="1.0" encoding="Shift_JIS"?>
<user>
	<Name>太郎ちゃん</Name>
	<attr>Male</attr>
	<age>20</age>
</user>
```

このようなXMLファイルを次のようなコードでパースしようとしましたら失敗しました．

```
package main

import (
	"os"
	"fmt"
	"encoding/xml"
)

func main() {
	fp, _ := os.Open("taro.xml")
	defer fp.Close()

	var user User
	decoder := xml.NewDecoder(fp)
	err = decoder.Decode(&user)
	if err != nil {
		fmt.Println(err.Error())
	}

	fmt.Printf("%#v", user)
}

type User struct {
	XMLName xml.Name `xml:"user"`
	Name string
	Gender string `xml:"attr"`
	Age int `xml:"age"`
}
```

```
xml: encoding "Shift_JIS" declared but Decoder.CharsetReader is nil
```

`Decoder.CharsetReader`がnilだと怒られています．なんとなくですが`xml.NewDecoder()`で受け取ったio.Readerとパーサの間に挟まって文字コードをUTF-8に変換してるんじゃないかなと思います．

`Decoder.CharsetReader`の型を見ると，`func(charset string, r io.Reader) (io.Reader, error)`となっています．よくわからないけど`Decoder.CharsetReader`の戻り値が`transform.Reader`を返すような関数を実装すれば動きそうな感じがしたので，とりあえず書いてみたら動きました．

```
package main

import (
	"encoding/xml"
	"fmt"
	"golang.org/x/text/encoding/japanese"
	"golang.org/x/text/transform"
	"io"
	"os"
)

func main() {
	fp, err := os.Open("taro.xml")
	defer fp.Close()

	var user User
	decoder := xml.NewDecoder(fp)
	decoder.CharsetReader = func(charset string, r io.Reader) (io.Reader, error) {
		return transform.NewReader(r, japanese.ShiftJIS.NewDecoder()), nil
	}
	err = decoder.Decode(&user)
	if err != nil {
		fmt.Println(err.Error())
	}
	fmt.Printf("%#v", user)
}

type User struct {
	XMLName xml.Name `xml:"user"`
	Name    string
	Gender  string   `xml:"attr"`
	Age     int      `xml:"age"`
}
```

`charset`にエンコーディングのタイプが渡されるので，CharsetReaderの中でエンコーディングごとに適した処理を書くのが本来のやり方っぽいですが，今回はShift_JISのXMLファイルをパースしたいだけなのでcharsetは無視しています．

```
main.User{XMLName:xml.Name{Space:"", Local:"user"}, Name:"太郎ちゃん", Gender:"Male", Age:20}
```

これでShift_JISのXMLファイルをパースすることが出来ました．
