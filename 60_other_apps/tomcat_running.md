## Tomcatのパスワードが脆弱なため任意のコードを実行できる問題
### 対象
Apache

### 危険度
High
* IDとパスワードが脆弱で、容易に推定できる
* この脆弱性を利用することでJavaで書かれた任意のコードを実行できる

### 解説
今回のサーバではApacheが動作しており、その上で各種アプリケーションが動作するようになっている。
その設定ファイルに以下の記述がある。
```apache
<VirtualHost *:80>
ServerName kanri-bank.mbsd.jp
ProxyPass / ajp://localhost:8009/
#DocumentRoot /var/www/html/kanri/

#<Directory "/var/www/html/kanri/">
#AllowOverride All
#</Directory>
</VirtualHost>
```

この記述は、`kanri-bank.mbsd.jp`として8009番ポートで動作しているTomcatにアクセスするようにする設定である。
しかし、このTomcatのユーザ設定に問題があり、大変危険なユーザ名・パスワードでログインできてしまう。

`/etc/tomcat/tomcat-users.xml`
```xml
<user name="admin" password="admin" roles="admin,manager,admin-gui,admin-script,manager-gui,manager-script,manager-jmx,manager-status" />
```
このため、悪意のある人が容易にアカウントを推測しログインすることができる。

また、このユーザの権限が強い影響で、管理画面からJavaのプログラムパッケージであるWARファイルを転送し、任意のJavaプログラムを実行できてしまう。

### 再現方法
1. 利用しているネットワークのDNSサーバの設定を変更するか、`/etc/hosts`といったhostsファイルの変更を行い、`kanri-bank.mbsd.jp`が当該サーバのIPアドレスを示すよう設定する
2. そのドメインでサーバにアクセスする
3. ユーザ名`admin`, パスワード`admin`でログインできる

### 条件
無

### 想定される被害・影響
* 当該サーバにおいて攻撃者が任意のコードを実行できる
* 読み込みアクセス権のあるファイル全ての読み込み、書き込みアクセス権のあるファイル全ての書き込み（改ざん）が可能
  - 機密ファイルの持ち出し等が考えられる

### 対策
* ユーザIDとパスワードを厳格に設定する
* ユーザのアクセス権限を最低限のものに絞る
* Tomcatを使っていないのであればサービスを終了させ、動かさないようにする
