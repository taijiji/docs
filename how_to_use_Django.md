#概要
PythonのWebフレームワークであるDjangoを使ってWebアプリを作ってみます。

PythonのWebフレームワークは幾つかあります。
- Flask : 比較的軽量なフレームワークです。
- Djanog : もりもりのフレームワークです。

業務で使う場合はDjagoであれば大体のものを書けるので、Djangoに慣れておくとよいでしょう。

#Vagrantで仮想マシンを立ち上げる

まずは適当なディレクトリを作ります。

```
% mkdir django_app
```

次に、下記のコマンドでvagrantの設定ファイルを作成します。

```
% cd django_app
% vagrant init
```

そうすると下記のようなファイルが作成されます。

```
% ls -al
total 8
drwxr-xr-x   4 taiji  staff   136  9  8 07:53 ./
drwxr-xr-x  16 taiji  staff   544  9  8 07:49 ../
drwxr-xr-x   3 taiji  staff   102  9  8 07:53 .vagrant/
-rw-r--r--   1 taiji  staff  3081  9  8 07:53 Vagrantfile
```

vagrantfileを下記のように編集します。
ここではDjangoで作成したwebアプリにアクセスするためのネットワーク設定も追加しています。


```rb
Vagrant.configure(2) do |config|
   config.vm.box = "centos70"
   config.vm.network "forwarded_port", guest: 8000, host: 80
end
```

次にvagrantで記述した仮想マシンを起動します。(起動まで数分かかります)

```
% vagrant up
```

仮想マシンが正常に立ち上がれば、下記コマンドで確認することができます。

```
% vagrant status
Current machine states:

default                   running (virtualbox)

The VM is running. To stop this VM, you can run `vagrant halt` to
shut it down forcefully, or you can run `vagrant suspend` to simply
suspend the virtual machine. In either case, to restart it again,
simply run `vagrant up`.
```

正しく作成されたか、ログインして確認します

```
% vagrant ssh
Welcome to your Vagrant-built virtual machine.
[vagrant@localhost ~]$
```

仮想マシン、ホストマシンのフォルダ共有が有効になっているか確認します。デフォルトでは、仮想マシンの/vagrantディレクトリと、ホストマシンのvagrantfileが設置されたディレクトリが共有されています。
ここでは共有機能をテストするために、簡単なファイルを設置してみます。

```:仮想マシン
[vagrant@localhost ~]$ cd /vagrant/
[vagrant@localhost vagrant]$ date > date.txt
[vagrant@localhost vagrant]$ cat date.txt
Mon Sep  7 23:14:11 UTC 2015

[vagrant@localhost vagrant]$ ls -al
total 12
drwxr-xr-x   1 vagrant vagrant  170 Sep  7 23:14 .
dr-xr-xr-x. 18 root    root    4096 Sep  7 22:57 ..
drwxr-xr-x   1 vagrant vagrant  102 Sep  7 22:53 .vagrant
-rw-r--r--   1 vagrant vagrant 3081 Sep  7 22:53 Vagrantfile
-rw-r--r--   1 vagrant vagrant   29 Sep  7 23:14 date.txt
```

```:ホストマシン
% ls -al
total 16
drwxr-xr-x   5 taiji  staff   170  9  8 08:14 ./
drwxr-xr-x  16 taiji  staff   544  9  8 07:49 ../
drwxr-xr-x   3 taiji  staff   102  9  8 07:53 .vagrant/
-rw-r--r--   1 taiji  staff  3081  9  8 07:53 Vagrantfile
-rw-r--r--   1 taiji  staff    29  9  8 08:14 date.txt
[taiji@aooni] ~/work/vagrant/centos7_app1
% cat date.txt
Mon Sep  7 23:14:11 UTC 2015
```

仮想マシンが正常に作成できたことが確認できました。

CentOSにインストールされている全パッケージを最新にしておきます。

```
sudo yum update -y
```

これで仮想マシンにおける準備はOKです。

