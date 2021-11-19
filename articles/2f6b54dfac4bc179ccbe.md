---
title: "[CentOS/Ubuntu/Macに]ag(the silver searcher)をインストール"
emoji: "🏹"
type: "tech"
topics: ["ubuntu", "centos", "mac", "ag", "grep"]
published: true
---


# この記事を書いた理由

※本記事は[こちらのQiita記事](https://qiita.com/digitalhimiko/items/dc5f797e4d05c97ab8cd)を筆者本人が転載したものです。

ファイルの中身を検索できるコマンドと言えば『grep』ですが、そのgrepを高速化した『grab』や『ack』、『ag』等が登場し移行する人が出てきています。

今回はその中で特に早いと噂[^1]の**ag**を、VirtualBox/Vagrantで仮想構築したCentOS8上に[GitHubのドキュメント](https://github.com/ggreer/the_silver_searcher)に沿って下記のコマンドでインストールしようとしました。しかし...

```
-- yumでも結果は同じ
$ sudo dnf update
$ sudo dnf install the_silver_searcher
-- 下記のエラーが出てインストール出来ない
Failed to set locale, defaulting to C.UTF-8
Last metadata expiration check: 0:04:22 ago on Wed Nov 18 17:23:56 2020.
No match for argument: the_silver_searcher
Error: Unable to find a match: the_silver_searcher
```

しかし上記のようにエラーが出てインストール出来なかったので上手く行く方法を下記していきます。
ついでにMacやUbuntuでも同じことをやっていきます。

# 各種OSにインストール
## CentOS8
- バージョン確認

```
$ cat /etc/redhat-release
CentOS Linux release 8.0.1905 (Core)
```

- インストール

```
-- README.mdのdependenciesに下記二つ記載あり
$ sudo dnf -y groupinstall "Development Tools"
$ sudo dnf -y install pcre-devel xz-devel zlib-devel
$ cd /usr/local/src
$ sudo git clone https://github.com/ggreer/the_silver_searcher.git
$ cd the_silver_searcher/
$ sudo ./build.sh
$ sudo make install
-- agが入っていることを確認
$ which ag
/usr/local/bin/ag
```

## Ubuntu
- バージョン確認

```
$ cat /etc/os-release
-- 一部省略
VERSION="20.04.1 LTS (Focal Fossa)"
```

- インストール

```
$ apt update
$ sudo apt install silversearcher-ag
-- agが入っていることを確認
$ which ag
/usr/bin/ag
```

※Ubuntuは仮想環境ではなくLENOVO ideapad 330Sに直インスコしたのを使っています。


## MacOS
- バージョン確認

```
$ sw_vers
ProductName:    Mac OS X
ProductVersion: 10.15.7
BuildVersion:   19H15
```

- インストール

```
$ sudo brew install the_silver_searcher
-- agが入っていることを確認 
$ which ag
/usr/local/bin/ag
```

※Homebrewが未インストールなら[このリンク先記事](https://qiita.com/digitalhimiko/items/438fb17706873b476040#%E3%81%BE%E3%81%9A%E3%81%AFhomebrew%E3%82%92%E5%B0%8E%E5%85%A5%E3%81%97%E3%82%88%E3%81%86)を参考にインストールしてください。

今後、時間を見つけてagの使い方やgrepとの比較について軽く記事を書く予定です。:rocket:
本日は以上です。

# 参考記事

- [The Siver Searcher README.md/GitHub](https://github.com/ggreer/the_silver_searcher)
- [Installing ag on CentOS/GitHub](https://gist.github.com/rkaneko/988c3964a3177eb69b75)

[^1]: 筆者の感覚値としても圧倒的に早いし[grepより速いackより3〜5倍高速](https://www.softantenna.com/wp/software/the-silver-searcher/)との情報もある。ただこれに[異議を唱える人](https://entree.hatenadiary.org/entry/20150105/1420467355)もおり筆者も数値的な検証はしていないのでここで明言はできません。ごめんね。

