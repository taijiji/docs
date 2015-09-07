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

この中のvagrantfileを下記のように編集します

```rb
Vagrant.configure(2) do |config|
   config.vm.box = "centos70"
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
[taiji@aooni] ~/work/vagrant/centos7_ipdesigner
% cat date.txt
Mon Sep  7 23:14:11 UTC 2015
```

仮想マシンが正常に作成できたことが確認できました。

ひとまずCentOSにインストールされているモジュールを最新にします。

```
yum install
```


次章では、仮想マシンにDjangoとPythonをインストールしていきます。

# Python3系をインストール
CentOS7では、デフォルトでPython2.7.5がインストールされています。

```
[vagrant@localhost vagrant]$ python --version
Python 2.7.5
```

ここではPython3系の最新版であるPython3.4.3をインストールします。