次章では、仮想マシンにDjangoとPythonをインストールしていきます。

# Python3系をインストール
CentOS7では、デフォルトでPython2.7.5がインストールされています。

```
[vagrant@localhost vagrant]$ python --version
Python 2.7.5
```

ここではPython3系の最新版であるPython3.4.3をダウンロードします。
```
[vagrant@localhost ~]$ cd /usr/local/src
[vagrant@localhost src]$ sudo wget https://www.python.org/ftp/python/3.4.3/Python-3.4.3.tgz
```

次にPython3.4.3をインストールしていきます。
```
[vagrant@localhost src]$ sudo tar xzvf Python-3.4.3.tgz
[vagrant@localhost src]$ ls
Python-3.4.3  Python-3.4.3.tgz
[vagrant@localhost src]$ cd Python-3.4.3
[vagrant@localhost Python-3.4.3]$ sudo ./configure
[vagrant@localhost Python-3.4.3]$ sudo make
[vagrant@localhost Python-3.4.3]$ sudo make altinstall
```

/usr/local/bin/配下にpython3.4がインストールされたことが確認できます。

```
[vagrant@localhost Python-3.4.3]$ ls -al /usr/local/bin
total 22356
drwxr-xr-x.  2 root root     4096 Sep  8 00:15 .
drwxr-xr-x. 12 root root     4096 Jul 14 05:11 ..
-rwxr-xr-x   1 root root      101 Sep  8 00:15 2to3-3.4
-rwxr-xr-x   1 root root      241 Sep  8 00:15 easy_install-3.4
-rwxr-xr-x   1 root root       99 Sep  8 00:15 idle3.4
-rwxr-xr-x   1 root root      213 Sep  8 00:15 pip3.4
-rwxr-xr-x   1 root root       84 Sep  8 00:15 pydoc3.4
-rwxr-xr-x   2 root root 11423753 Sep  8 00:14 python3.4
-rwxr-xr-x   2 root root 11423753 Sep  8 00:14 python3.4m
-rwxr-xr-x   1 root root     3032 Sep  8 00:15 python3.4m-config
-rwxr-xr-x   1 root root      236 Sep  8 00:15 pyvenv-3.4

[vagrant@localhost ~]$ /usr/local/bin/python3.4 --version
Python 3.4.3
```

Python 3.4.3コマンドのPATHを通します。

```
[vagrant@localhost Python-3.4.3]$ sudo ln -s /usr/local/bin/python3.4 /usr/bin/python3
```

こうすることで、python3 コマンドでpython3.4.3を呼び出すことができます。

```
[vagrant@localhost ~]$ python3 -V
Python 3.4.3

[vagrant@localhost ~]$ sudo python3 -V
Python 3.4.3
```

次に、pipをインストールしていきます。
pipはPythonのサードパーティ製パッケージをインストールするコマンドです。
pipコマンドは、Python3.3以前ではインストールする必要がありましたが、
Python3.4以降ではデフォルトで提供されています。

```
[vagrant@localhost ~]$ python3.4 -m pip list
You are using pip version 6.0.8, however version 7.1.2 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
pip (6.0.8)
setuptools (12.0.5)
```

pipのversionが古いようなので下記コマンドでpipをupgradeします。
```
[vagrant@localhost ~]$ sudo python3 -m pip install --upgrade pip

[vagrawnt@localhost ~]$ sudo python3 -m pip --version
pip 7.1.2 from /usr/local/lib/python3.4/site-packages (python 3.4)
```

次にvenvを構築します。
venvはプロジェクト単位でインストールするパッケージやpythonのバージョンを切り替えることができる仮想環境です。
Python2系ではvirtualenvなどをインストールする必要がありましたが、
Python3系ではpyvenvという名前でデフォルト機能として提供されています。

まず/vagrantディレクトリにアリケーション用のディレクトリを作ります。
```
[vagrant@localhost ~]$ cd /vagrant/
[vagrant@localhost vagrant]$ mkdir app1
```

