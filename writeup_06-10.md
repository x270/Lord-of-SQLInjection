## 06 - darkelf
今度は`and`と`or`が禁止されている。  
05と同じ方法が使えるのでそのまま実行する。

### 回答例
```
?pw=0'||id='admin
```
> query : select id from prob_darkelf where id='guest' and pw='0'||id='admin'
