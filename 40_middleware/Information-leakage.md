## セッションハイジャックが出来てしまう脆弱性

### 対象

ユーザすべて

### 危険度

High

* 想定される被害・影響が非常に大きく、容易に確認出来るため
### 解説

apacheのドキュメントルートの中にあるセッションIDを見る事が出来るので、
なりすましが出来る

### 再現方法

* 以下のコマンドを打つ
  `$curl http://${SERVER_IP}/tmp/`
  以下の通りの返答が帰ってくる
```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
<html>
 <head>
  <title>Index of /tmp</title>
 </head>
 <body>
<h1>Index of /tmp</h1>
  <table>
   <tr><th valign="top"><img src="/icons/blank.gif" alt="[ICO]"></th><th><a href="?C=N;O=D">Name</a></th><th><a href="?C=M;O=A">Last modified</a></th><th><a href="?C=S;O=A">Size</a></th><th><a href="?C=D;O=A">Description</a></th></tr>
   <tr><th colspan="5"><hr></th></tr>
<tr><td valign="top"><img src="/icons/back.gif" alt="[PARENTDIR]"></td><td><a href="/">Parent Directory</a>       </td><td>&nbsp;</td><td align="right">  - </td><td>&nbsp;</td></tr>
<tr><td valign="top"><img src="/icons/unknown.gif" alt="[   ]"></td><td><a href="0a06202efd2cf39ddf2cfc187ea2f2999364b3d8">0a06202efd2cf39ddf2c..&gt;</a></td><td align="right">2017-10-29 19:44  </td><td align="right">113 </td><td>&nbsp;</td></tr>
<tr><td valign="top"><img src="/icons/unknown.gif" alt="[   ]"></td><td><a href="0a1b2fbea2cbd97b5343d58eae7a872f1de51690">0a1b2fbea2cbd97b5343..&gt;</a></td><td align="right">2017-10-29 19:44  </td><td align="right">113 </td><td>&nbsp;</td></tr>
<tr><td valign="top"><img src="/icons/unknown.gif" alt="[   ]"></td><td><a href="0a4ca1b8c6de41de6aa5bf02a7200e020ea11b02">0a4ca1b8c6de41de6aa5..&gt;</a></td><td align="right">2017-10-29 19:38  </td><td align="right">113 </td><td>&nbsp;</td></tr>
<tr><td valign="top"><img src="/icons/unknown.gif" alt="[   ]"></td><td><a href="0a5cfc30f55a27614ff83c91b695584f12990d3f">0a5cfc30f55a27614ff8..&gt;</a></td><td align="right">2017-11-02 11:40  </td><td align="right">114 </td><td>&nbsp;</td></tr>
<tr><td valign="top"><img src="/icons/unknown.gif" alt="[   ]"></td><td><a href="0a6b8056257ab8bc442589b6cb8e0cd987ae3c2a">0a6b8056257ab8bc4425..&gt;</a></td><td align="right">2017-11-02 11:41  </td><td align="right">114 </td><td>&nbsp;</td></tr>
<tr><td valign="top"><img src="/icons/unknown.gif" alt="[   ]"></td><td><a href="0a9bb9683439c72901ef9fa83c9670f63d4b5a27">0a9bb9683439c72901ef..&gt;</a></td><td align="right">2017-10-29 19:37  </td><td align="right">113 </td><td>&nbsp;</td></tr>
<tr><td valign="top"><img src="/icons/unknown.gif" alt="[   ]"></td><td><a href="0a22f8cc3a1d90207352c535bab20c628e37ae0a">0a22f8cc3a1d90207352..&gt;</a></td><td align="right">2017-11-03 14:03  </td><td align="right">114 </td><td>&nbsp;</td></tr>
<tr><td valign="top"><img src="/icons/unknown.gif" alt="[   ]"></td><td><a href="0a65e5cd5b82db62165500503a28b51045da3731">0a65e5cd5b82db621655..&gt;</a></td><td align="right">2017-11-02 11:42  </td><td align="right">114 </td><td>&nbsp;</td></tr>
<tr><td valign="top"><img src="/icons/unknown.gif" alt="[   ]"></td><td><a href="0a879c9a576b00d5e497e569280d6b7b891f37b1">0a879c9a576b00d5e497..&gt;</a></td><td align="right">2017-11-02 11:43  </td><td align="right">114 </td><td>&nbsp;</td></tr>
```

### 条件
特になし

### 想定される被害・影響
セッションハイジャックが可能

### 対策
* Apacheが当該ファイルを参照できないよう設定変更を行う
* Index指定を削除することでファイル一覧を取得できないようにする
