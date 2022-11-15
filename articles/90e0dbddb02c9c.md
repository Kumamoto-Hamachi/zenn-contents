---
title: "gRPCの動作確認にはBloomRPCとevansが便利！という話"
emoji: "🌸"
type: "tech"
topics: ["grpc", "bloomrpc", "evans"]
published: true
---
![](https://storage.googleapis.com/zenn-user-upload/58df7747b55b-20221115.png)


# 1. はじめに
gRPCの動作確認、みなさんはどうされてますか？
サービスのメソッド1つ1つの確認のためにクライアントをわざわざ実装するのは面倒ですよね。

今回はクライアントの実装よりもずっと楽かつ便利にgRPCの動作確認を出来る[BloomRPC](https://github.com/bloomrpc/bloomrpc)との[evans](https://github.com/ktr0731/evans)それぞれの良さ・使い分けについて自分の考えを言語化してまとめてみました。

# 2. BloomRPC
BloomRPCはGUIで直感的に操作可能な作りになっています。
インストールも簡単でMacならHomebrewで簡単に入れることが出来ます。

```
$ brew install --cask bloomrpc
```

またLinuxでもAppImage形式^[AppImageについてはこちらの[LinuxでAppImage形式のアプリを使う方法と注意点のまとめ | virtualiment](https://virment.com/how-to-use-appimage-linux/)
の記事の解説が分かりやすく、参考になります。]でファイルが配布されています。

[リンク先のReleases](https://github.com/bloomrpc/bloomrpc/releases)から拡張子が`.AppImage`になっているファイルをダウンロードした上で実行権限を付与してやるだけで良いので、非常にお手軽に使い始められます。

```
-- ダウンロードしたディレクトリに移動
$ cd {BloomRPCのAppImageダウンロード先パス}
-- 実行権限を付与
$ chmod +x ./BloomRPC-{バージョン}.AppImage
-- 実行
$ ./BloomRPC-{バージョン}.AppImage
```

## BloomRPCの良さ
### IDLの型から良い感じにパラメーターを埋めておいてくれる
BloomRPCはproto等のIDLのファイルのmessageの型などからパラメーターを良い感じに埋めておいてくれる機能があります。

![](https://storage.googleapis.com/zenn-user-upload/732407a47bdf-20221115.png)

これのおかげで特に重要でない項目に関してはBloomRPCの埋めてくれた値をそのまま流用して動作確認を行えるので非常に楽です。

WebフレームワークでのテストなどでRailsの[Factory Girl](https://github.com/thoughtbot/factory_bot_rails)やDjangoの[factory_boy](https://factoryboy.readthedocs.io/en/stable/index.html)がテストデータを良い感じに用意してくれる良さに近いです。

### streamingでの通信方式の確認やポート番号変更、metadataの設定等の操作性がシンプル

双方向通信(Bidirectional streaming RPC)^[gRPCではデータのやり取りをunary(単一)でやるかstreamingでやるかの2種類あり、それをclientとserverそれぞれがどちらを選択するかによって4つの通信方式のどれに分類されるかが決まります。今回例にしているBidirectional streaming RPCという通信方式の場合、client(request)もserver(response)も両方streamingの通信を行います。]を例にとってやると、

![](https://storage.googleapis.com/zenn-user-upload/502df72ed42c-20221115.gif)

上記のようにrequest、responseそれぞれのstreamのデータをタブ形式で切り替えて見れるのが非常に手軽です。

またタブ切り替えで別のポート番号のgRPCの確認なども容易で切り替えが出来ます。

さらにmetadata^[特定の RPC 呼び出しに関する情報(認証の詳細など)。キーと値のペアのリスト形式で提供される。]の設定も下のタブから簡単に行うことが出来ます。

![](https://storage.googleapis.com/zenn-user-upload/6b2e3aeb079f-20221115.png)


# 3. evans
evansはCLIのgRPCクライアントツールの1つです。REPL(対話)モードとCLIモードの2つを切り替えることが出来ます。
evansは基本的にBloomRPCに出来ることは何でも出来る(+BloomRPCには出来ないgRPCリフレクションにも対応出来る)ので、CLIツールの方がスキな方にはevansがオススメです。

MacならHomebrewで簡単に導入できます。

```
$ brew tap ktr0731/evans
$ brew install evans
```

Linuxだと[Relases](https://github.com/ktr0731/evans/releases)からファイルをダウンロードして解凍してやるとすぐ使えます。

```
-- ダウンロード後
$ tar -zxvf evans_linux_amd64.tar.gz
$ mv evans ~/.local/bin/
```

## evansの良さ
### REPLモードでの補完機能が優秀

evansは対話モードで実行した際、かなり気の利いた感じで補完を提案してくれるのでかなり楽にツール操作を行うことが出来ます。

![](https://storage.googleapis.com/zenn-user-upload/68edd7b129ae-20221116.png)


### gRPCリフレクション(Server Reflection)への対応
BloomRPC^[2022年11月16日現在も対応継続中のよう。[Add support for GRPC Server Reflection Protocol · Issue #1 · bloomrpc/bloomrpc](https://github.com/bloomrpc/bloomrpc/issues/1)]及びgRPC公式のCLIツールの[gprc_cli](https://github.com/grpc/grpc/blob/master/doc/command_line_tool.md)が未対応のgRPCリフレクションもevansなら対応済です。

gRPCリフレクションを利用する際は`-r`オプション(`--reflection`)を付けます。
CLIモードでの例を下記に示します。

```
-- サービスの列挙
$ evans -r cli list --port 1234
ClubManager
UserManager
grpc.reflection.v1alpha.ServerReflection
-- メソッドの列挙
$ evans -r cli list --port 1234 UserManager
UserManager.AddUser
UserManager.CountAlreadyUsers
UserManager.GetUser
UserManager.GetUsersByIds
UserManager.GetUsersByType
-- 特定のメソッドの定義確認
$ evans -r cli desc --port 1234 UserManager.AddUser
UserManager.AddUser:
rpc AddUser ( .User ) returns ( .UserResponse );
```

REPLモードでも同様のことが行えます。

![](https://storage.googleapis.com/zenn-user-upload/2b0c4c177eb3-20221116.png)



# 4. 参考
- [Core concepts, architecture and lifecycle | gRPC](https://grpc.io/docs/what-is-grpc/core-concepts/)
- [gRPC Reflection — gRPC Python 1.46.2 documentation](https://grpc.github.io/grpc/python/grpc_reflection.html)
- [gRPC と gRPC クライアントツール Evans | メルカリエンジニアリング](https://engineering.mercari.com/blog/entry/grpc_and_evans/)
- [LinuxでAppImage形式のアプリを使う方法と注意点のまとめ | virtualiment](https://virment.com/how-to-use-appimage-linux/)
- [Add support for GRPC Server Reflection Protocol · Issue #1 · bloomrpc/bloomrpc](https://github.com/bloomrpc/bloomrpc/issues/1)