次に作成したディレクトリに、pyvenvで仮想環境を構築します。
このときPython3.4.3をデフォルト設定するようにします。

```
[vagrant@localhost vagrant]$ cd app1
[vagrant@localhost app1]$ pyvenv-3.4 env_app1
``

作成した仮想環境にログインします。

```
[vagrant@localhost app1]$ source env_app1/bin/activate
(env_app1) [vagrant@localhost app1]$
```

アプリケーション専用の仮想環境を構築することができました。
仮想環境の状態を確認してみます。

```
(env_app1)[vagrant@localhost app1]$ python --version
Python 3.4.3
(env_app1)[vagrant@localhost app1]$ pip --version
pip 6.0.8 from /vagrant/app1/env_app1/lib/python3.4/site-packages (python 3.4)

(env_app1) [vagrant@localhost app1]$ pip list
You are using pip version 6.0.8, however version 7.1.2 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
pip (6.0.8)
setuptools (12.0.5)
```

pyvenv環境のpipもバージョンが古いので、upgradeします。
```
(env_app1) [vagrant@localhost app1]$ pip install --upgrade pip

(env_app1) [vagrant@localhost app1]$ pip list
pip (7.1.2)
setuptools (12.0.5)
```

なお仮想環境から離脱する場合は、下記のようにします。

```
(env_app1)[vagrant@localhost app1]$ deactivate
[vagrant@localhost app1]$
```

環境構築はこれで完成です。
次の章では、Djangoのアプケーションを開発していきます。

#Djangoをインストール
Djangoをインストールします。

```
(env_app1)[vagrant@localhost app1]$ pip install django

(env_app1)[vagrant@localhost app1]$ pip freeze
Django==1.8.4
wheel==0.24.0
```

#Djangoプロジェクトを生成

```
(env_app1)[vagrant@localhost app1]$ django-admin startproject app1

(env_app1)[vagrant@localhost app1]$ ls -al
total 0
drwxr-xr-x 1 vagrant vagrant 136 Sep  8 03:06 .
drwxr-xr-x 1 vagrant vagrant 204 Sep  8 02:53 ..
drwxr-xr-x 1 vagrant vagrant 204 Sep  8 02:57 env_app1
drwxr-xr-x 1 vagrant vagrant 136 Sep  8 03:06 app1
```

プロジェクトを生成すると下記のようなディレクトリが作成されます。

```
app1/
|-- app1
|   |-- __init__.py
|   |-- settings.py
|   |-- urls.py
|   `-- wsgi.py
`-- manage.py
```

Djangoアプリを作成します。

```
(env_app1)[vagrant@localhost app1]$  python manage.py startapp as2518db
```

as2518ディレクトリが追加される

```
(env_app1)[vagrant@localhost app1]$ tree app1/
app1/
|-- as2518db
|   |-- __init__.py
|   |-- admin.py
|   |-- migrations
|   |   `-- __init__.py
|   |-- models.py
|   |-- tests.py
|   `-- views.py
|-- app1
|   |-- __init__.py
|   |-- __pycache__
|   |   |-- __init__.cpython-34.pyc
|   |   `-- settings.cpython-34.pyc
|   |-- settings.py
|   |-- urls.py
|   `-- wsgi.py
`-- manage.py
```

次にDBを構築します。
ここではmariadbを使います

```
(env_app1)[root@localhost app1]# yum install mariadb-server
(env_app1)[vagrant@localhost app1]$ sudo yum install mariadb-devel
(env_app1)[root@localhost app1]# python3 -m pip install PyMySQL
```

dbにrootユーザを作成します

```

```

```
(env_app1)[vagrant@localhost app1]$ python manage.py migrate
```



現在の状態でのDjango webアプリを立ち上げてみます。
```
(env_app1)[vagrant@localhost app1]$ python manage.py runserver 0.0.0.0:8000

```
