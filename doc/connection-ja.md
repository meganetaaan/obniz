# obniz connection

obnizクラスはobnizをjavascriptの中で抽象化したクラスです。
idを指定してインスタンス化することで通信内容を意識せずにobnizやobnizに繋がれた部品の操作ができるようになります。

## basics

obnizはidをもとにインスタンス化します。
HTMLの場合はscriptタグで読み込むだけでObnizクラスが利用できます。nodejsの場合はnpmでインストールした後に、

```javascript
var Obniz = require('obniz');
```
このようにクラスを取得できます。

そして、接続して利用したいobnizのidを与えてインスタンス化します。
```javascript
var obniz = new Obniz('1234-5678');
```
２台以上のobnizを使いたい場合は
```javascript
var obnizA = new Obniz('1234-5678');
var obnizB = new Obniz('0000-0000');
```
のように記載することになります。

インスタンス化した段階で[obniz Websocket API](https://obniz.io/doc/about_obniz_api)を使いobnizとの接続を行います。
接続が完了するとonconnectが呼ばれます。また、oncloseは切断時に呼ばれます。
標準で自動接続が行われますので、切断が起きても、継続的に接続を試みます。

```javascript
var obniz = new Obniz('1234-5678');
obniz.onconnect = async function() {

}
obniz.onclose = async function() {

}
```

ioのon,offなどは接続することで操作できるようになりますので、onconnectの中でobnizに対して行いたいことを記載します。

```javascript
var obniz = new Obniz('1234-5678');
obniz.onconnect = async function() {
  obniz.io0.output(true);
}
```

インスタンス化するときのオプションもありますので、下のそれぞれの関数リファレンスを御覧ください。

## new Obniz('obniz id', { options })

obnizインスタンス化します。
obniz idに半角文字列でobnizの番号を入力します。ハイフン(-)はあってもなくても大丈夫です。数値では認識できません。

```javascript
new Obniz('1234-5678') // OK
new Obniz('12345678') // OK
new Obniz(12345678) // Can't accept
```

また、obnizにアクセストークンを設定しているobnizの場合は以下のようにオプションで指定します。

```javascript
new Obniz('1234-5678', {access_token: 'your token here'})
```

obniz idとして正しくないと判断された場合は接続は行われません。nodejsの場合はエラーとなります。
ブラウザ上で実行している場合はpromptが表示されます。そこでidを入れることでそのobnizに接続することが可能です。
これは正しくない形式の場合にのみ表示されます。正しくても存在しないobniz idを入れた場合には表示されません。

![](images/obniz_prompt.png)

idが正しければ、クラウドに接続を行い、接続が完了するとonconnectが呼ばれます。

ただし、obnizとobniz.jsを利用している端末が同一LAN内にある可能性が高い場合、obniz.jsは同時にobnizと直接Websocket接続を行おうとします。これをlocal connectといいます。
local connectできた場合はほとんどのコマンドで、クラウドを経由せずにobnizを操作可能です。ただし、local connectしていてもクラウドとの接続は切断されません。
また、クラウドとの接続が切断されるとlocal connectも切断されます。

![](images/local_connect.png)

onconnectが呼ばれるタイミングは、クラウドにつながったとしてもlocal_connectが利用できるかどうかによって変わってきます。できるだけlocal_connect経由になるように接続を少しまちます。
以下のような流れとなります。

![](images/onconnect_flow.png)

obnizをインスタンス化するときの第二引数はオプションです。
そのoptionで以下の設定が指定できます。

name | type | default | description
--- | --- | --- | ---
binary | `boolean` | true | APIの通信ではjsonでなく圧縮形式が使えますが、それのon-offとなります。falseにした場合local_connectは利用できません。
local_connect | `boolean` | true | obniz.js はクラウドAPI経由でobnizとつないだあとに可能であればobnizと直接接続しようとします。falseにすることでそれを使わない設定にできます。binaryがfalseの場合はlocal_connectは自動的にfalseになります。
debug_dom_id | `string` | 'obniz-debug' | HTMLではここで指定されたidを持つDOMにオンラインステータスなど各種情報が出力されます
auto_connect | `boolean` | true | 標準でobniz.jsは自動的に接続を行い、切れても再接続を自動で行いますが、これによりoffにできます。
access_token | `string` | null | access_tokenが発行されているobnizに接続する場合は指定してください。
reset_obniz_on_ws_disconnection | `boolean` | true | obniz.jsとクラウドとの接続が切れた時にクラウドがobnizをリセットするかどうかを決められます。


## connect()
optionのauto_connectをfalseにしている場合に利用できます。connectを呼ぶことでobnizとの接続を試みます。

```javascript
var obniz = new Obniz('1234-5678', { auto_connect: false });

obniz.connect();
obniz.onconnect = async function() {
  obniz.io0.output(true);
}
```

## close()
現在の接続を切断します。
auto_connectがfalseになっていないと、また自動的に再接続されます。

```javascript
var obniz = new Obniz('1234-5678', {
  auto_connect: false,
  reset_obniz_on_ws_disconnection: false
  });

obniz.connect();
obniz.onconnect = async function() {
  obniz.io0.output(true);
  obniz.close();
}
```

## debugprint
通信でやりとりされるjsonを出力したり、接続状態などのobniz.js内のログがconsole.logに出力されます。

```javascript
var obniz = new Obniz('1234-5678');
obniz.debugprint = true
obniz.onconnect = async function() {
  obniz.io0.output(true);
}
```

## resetOnDisconnect(reset)
オプションである ```reset_obniz_on_ws_disconnection``` の設定をあとから変更する場合に使う関数です。
obniz.jsでは標準でtrueなので、obniz.jsはユーザーがobniz cloudへのweb socketを切断するとリセットするようになっています。
リセットするので、出力されている電圧などももとに戻り、pwmなども全て停止します。
この関数でそれを無効にし、リセットしないようにできます。
この設定はユーザーのweb socketが切断されるまで保持されます。
```Javascript
// Example
obniz.resetOnDisconnect(false);
```