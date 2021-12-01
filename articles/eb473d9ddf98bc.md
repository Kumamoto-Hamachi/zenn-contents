---
title: "[pyenv/poetryでお気楽環境構築してみた]poetryで指定したバージョンになっていない=>キャッシュが残ってただけ"
emoji: "📝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["python", "poetry", "pyenv", "環境構築"]
published: false
---
# 0. はじめに


# 0-1. 前提/環境

- OS

```
$ cat /etc/os-release
NAME="Ubuntu"
VERSION="20.04.3 LTS (Focal Fossa)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 20.04.3 LTS"
VERSION_ID="20.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=focal
UBUNTU_CODENAME=focal
```

OSはUbuntuだがMacOSでもある程度読み替えれば参考になる内容かと思います.

- pyenv, poetry

pyenvについてはOSや使っているシェルスクリプト[GitHub pyenv Installation](https://github.com/pyenv/pyenv#installation)を参考にインストールする.

```
$ pyenv install 3.8.2
$ pyenv install 3.7.10
$ pyenv global 3.8.2
$ pyenv versions
  system
  3.7.10
* 3.8.2 (set by /home/kumamoto/.pyenv/version)
```

\* インストールするバージョンは, 任意のもので良い

poetryについては, [GitHub poetry Installation](https://github.com/python-poetry/poetry)を参考に下記のように準備.

```
$ curl -sSL https://install.python-poetry.org | python3 -
```

# 1. pyenvで特定のディレクトリ(フォルダ)以下でPythonのバージョンを変える

```
$ cd /path/to/your/test_env
$ pyenv local 3.7.10
```

# 2. poetryで使用するPythonのバージョンを指定して仮想環境を作る

```
$ poetry init
Package name [project_xyz]:
Version [0.1.0]:
Description []:
Author [John Doe <johnd@example.com>, n to skip]:
License []:
Compatible Python versions [^3.9]:

-- 入れたいパッケージをインタラクティブに入れるか聞かれる
Would you like to define your main dependencies interactively? (yes/no) [yes]
-- 入れたい開発に用いるパッケージをインタラクティブに入れるか聞かれる
Would you like to define your development dependencies interactively? (yes/no) [yes]
```

`pyproject.toml`が出来ている.先程指定したパッケージが記述されているはず.
後々追加するには`poetry add 入れたいパッケージ名`とする.


```
$ poetry env use 3.7.10
$ poetry env info
Virtualenv
Python:         3.7.10
Implementation: CPython
Path:           /home/kumamoto/.cache/pypoetry/virtualenvs/django-study-for-lap-b9Ea8HW7-py3.7
Valid:          True

System
Platform: linux
OS:       posix
Python:   /home/kumamoto/.pyenv/versions/3.7.10
```

```
$ poetry install
```


# 3. 番外編:バージョンを変えて改めて環境構築するとバージョンが合わない...？を解決
## Pythonのバージョンを変える

`pyproject.toml`を書き換え.

```
[tool.poetry.dependencies]
python = "3.7.10"
```

それをpoetry.lockに反映させます.

```
--lockを付けないとinstallもされます
$ poetry update --lock
```

\* 補足 まずはupdateしよう

`pyproject.toml`の中身を変えていきなりこれを`poetry install`しようとすると,下記のようなエラーになります.

```
Installing dependencies from lock file
Warning: The lock file is not up to date with the latest changes in pyproject.toml. You may be getting outdated dependencies. Run update to update them.
```

pyenvのバージョンを変えて`poetry install`します.

```
$ pyenv local 3.8.2
$ poetry install
```

仕上げに`pyenv`の仮想環境で作ったPythonのパスを`poetry`に教えてやります.

```
$ which python
/home/kumamoto/.pyenv/shims/python
$ poetry env use /home/kumamoto/.pyenv/shims/python
```

## 切り替えてもバージョンが切り替わらない...？

ちゃんとpoetryの仮想環境に入る前はPythonのバージョンは3.8.2です.

```
$ pyenv versions
system
3.8.2(set by /home/kumamoto/Documents/test_env/.python-version)

$ python --version
3.8.2
```

しかし, poetryの仮想環境下だと3.7.10のままです.

```
$ poetry shell
$ python --version
Python 3.7.10
```

実際, poetryの仮想環境の情報を下記で確認するとPathの名前は`3.8`風になっていますが実際の
バージョンは`3.7.10`になっています.(かなしみ)

```
$ poetry env info
Virtualenv
Python:         3.7.10
Implementation: CPython
Path:           /home/kumamoto/.cache/pypoetry/virtualenvs/django-study-for-lap-b9Ea8HW7-py3.8
Valid:          True

System
Platform: linux
OS:       posix
Python:   /home/kumamoto/.pyenv/versions/3.7.10
```

\* ちなみにPathのみ確認したい場合は, `poetry env info --path`

Path部分をにcacheと入っていることからどうやらcacheが残っていておせっかいをしているだろう
ということがわかりました.

## キャッシュをぶっ壊す〜

```
$ cd /home/kumamoto/.cache/pypoetry/virtualenvs
$ rm -rf test_env-py3.8
```

ちなみに`poetry cache clear`というコマンドもありますがこれはpypiのキャッシュを消す
用のコマンドらしいので試してみましたが意味はなさそうでした.
