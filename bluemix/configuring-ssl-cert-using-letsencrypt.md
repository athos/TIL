# Bluemix用のSSL証明書をLet's Encryptで発行する

諸事情によってBluemixで構築しているWebアプリにSSL接続が必要になったので、Bluemix用のSSL証明書を用意した。その際、最近流行りのLet's Encryptで証明書を発行しようと思ったのだけど、Let's Encrypt自体は基本的には証明書を発行するサーバ上でLet's Encryptクライアントを実行することでドメイン所有者であることを認証するようなプロセスになっているらしい。

ところが、今のBluemix(というかClound Foundry)にはコンテナに直接ログインできるような仕組みがない[1]ようなので、Let's Encryptのこの認証プロセスはBluemixに対してそのままの形では適用できない。結論としては、Let's Encryptのマニュアル認証モードを使って認証を済ませたが、いくつか躓いたところもあるので備忘録も兼ねてまとめておく(ちなみに、`cf`コマンドのプラグインでログイン可能にするものもあるらしいそちらでLet's Encryptを実行した方が手っ取り早いのかもしれない)。

SSL証明書発行までの全体のステップは以下のとおり：

1. 所有するドメインのDNSレコードへBluemixのIPアドレスを登録する
2. Bluemixの経路に所有するドメインを追加する
3. Let's Encryptのマニュアルモードで証明書を発行する
4. Bluemixの経路情報に証明書を追加する

## 所有するドメインのDNSレコードへBluemixのIPアドレスを登録する
Let's EncryptではDNSのAレコードに証明書を発行するサーバのIPアドレスが登録されていることが要求される。Bluemixでカスタムドメインを設定する場合には、地域ごとにあらかじめ決められているIPアドレスを指定することになっているので、そのIPアドレスを所有するドメインのAレコードに登録してやる[2]。最初このIPアドレスが分からずに設定に手こずってた。

- US-SOUTH: 75.126.81.68
- EU-GB: 5.10.124.142
- AU-SYD: 168.1.35.166

## Bluemixの経路に所有するドメインを追加する
ステップ1.でDNSの設定をしたドメインをBluemixの経路へ設定する。アプリケーションの「経路」の編集ボタンで開くダイアログから「ドメインの管理」を選び、カスタムドメインに追加する。一旦設定を保存するとSSL証明書を追加できるようになるので、ステップ3.で証明書を発行した後にここに戻ってくる。

## Let's Encryptのマニュアルモードで証明書を発行する
さて、ようやくLet's Encrypt。詳細は[ここ](https://xomino.com/2016/02/09/using-lets-encrypt-to-create-an-ssl-certificate-for-my-bluemix-hosted-web-site/)に書いてあるとおり、Let's Encryptのマニュアル認証モードで証明書を発行する。基本的にはチャレンジレスポンス方式で、URLのパスとして渡ってくるトークンに対して対応するレスポンスを返すようなちょっとしたWebアプリを用意してやる必要がある。

```
$ letsencrypt-auto certonly --manual --email <連絡先メールアドレス> -d <設定したカスタムドメイン>
```

上記のコマンドを実行すると、以下のようなメッセージが表示される：

```
Make sure your web server displays the following content at
http://<設定したカスタムドメイン>/.well-known/acme-challenge/BmlHW8d1U5e9x8MhwC_l7R_2ypL2g9tnRnblPXiwqrE before continuing:

BmlHW8d1U5e9x8MhwC_l7R_2ypL2g9tnRnblPXiwqrE.1s48hqb8e1PR91xWLMQ_u2nU7dwfLKuQtwzTpbHZGGE
```

メッセージが示しているとおり、`/.well-known/acme-challenge/BmlHW8d1U5e9x8MhwC_l7R_2ypL2g9tnRnblPXiwqrE`というパスにリクエストが来るので、それに対して`BmlHW8d1U5e9x8MhwC_l7R_2ypL2g9tnRnblPXiwqrE.1s48hqb8e1PR91xWLMQ_u2nU7dwfLKuQtwzTpbHZGGE`をレスポンスを返してやるようにする。やりかたはいろいろあるだろうけど、今回はあらかじめBluemixで動いているWebアプリがある状態だったので、そのWebアプリにレスポンスを登録するAPIを追加して、`/.well-known/acme-challenge/:id`に対するリクエストが来たらその登録したレスポンスを返すようにWebアプリを一時的に修正した。

証明書の発行に成功すると、`/etc/letsencrypt/live/<カスタムドメイン>`あたりに証明書が生成されるのでそのファイルを使ってステップ4.を進める。

## Bluemixの経路情報に証明書を追加する
ステップ3.で発行したcert.pemとprivkey.pemを(実際にはこれらのファイルはシンボリックリンクなのでリンクの先の実体ファイルを)ステップ2.で設定したカスタムドメインに付加する。

[1]: http://qiita.com/KenichiSekine/items/f287ccb8a9c988edc160
[2]: https://console.ng.bluemix.net/docs/manageapps/secapps.html
