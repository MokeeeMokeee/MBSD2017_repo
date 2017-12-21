# IPv6のファイアウォールが無効になっている問題

### 対象
サーバシステム全体

### 危険度
Middle
* サーバがIPv6で接続されている場合、本来アクセスできてはいけないはずのサービスにアクセスされる可能性がある

### 解説
このシステムではIPv6のファイアウォールが実質存在していない。
これにより、本来アクセスできないはずのサービスにアクセスできてしまう。
アクセスされてしまう可能性のあるサービス
* 8080/TCPで動作しているJavaプログラム
* 8009/TCPで動作しているTomcat
* 3306/TCPで動作しているMySQL

### 再現方法
IPv6アドレスからMySQLサーバに接続する
ex) サーバのIPv6アドレスが`2408:210:c5ef:6f00::51dd:34:8931`の場合
`mysql -h [2408:210:c5ef:6f00::51dd:34:8931] -uroot -ppassword`

### 条件
本来アクセスしてはいけない場所にアクセスができる脆弱性が存在すること

### 想定される被害・影響
アクセス権に不備のあるサービスへの侵入を許してしまう

### 対策
IPv4と同様にiptablesの設定を行う
```sh
iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A INPUT -p icmpv6 -j ACCEPT
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 443 -j ACCEPT
iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 34567 -j ACCEPT
iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
iptables -A INPUT -p udp -m state --state NEW -m udp --dport 161 -j ACCEPT
iptables -A INPUT -j REJECT
iptables -A FORWARD -j REJECT
```