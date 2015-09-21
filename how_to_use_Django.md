# Title: Python Djangoで作るWebアプリケーション

#概要
PythonのWebフレームワークであるDjangoを使って、Webアプリケーションを作っていきます。
私自身がWebアプリケーション開発の勉強中なので、
備忘録と手順をまとめる意図でゼロベースでインフラ環境を構築するところから書いています。

#Python Webフレームワーク
日本でWebアプリケーション開発というと、RubyとそのWebフレームワークであるRuby on Railsが圧倒的に有名ですが、
PythonでもWebフレームワークがいくつか用意されており、その中でも有名で多く使われている2つを紹介します。

## [Django](https://www.djangoproject.com/)
Python Webフレームワークとしては最も利用されています。テンプレートエンジンやORM(データベースとプログラム上のオブジェクトをマッピングしてくれる機能)、テスト機能や日本語化機能などを包括しているオールインワン型のWebフレームワークです。またWebアプリケーションの管理者用GUIを自動生成してくれる便利な機能もあります。操作する対象ファイルが多いので学習コストはありますが、実サービスでも十分運用していくことができるので長く使い続けることができます。

## [Flask](http://flask.pocoo.org/)
Flaskは軽量なWebフレームワークで、Djangoの次に人気があるようです。オールインワン型のDjanogに比べると、 かなり少ないファイル構造で構成されており、学習コストも低く初心者向きです。
データベースを使わないような小さいWebアプリケーションの開発に向いています。
規模の大きいWebアプリケーションを作成する場合には、Flask以外のPythonパッケージを組み合わせながら開発することになります。

----------------------------

これらのほかにBottle, Pyramid, Tornade, PloneといったWebフレームワークがありますが、現時点では上記２つが非常に多く使われているようです。

今回はDjnagoを用いてWebアプリケーションを開発していきます。
業務で使うWebアプリケーションを開発する場合は、Djangoであれば広範囲の機能をカバーすることができます。
初めは苦戦はするかもしれませんが、早いうちにDjangoに慣れておくことをお勧めします。
Djangoを使うことで、モダンなWeb開発プロセスも学ぶことができます。


#環境構築
まずは環境構築を進めていきます。以下の環境で開発を進めていきます。

- ホストマシン
    - MacBookAir OSX Yosemite 10.10.5
- 仮想マシン管理ソフトウェア
    - Virtualbox 4.3.18
    - Vagrant 1.7.4
- 仮想マシン
    - CentOS 7.1.1503
- Python version
    - Python 3.4.3
- Web フレームワーク
    - Django 1.8.4
- データベース
    - MariaDB 10.1.7
- Webサーバ
    - nginx 1.9.4

## Vagrantで仮想マシンを構築
開発用途で仮想マシンを作ったり、壊したりすることが多い場合は、Vagrantを利用すると便利です。
またVagrantのフォルダ共有機能を使うことで、仮想マシン上にあるファイルをホストマシンの使い慣れたテキストエディタで編集することができるので、
アプリケーションを楽に開発することができます。

まずは適当なディレクトリを作ります。

```
% mkdir django_apps
% cd django_apps
```

次に、下記のコマンドでvagrantの設定ファイルを作成します。

```
% vagrant init
```

そうすると下記のようなファイルが作成されます。

```
% ls -al
total 8
drwxr-xr-x   3 taiji  staff   102  9 18 01:04 ./
drwxr-xr-x  18 taiji  staff   612  9 18 01:03 ../
-rw-r--r--   1 taiji  staff  3016  9 18 01:04 Vagrantfile
```

次にVagrant boxで公開されているCentOS7のイメージをダウンロードおよびインストールをします。  
まずhttp://www.vagrantbox.es/ にて公開されているCentOS7のboxファイルのダウンロードを実施します。
（完了までに数分かかります。）

```
% vagrant box add centos70 https://github.com/holms/vagrant-centos7-box/releases/download/7.1.1503.001/CentOS-7.1.1503-x86_64-netboot.box

==> box: Box file was not detected as metadata. Adding it directly...
==> box: Adding box 'centos70' (v0) for provider:
    box: Downloading: https://github.com/holms/vagrant-centos7-box/releases/download/7.1.1503.001/CentOS-7.1.1503-x86_64-netboot.box
==> box: Successfully added box 'centos70' (v0) for 'virtualbox'!
```

vagrantfileを編集して、下記のような内容に変更します。
ここでDjangoで作成したWebアプリケーションにhttpアクセスするためのネットワーク設定も追加しています。

```rb
Vagrant.configure(2) do |config|
   config.vm.box = "centos70"
   config.vm.network "private_network", ip: "192.168.33.15"
end
```

Vagrantfileで記述した設定で仮想マシンを起動します。(起動まで数分かかります)

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

仮想マシン、ホストマシンのフォルダ共有が有効になっているか確認します。
デフォルトでは、仮想マシンの/vagrantディレクトリと、ホストマシンのVagrantfileが設置されたディレクトリが共有されています。
ここでは共有機能をテストするために、簡単なファイルを設置してみます。

```:仮想マシン
[vagrant@localhost ~]$ cd /vagrant/
[vagrant@localhost vagrant]$ date >date.txt
[vagrant@localhost vagrant]$ cat date.txt
Fri Sep 18 04:57:51 UTC 2015

[vagrant@localhost vagrant]$ ls -la
total 12
drwxr-xr-x   1 vagrant vagrant  170 Sep 18 04:57 .
dr-xr-xr-x. 18 root    root    4096 Sep 18 04:48 ..
drwxr-xr-x   1 vagrant vagrant  102 Sep 17 16:11 .vagrant
-rw-r--r--   1 vagrant vagrant 3226 Sep 18 04:44 Vagrantfile
-rw-r--r--   1 vagrant vagrant   29 Sep 18 04:57 date.txt
```

```:ホストマシン
% ls -la
total 16
drwxr-xr-x   5 taiji  staff   170  9 18 13:57 ./
drwxr-xr-x  18 taiji  staff   612  9 18 01:03 ../
drwxr-xr-x   3 taiji  staff   102  9 18 01:11 .vagrant/
-rw-r--r--   1 taiji  staff  3226  9 18 13:44 Vagrantfile
-rw-r--r--   1 taiji  staff    29  9 18 13:57 date.txt

% cat date.txt
Fri Sep 18 04:57:51 UTC 2015
```

仮想マシンが正常に作成できたことが確認できました。

CentOSにインストールされている全パッケージを最新にしておきます。

```
[vagrant@localhost vagrant]$ sudo yum update -y
```

これで仮想マシンにおける準備はOKです。

次章では、仮想マシンにPythonとDjanogをインストールしていきます。

## Python3系をインストール
CentOS7では、デフォルトでPython2.7.5がインストールされています。

```
[vagrant@localhost vagrant]$ python --version
Python 2.7.5
```

ここではPython3系の最新版であるPython3.4.3をインストールします。

```
[vagrant@localhost ~]$ cd /usr/local/src
[vagrant@localhost src]$ sudo wget https://www.python.org/ftp/python/3.4.3/Python-3.4.3.tgz
[vagrant@localhost src]$ sudo tar xzvf Python-3.4.3.tgz
[vagrant@localhost src]$ ls -al
total 19108
drwxr-xr-x.  3 root   root         48 Sep 18 05:05 .
drwxr-xr-x. 12 root   root       4096 Jul 14 05:11 ..
drwxrwxr-x  15 veewee veewee     4096 Feb 25  2015 Python-3.4.3
-rw-r--r--   1 root   root   19554643 Feb 25  2015 Python-3.4.3.tgz
[vagrant@localhost src]$ cd Python-3.4.3
[vagrant@localhost Python-3.4.3]$ sudo ./configure
[vagrant@localhost Python-3.4.3]$ sudo make
[vagrant@localhost Python-3.4.3]$ sudo make altinstall
```

/usr/local/bin/配下にpython3.4がインストールされたことが確認できます。

```
[vagrant@localhost Python-3.4.3]$ ls -al /usr/local/bin
total 22356
drwxr-xr-x.  2 root root     4096 Sep 18 05:09 .
drwxr-xr-x. 12 root root     4096 Jul 14 05:11 ..
-rwxr-xr-x   1 root root      101 Sep 18 05:09 2to3-3.4
-rwxr-xr-x   1 root root      241 Sep 18 05:09 easy_install-3.4
-rwxr-xr-x   1 root root       99 Sep 18 05:09 idle3.4
-rwxr-xr-x   1 root root      213 Sep 18 05:09 pip3.4
-rwxr-xr-x   1 root root       84 Sep 18 05:09 pydoc3.4
-rwxr-xr-x   2 root root 11423753 Sep 18 05:09 python3.4
-rwxr-xr-x   2 root root 11423753 Sep 18 05:09 python3.4m
-rwxr-xr-x   1 root root     3032 Sep 18 05:09 python3.4m-config
-rwxr-xr-x   1 root root      236 Sep 18 05:09 pyvenv-3.4
```

インストールされたPythonのバージョンを確認します。

```
[vagrant@localhost ~]$ /usr/local/bin/python3.4 --version
Python 3.4.3
```

Python 3.4.3コマンドのPATHを通します。

```
[vagrant@localhost Python-3.4.3]$ sudo ln -s /usr/local/bin/python3.4 /usr/bin/python3
```

こうすることで「python3」コマンドでPython3.4.3を呼び出すことができるので、用途に応じてPython2系とPython3系を使い分けることができます。

```
[vagrant@localhost Python-3.4.3]$ python3 --version
Python 3.4.3
[vagrant@localhost Python-3.4.3]$ python --version
Python 2.7.5
```


## pipをインストール

Pythonのパッケージマネージャであるpipを利用する環境を構築します。
Python2系では2.7.9以降、Python3系では3.4以降からpipがデフォルト機能としてインストールされています。

```
[vagrant@localhost ~]$ python3 -m pip list
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

pipを使うたびに「 python3 -m pip」というコマンドを打つのは面倒なので、
ここではシンボリックリンクで「pip3」コマンドでPython3系のpipを呼び出すようにします。

```
[vagrant@localhost Python-3.4.3]$ sudo ln -s /usr/local/bin/pip3.4  /usr/bin/pip3

[vagrant@localhost Python-3.4.3]$ pip3 --version
pip 7.1.2 from /usr/local/lib/python3.4/site-packages (python 3.4)
```

###参考 : Python2.7.8以前、Python3.3系以前でpipをインストール
Python2.7.8以前、Python3.3系以前では、pipは別途インストールする必要があります。
その場合は下記の手順を試してください。
今回はPython3.4系を利用しますので、実行しなくてOKです。

```
$ sudo wget https://bootstrap.pypa.io/get-pip.py

#rootユーザにインストール
$ sudo python get-pip.py

#非rootユーザにインストール
$ python get-pip.py –user
```

## pyvenvで仮想実行環境を構築
次にpyvenv環境を構築します。
pyenvは利用パッケージやpythonのバージョンを、プロジェクト単位で切り替えることができる仮想実行環境です。
開発段階においては複数の機能実装を並行して開発する場合が多く、
仮想実行環境を利用することでより、アプリケーションごとに開発環境を切り替えてテストしたり、
アプケーションを動作させるためのパッケージを明文化することができます。
Python2系ではvirtualenvなどをインストールする必要がありましたが、
Python3系ではpyvenvというデフォルト機能として提供されています。

まず/vagrantディレクトリにアリケーション用のディレクトリを作ります。

```
[vagrant@localhost ~]$ cd /vagrant/
[vagrant@localhost vagrant]$ mkdir django_apps
[vagrant@localhost vagrant]$ cd django_apps/
```

次に作成したディレクトリに、pyvenvで仮想環境を構築します。
このときPython3.4.3をデフォルト設定するようにします。

```
[vagrant@localhost app1]$ pyvenv-3.4 venv_app1

[vagrant@localhost django_apps]$ ls -la
total 0
drwxr-xr-x 1 vagrant vagrant 102 Sep 18 06:05 .
drwxr-xr-x 1 vagrant vagrant 204 Sep 18 06:02 ..
drwxr-xr-x 1 vagrant vagrant 238 Sep 18 06:04 venv_app1
```

作成した仮想実行環境に読み込みます。

```

[vagrant@localhost django_apps]$ source venv_app1/bin/activate
(venv_app1) [vagrant@localhost django_apps]$
```

専用の仮想実行環境を構築することができました。
仮想環境の状態を確認してみます。

```
(venv_app1) [vagrant@localhost django_apps]$ python --version
Python 3.4.3
(venv_app1) [vagrant@localhost django_apps]$ which python
/vagrant/django_apps/venv_app1/bin/python
```

このように、作成されたvenv_app1ディレクトリ配下に新たにPython実行環境が作成されていることがわかります。
pipについても同様に、venv_app1ディレクトリ配下に作成されていることが確認できます。

```
(venv_app1) [vagrant@localhost django_apps]$ pip --version
pip 6.0.8 from /vagrant/django_apps/venv_app1/lib/python3.4/site-packages (python 3.4)
```

pyvenv環境のpipもバージョンが古いので、upgradeしておきます。

```
(venv_app1) [vagrant@localhost django_apps]$ pip install --upgrade pip
```

なお仮想実行環境から離脱する場合は、下記のようにします。

```
(env_app1)[vagrant@localhost app1]$ deactivate
[vagrant@localhost app1]$
```

再度、仮想実行環境にログインする場合は、作成時と同じコマンドを入力します。

```
[vagrant@localhost django_apps]$ source venv_app1/bin/activate
(venv_app1) [vagrant@localhost django_apps]$
```

以降の章では、作成した仮想実行環境「venv_app1」を使って進めていきます。

## MriaDBをインストール
MariaDBはMySQLをフォークして立ち上げられたプロジェクトであり、MySQLと機能互換があります。
MySQLと同様のデータベース操作コマンドやPythonパッケージを使うことができます。

MariaDBをインストールしていきます。

```
(venv_app1) [vagrant@localhost django_apps]$ sudo yum install -y mariadb-server mariadb-devel
```

MariaDBをPythonプログラムから操作するために、mysqlclientパッケージをインストールします。

```
(venv_app1) [vagrant@localhost django_apps]$ pip install  mysqlclient
```

MariaDBを起動します

```
(venv_app1) [vagrant@localhost django_apps]$ sudo systemctl start mariadb
(venv_app1) [vagrant@localhost django_apps]$ sudo systemctl enable mariadb
```

MariaDBにrootユーザでログインします。

```
(venv_app1) [vagrant@localhost django_apps]$  mysql -u root

Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 2
Server version: 5.5.44-MariaDB MariaDB Server

Copyright (c) 2000, 2015, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```

今回作成するWebアプリケーション用に、新規にデータベースを作成します。
ここでは、「app1_db」という名前のデータベースを作成します。

```
MariaDB [(none)]> CREATE DATABASE app1_db CHARACTER SET utf8;

Query OK, 1 row affected (0.00 sec)
```

作成したデータベースにアクセスできるユーザを作成します。
ここでは「app1_user」という名前のユーザを作成します。
パスワードは「app1_passwd」を設定しています。
```
MariaDB [(none)]> GRANT ALL PRIVILEGES ON app1_db.* TO app1_user@localhost IDENTIFIED BY 'app1_passwd';

Query OK, 0 rows affected (0.00 sec)
```

一旦MySQLプロンプトを終了します。

```
MariaDB [(none)]> exit
Bye
```

正常にデータベースとユーザが作成されたか確認するために、新しいユーザでログインします。

```
# -pの後にスペースを入れないように注意
(venv_app1) [vagrant@localhost django_apps]$  mysql -u app1_user -papp1_passwd

MariaDB [(none)]> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| app1_db            |
| test               |
+--------------------+
3 rows in set (0.00 sec)
```
app1_dbが正常に作成されたことを確認することができました。

これでMariaDBの初期設定は完了です。
Djangoを使っていると、直接、MariaDBコマンドを叩く機会は多くはありません。
Webアプリケーションを開発するときは、DjangoのORM(Object-Relational Mapping)機能を利用することで
Models.pyという設定ファイルに書かれた内容を元に、動的にデータベースが反映されていきます。

## nginxをインストール

Webサーバとしてnginxをインストールしていきます。

仮想マシンに、nginx yum レポジトリを追記します。

```
(venv_app1) [vagrant@localhost django_apps]$ cd /etc/yum.repos.d/

(venv_app1) [vagrant@localhost django_apps]$ sudo vi nginx.repo

# 下記を記載
## ここでは最新versionをインストールするように記載しています。
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=0
enabled=1
```

nginx.repoが作成されたことを確認します。

```
(venv_app1) [vagrant@localhost yum.repos.d]$ ls -al
total 44
drwxr-xr-x.  2 root root 4096 Sep 18 09:58 .
drwxr-xr-x. 79 root root 8192 Sep 18 06:33 ..
-rw-r--r--.  1 root root 1664 Mar 31 22:27 CentOS-Base.repo
-rw-r--r--.  1 root root 1309 Mar 31 22:27 CentOS-CR.repo
-rw-r--r--.  1 root root  649 Mar 31 22:27 CentOS-Debuginfo.repo
-rw-r--r--.  1 root root 1331 Mar 31 22:27 CentOS-Sources.repo
-rw-r--r--.  1 root root 1002 Mar 31 22:27 CentOS-Vault.repo
-rw-r--r--.  1 root root  290 Mar 31 22:27 CentOS-fasttrack.repo
-rw-r--r--   1 root root  109 Sep 18 09:58 nginx.repo
```

次に、yumコマンドでnginxをインストールします。

```
(venv_app1) [vagrant@localhost yum.repos.d]$ sudo yum install -y nginx
```

nginxコマンドを使って、正しくインストールできたことを確認します。

```
(venv_app1) [vagrant@localhost yum.repos.d]$ nginx -v
nginx version: nginx/1.9.4
```

nginxを起動させます。

```
(venv_app1) [vagrant@localhost yum.repos.d]$ sudo systemctl start nginx.service
(venv_app1) [vagrant@localhost yum.repos.d]$ sudo systemctl enable nginx.service
```

nginxは下記 2ファイルで設定されています。
デフォルトでは「/usr/share/nginx/html」ディレクトリ配下の「index.html」をWebアクセスできるように設定されています。


```
(venv_app1) [vagrant@localhost yum.repos.d]$ less /etc/nginx/nginx.conf

user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```

```
(venv_app1) [vagrant@localhost yum.repos.d]$ less /etc/nginx/conf.d/default.conf
server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/log/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```

デフォルトで用意されているindex.htmlにHTTPアクセスできるか確認してみます。

まず確認する前に、firewalldを停止状態にします。
(フィルタ機能が全停止するので、本番環境での実施には十分に注意してください。)

```
(venv_app1) [vagrant@localhost yum.repos.d]$ sudo systemctl stop firewalld
(venv_app1) [vagrant@localhost yum.repos.d]$ sudo systemctl disable firawalld
```

ここではVagrantfileのprivate_networkの業で指定したのIPアドレスを利用して、ホストマシンのWebブラウザからアクセスすることができます。

```
http://192.168.33.15/
```

成功していればWebブラウザで以下のような画面が表示されます。
[nginx snapshot](./nginx_snapshot.png)

以上でnginxの構築は完了です。


## Djangoをインストール
Djangoのパッケージをpipを使ってインストールします。
pipを使うことで、以下のように簡単にインストールをすることができます。

```
(venv_app1) [vagrant@localhost django_apps]$ pip install django

(venv_app1) [vagrant@localhost django_apps]$ pip list
Django (1.8.4)
pip (7.1.2)
setuptools (12.0.5)
```

## uWSGIをインストール
Djangoで開発したWebアプリケーションとWebサーバを連動させるインタフェース(WSGI: Web Server Gateway Interfaceと呼ばれます)として、
uWSGIを利用します。

pipを使ってuWSGIパッケージをインストールします。

```
(venv_app1) [vagrant@localhost django_apps]$ pip install uwsgi

(venv_app1) [vagrant@localhost django_apps]$ pip list
Django (1.8.4)
mysqlclient (1.3.6)
pip (7.1.2)
setuptools (12.0.5)
uWSGI (2.0.11.1)
```

実際にDjangoアプリケーションとWebサーバを連動させるにはもう少し設定が必要ですが、
そちらはDjangoでWebアプリケーションを作るときに、手順を説明していきます。

--------------------------------

環境構築はこれで完了です。
次章から、いよいよDjangoを使ってアプケーションを開発していきます。

# DjangoでWebアプリケーションを作る
Djangoはサービス全体を示す「プロジェクト」という単位と、
サービスの１機能を示す「アプリケーション」という単位が存在します。
プロジェクトとアプリケーションの範囲は厳密に定義されているわけではありませんが、
モデル自体はアプリケーションごとに定義しますが、
実際のデータベースはプロジェクト全体で共有することになります。

ここではプロジェクト名を「pj1」、
アプリケーション名を「app1」という名前でを作っていきます。
Webアプケーションの例として、IPアドレスの利用状況を管理するアプリケーションを作っていきます。
なお、ここでは/vagrantディレクトリ配下にDjangoプロジェクトを作成していくことで、ホストマシン側のテキストエディタでアプリケーションファイルを編集しています。

## Djangoプロジェクトを生成
まず初めにDjangoプロジェクトを作成していきます。

```
(venv_app1) [vagrant@localhost ~]$ cd /vagrant/django_apps/
(venv_app1) [vagrant@localhost django_apps]$

(venv_app1) [vagrant@localhost django_apps]$ django-admin startproject pj1

(venv_app1) [vagrant@localhost django_apps]$ ls -al
total 0
drwxr-xr-x 1 vagrant vagrant 136 Sep 18 14:36 .
drwxr-xr-x 1 vagrant vagrant 204 Sep 18 10:47 ..
drwxr-xr-x 1 vagrant vagrant 136 Sep 18 14:36 pj1
drwxr-xr-x 1 vagrant vagrant 272 Sep 18 06:12 venv_app1
```

プロジェクトを生成すると下記のようなディレクトリが作成されます。

```
pj1/
|-- manage.py
`-- pj1
    |-- __init__.py
    |-- settings.py
    |-- urls.py
    `-- wsgi.py
```

pj1/pj1ディレクトリ配下のファイル群が、全アプリケーションに共通する設定ファイルです。

またpj1ディレクトリ配下に自動で作成されるmanage.pyが、
Djangoを使ってWebアプリケーションを開発する上で様々な便利な機能を備えるプログラムです。

例えば、Djangoの開発用簡易Webサーバを立ち上げる機能などがあります。
これによりngixなどのWebサーバを用意せずとも、開発中のアプリケーションをWebブラウザで確認することが可能です。
以下のコマンドで、簡易Webサーバを起動します。

```
(venv_app1) [vagrant@localhost django_apps]$ python pj1/manage.py runserver 0.0.0.0:8000
```

起動後、ホストマシンのWebブラウザで下記URLを入力してみます。

```
http://192.168.33.15:8000/
```

成功すると、Webブラウザで以下の画面を確認することができます。

[django_snapshot](./django_snapshot.png)


## Djanogプロジェクトの初期設定
アプリケーション全体で共通するDjangoプロジェクトの初期設定を進めていきます。

pj1/pj1/settings.pyを編集していきます。

```
(venv_app1) [vagrant@localhost]$ cd /vagrant/django_apps/
(venv_app1) [vagrant@localhost django_apps]$ vi pj1/pj1/settings.py
```

```
# 追加・削除した部分のみを記載しています。

# データベースはデフォルトでSQLiteが設定されているため、MySQLに変更する
 DATABASES = {
    'default': {

        # 以下を削除
        #'ENGINE': 'django.db.backends.sqlieete3',
        #'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),

        # 作成したデータベース情報を追記
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'app1_db',
        'USER': 'app1_user',
        'PASSWORD': 'app1_passwd',
        'HOST': 'localhost',
        'PORT': '3306',
    }
 }

(中略)

# 言語設定を英語から日本語に変更

# 以下を削除
# LANGUAGE_CODE = 'en-us'
# TIME_ZONE = 'UTC'

#以下を追記
LANGUAGE_CODE = 'ja'
TIME_ZONE = 'Asia/Tokyo'

```

pj1/pj1/settings.pyを変更すると言語設定が日本語に変更されています。
再び簡易Webサーバを起動して、ホストマシンWebブラウザでみてみましょう。
Djangoで作成されたデフォルトのWebページが日本語表示になったことが確認できます。

```
(venv_app1) [vagrant@localhost django_apps]$ python pj1/manage.py runserver 0.0.0.0:8000
```

[django_ja_snapshot](./django_ja_snapshot.png)

以上でDjangoプロジェクトの初期設定は完了です。

# データベースを初期化
データベースを初期化するため以下のコマンドを実行します。
実行するとSQL文が発行され、データベース「app1_db」に管理用テーブルが生成されます。
このときに対話形式で、管理者情報を入力する必要があります。
ここでは適当にID:admin, Pass:admin, e-mail:なしで作成しました。

```
(venv_app1) [vagrant@localhost django_apps]$ python pj1/manage.py syncdb

Would you like to create one now? (yes/no): yes
Username (leave blank to use 'vagrant'): admin
Email address: （無記入でEnter）
Password: admin
Password (again): admin
```

もし管理者ユーザを増やしたいときは、下記コマンドを実行することで上記と同じ対話モードを実行することができます。

```
$ python pj1/manage.py createsuperuser
```

MariaDBにログインしてみると、データベースの中にDjango用テーブルが作成されていることが確認できます。

```
(venv_app1) [vagrant@localhost django_apps]$  mysql -u app1_user -papp1_passwd

MariaDB [app1_db]> USE app1_db;
Database changed

MariaDB [app1_db]> SHOW TABLES;
+----------------------------+
| Tables_in_app1_db          |
+----------------------------+
| auth_group                 |
| auth_group_permissions     |
| auth_permission            |
| auth_user                  |
| auth_user_groups           |
| auth_user_user_permissions |
| django_admin_log           |
| django_content_type        |
| django_migrations          |
| django_session             |
+----------------------------+
10 rows in set (0.00 sec)
```

上記を実行すると、Django 管理サイトが自動で生成されます。
簡易Webサーバを立ち上げて、ホストマシンのWebブラウザで確認してみましょう。

```
(venv_app1) [vagrant@localhost django_apps]$ python pj1/manage.py runserver 0.0.0.0:8000
```

```
http://192.168.33.15:8000/admin/
```

すると以下の管理サイトが表示されます。  
[django_admin_snapshot](./django_admin_snapshot.png)

Webブラウザで先ほど入力した管理者情報(ここではID:admin, PASS:admin)を入力すると管理サイトに入ることができます。  
[django_admin_login](./django_admin_login.png)

データベースを直接触らずとも、この管理者画面で管理者ユーザやグループを追加することができます。

Djangoではユーザ管理に限らず、データベースが持つすべての情報をこの管理サイトで追加・修正・削除することができます。
このように専用アプリケーションを自分で作成せずとも、自動でデータベースの管理機能を作成してくれるところがDjangoの非常に便利な点です。

## Djangoアプリケーションを生成
次に、Djangoアプリケーションを作っていきます。

```
(venv_app1) [vagrant@localhost django_apps]$ cd /vagrant/django_apps/pj1/
(venv_app1) [vagrant@localhost pj1]$ python manage.py startapp app1
```

アプリケーションを生成すると、pj1ディレクトリ配下に、さらにapp1ディレクトリが作成されます。
pj1/pj1ディレクトリ配下のファイル群が、全アプリケーションに共通する設定ファイルであることに対して、
pj1/app1ディレクトリ配下のファイル群がDjangoアプリケーション固有の設定ファイルです。

```
pj1/
|-- app1
|   |-- __init__.py
|   |-- admin.py
|   |-- migrations
|   |   `-- __init__.py
|   |-- models.py
|   |-- tests.py
|   `-- views.py
|-- manage.py
`-- pj1
    |-- __init__.py
    |-- __pycache__
    |   |-- __init__.cpython-34.pyc
    |   `-- settings.cpython-34.pyc
    |-- settings.py
    |-- urls.py
    `-- wsgi.py
```

## モデルを定義する
ここからアプリケーションの中身を作っていきます。
まずモデルを定義し、データベースのテーブル構造を定義していきます。

Djangoでは、アプリケーションごとのディレクトリ配下のmodels.pyを修正し、
テーブルをClass、フィールドを変数として定義していきます。

### モデル要件
ここでは、IPアドレスを管理するアプリケーションを想定して、以下のようなモデルを定義します。

- IPアドレスの利用状況を管理するアプリケーション
  - IP addressテーブル
    - IP addressフィールド
      - 所有するIPv4アドレス群を示す
      - ネットワークセグメントは無視して、/32のアドレス単体を管理
      - 値は一意
    - Statusフィールド
      - IPアドレスの利用状況のステータスを保持
      - in_use / avalable / not_available の３つの状態を持つ
      - ブランクの状態は許可しない
    - Descriptionフィールド
      - IPアドレスの利用状況についてのメモ
      - 自由記述可
      - ブランクの状態を許可する

### models.pyの編集
モデル要件を満たすように pj1/app1/models.pyを以下のように編集します。
status_listという状態候補を用意することで、管理サイトやWebアプリケーションから
プルダウンメニューでの選択を指定することができます。
```
(venv_app1) [vagrant@localhost django_apps]$ vi pj1/app1/models.py
```

```py
from django.db import models

class Ipaddress(models.Model):

    status_list = (
        ('in_use', 'in use'),
        ('available', 'available'),
        ('not_available', 'not available')
    )
    ipaddress = models.GenericIPAddressField(verbose_name='IP address', unique=True)
    status = models.CharField(verbose_name='Status', choices=status_list, max_length=16)
    description = models.CharField(verbose_name='Discription', blank = True, max_length=255)
```

### Djangoアプリケーションのモデル定義を有効化
次にDjangoプロジェクトに対して、「app1」のモデル定義を有効化します。

pj1/pj1/settings.pyの「INSTALLED_APPS」に追記します。

```
(venv_app1) [vagrant@localhost django_apps]$ vi pj1/pj1/settings.py
```

```
INSTALLED_APPS = (
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    #アプリケーション名を追記
    'app1'
 )
```

### モデル定義をデータベースに反映
次に追加したモデルをデータベースに反映させます。

以下のコマンドでデータベースとの差分内容をSQL文にして発行します。
(この時点ではSQLの実行はされていません。)

```
(venv_app1) [vagrant@localhost django_apps]$ python pj1/manage.py makemigrations app1

Migrations for 'app1':
  0001_initial.py:
    - Create model IPaddress
```

発行されたSQL文は下記コマンドで確認できます。

```
(venv_app1) [vagrant@localhost django_apps]$ python pj1/manage.py sqlmigrate app1 0001

BEGIN;
CREATE TABLE `app1_ipaddress` (`id` integer AUTO_INCREMENT NOT NULL PRIMARY KEY, `ipaddress` char(39) NOT NULL UNIQUE, `status` varchar(16) NOT NULL, `description` varchar(255) NOT NULL);

COMMIT;
```

次に、以下のコマンドで発行されたSQL文を実行します。この時点でデータベースが更新されます。

```
(venv_app1) [vagrant@localhost django_apps]$ python pj1/manage.py migrate

Operations to perform:
  Synchronize unmigrated apps: staticfiles, messages
  Apply all migrations: admin, app1, auth, contenttypes, sessions
Synchronizing apps without migrations:
  Creating tables...
    Running deferred SQL...
  Installing custom SQL...
Running migrations:
  Rendering model states... DONE
  Applying app1.0001_initial... OK
```

MariaDBで確認すると新たにテーブル「app1_ipaddress 」が追加されたことが確認できます。

```
(venv_app1) [vagrant@localhost django_apps]$ mysql -u app1_user -papp1_passwd

MariaDB [(none)]> USE app1_db;
Database changed

MariaDB [app1_db]> SHOW TABLES;
+----------------------------+
| Tables_in_app1_db          |
+----------------------------+
| app1_ipaddress              |
| auth_group                 |
| auth_group_permissions     |
| auth_permission            |
| auth_user                  |
| auth_user_groups           |
| auth_user_user_permissions |
| django_admin_log           |
| django_content_type        |
| django_migrations          |
| django_session             |
+----------------------------+
11 rows in set (0.00 sec)
```

以降、models.pyを編集するたびに「python pj1/manage.py makemigrations app1」「python pj1/manage.py migrate」の２つのコマンドを実行することでデータベースを変更していきます。

### 管理サイトでアプリケーション用テーブルを登録
データベースは更新されましたが、現時点では管理サイト上にapp1用のテーブルは追加されていません。
管理サイトで管理するためにはapp1ディレクトリ配下のadmin.pyを編集していきます。

```
vi pj1/app1/admin.py
```

```
from django.contrib import admin

# 追記
from app1.models import Ipaddress

# 追記
admin.site.register(Ipaddress)
```

簡易Webサーバを立ち上げ、ホストマシンのWebブラウザで管理サイトを確認します。

```
(venv_app1) [vagrant@localhost django_apps]$ python pj1/manage.py runserver 0.0.0.0:8000
```

```
http://192.168.33.15:8000/admin/
```

このようにDjango 管理サイトにapp1のテーブル情報が表示されます。
[django_admin_app1_snapshot](./django_admin_app1_snapshot.png)

「追加」のボタンを押すことで、Ipaddressテーブルに値が追加できます。
[django_admin_app1_regist_snapshot](./django_admin_app1_regist_snapshot.png)

管理サイトを使って、3つのIPアドレス情報を登録してみます。

```
IP address : 192.168.0.0
Status: not_available
Discription: Network address

IP address : 192.168.0.1
Status: available
Discription: (blank)

IP address : 192.168.0.2
Status: available
Discription: (blank)
```

ここで管理サイト上に、登録した情報を出力させるには、models.pyに__str__関数を追加する必要があります。

```
(venv_app1) [vagrant@localhost django_apps]$ vi pj1/app1/models.py
```

```py
from django.db import models

class Ipaddress(models.Model):
    status_list = (
        ('in_use', 'in use'),
        ('available', 'available'),
        ('not_available', 'not available')
    )
    ipaddress = models.GenericIPAddressField(verbose_name='IP address', unique=True)
    status = models.CharField(verbose_name='Status', choices=status_list, max_length=16)
    description = models.CharField(verbose_name='Discription', blank = True, max_length=255)

    # ここを追加
    def  __str__(self):
        return self.ipaddress
```
このようにすると、IPaddressテーブルの中身を表示させることができるようになります。
[django_admin_app1_regist2_snapshot](django_admin_app1_regist2_snapshot.png)

さらに、IPaddressテーブルのフィールド情報も表示させる場合はadmin.pyを編集します。

```
(venv_app1) [vagrant@localhost django_apps]$ vi pj1/app1/admin.py
```

```
from django.contrib import admin
from app1.models import Ipaddress

# 以下を追加
class IpaddressAdmin(admin.ModelAdmin):
    list_display = ('ipaddress', 'status', 'description')

admin.site.register(Ipaddress, IpaddressAdmin)
```

このようにすることで管理サイトの表示をカスタマイズしていくことができます。
[django_admin_app1_regist3_snapshot](django_admin_app1_regist3_snapshot.png)


念のため、実際のデータベースで正しくアドレス情報が追加されたか確認しておきます。

```
(venv_app1) [vagrant@localhost django_apps]$ mysql -u app1_user -papp1_passwd

MariaDB [(none)]> USE app1_db;

MariaDB [app1_db]> SELECT * FROM app1_ipaddress;
+----+-------------+--------------+-----------------+
| id | ipaddress   | status       | description     |
+----+-------------+--------------+-----------------+
|  1 | 192.168.0.0 | not_avilable | Network address |
|  2 | 192.168.0.1 | available    |                 |
|  3 | 192.168.0.2 | available    |                 |
+----+-------------+--------------+-----------------+
3 rows in set (0.00 sec)
```

このようにして、DjangoではSQLコマンドを叩かずに管理サイトからレコード情報を追加・削除していくことができます。

### Tips データベースのレコード情報をテキストファイルに書き出す
データベースを扱っていると、その時点での情報を書き出す場面が出て来ます。
Djangoではデータベースのレコード情報をJSON/XML/YAMLのいずれかの形式で書き出す機能があります。
ここではJSON形式で書き出します。

```
(venv_app1) [vagrant@localhost django_apps]$ mkdir pj1/app1/dumpdata

(venv_app1) [vagrant@localhost django_apps]$ python pj1/manage.py dumpdata app1 --format=json --indent=2 > pj1/app1/dumpdata/ap1_db_dump.json
```

```
(venv_app1) [vagrant@localhost django_apps]$ cat pj1/app1/dumpdata/ap1_db_dump.json
[
{
  "pk": 1,
  "fields": {
    "status": "not_avilable",
    "ipaddress": "192.168.0.0",
    "description": "Network address"
  },
  "model": "app1.ipaddress"
},
{
  "pk": 2,
  "fields": {
    "status": "available",
    "ipaddress": "192.168.0.1",
    "description": ""
  },
  "model": "app1.ipaddress"
},
{
  "pk": 3,
  "fields": {
    "status": "available",
    "ipaddress": "192.168.0.2",
    "description": ""
  },
  "model": "app1.ipaddress"
}
]
```

作成されたファイルをgitで管理しておけば、レコード情報を時系列で管理することができます。  
なお、書き出したファイルを読み込みたい場合は、下記のコマンドを実施します。

```
(venv_app1) [vagrant@localhost django_apps]$ python pj1/manage.py loaddata pj1/app1/dumpdata/ap1_db_dump.json

Installed 3 object(s) from 1 fixture(s)
```

## DjangoアプリケーションのViewを定義
モデル部分ができたので、次はDjangoアプリケーション表示を決定するViewを作っていきます。

### URLを定義
まずはWebアプリケーションのURLを定義します。
pj1/pj1/ulrs.pyを編集することでDjangoアプリケーション単位のURLを定義します。
Djangoアプリケーションごとのページなどのさらに細かいURLを定義するときは、
アプリケーションディレクトリ配下のurls.pyを作成して追加してます。

まずpj1/pj1/ulrs.pyを修正します。
ここでは、 以下のURLが有効になるように設定しています。
- http://192.168.33.15:8000/app1/

```
 (venv_app1) [vagrant@localhost django_apps]$ vi pj1/pj1/urls.py
```

```py
from django.conf.urls import include, url
from django.contrib import admin

urlpatterns = [
    url(r'^admin/', include(admin.site.urls)),

    # 下記を追加
    url(r'^app1/', include('app1.urls', namespace = 'app1')),
]
 ```

次に、アプリケーションディレクトリ配下に新たにulrs.pyを作成します。
ここでは以下のURLが有効になるように設定しています。
- http://192.168.33.15:8000/app1/ipaddress/

```
 (venv_app1) [vagrant@localhost django_apps]$ vi pj1/app1/urls.py
```

```py
from django.conf.urls import patterns, url
from app1 import views

urlpatterns = patterns('',
    url(r'^ipaddress/^$', views.index, name='ipaddress_list'),
)
```


### Viewを定義( Hello Django編 )
アプリケーションディレトクリ配下のveiws.pyを編集することでview定義します。
まずは最も単純な「Hello Django」と表示するだけのViewを定義します。

```
(venv_app1) [vagrant@localhost django_apps]$ vi pj1/app1/models.py
```

```py
from django.http import HttpResponse

def ipaddress(request):
    return HttpResponse("Hello Django")
```

簡易Webサーバを立ち上げて、ホストマシンのWebブラウザで確認してみます。

```
(venv_app1) [vagrant@localhost django_apps]$ python pj1/manage.py runserver 0.0.0.0:8000
```

```
http://192.168.33.15:8000/app1/ipaddress/
```

[django_hello.png](./django_hello.png)

単純な文字表示だけのWebアプリケーションであればこのように作成できますが、
もう少し情報の多いWebアプリケーションはテンプレートエンジンやBootstrapなどのCSSフレームワークを利用して開発していきます。

### Bootstrapのインストール
以降のWebアプリケーションでは、CSSフレームワークであるBootstrapを使っていきます。
Bootstrapを利用すると、最低限のhtml記述だけで見栄えが整ったWebページを作成することができます。

まずBootstrapをダウンロードします。

```
(venv_app1) [vagrant@localhost django_apps]$ sudo wget https://github.com/twbs/bootstrap/releases/download/v3.3.5/bootstrap-3.3.5-dist.zip

(venv_app1) [vagrant@localhost django_apps]$ sudo yum install -y unzip

(venv_app1) [vagrant@localhost django_apps]$ sudo unzip bootstrap-3.3.5-dist.zip
```

正しく展開されると、下記のようなディレクリ構成になっています。

```
bootstrap-3.3.5-dist
|-- css
|   |-- bootstrap-theme.css
|   |-- bootstrap-theme.css.map
|   |-- bootstrap-theme.min.css
|   |-- bootstrap.css
|   |-- bootstrap.css.map
|   `-- bootstrap.min.css
|-- fonts
|   |-- glyphicons-halflings-regular.eot
|   |-- glyphicons-halflings-regular.svg
|   |-- glyphicons-halflings-regular.ttf
|   |-- glyphicons-halflings-regular.woff
|   `-- glyphicons-halflings-regular.woff2
`-- js
    |-- bootstrap.js
    |-- bootstrap.min.js
    `-- npm.js
```

次に、Bootstrapで利用するJavaScriptライブラリであるjQueryをダウンロードします。

```
(venv_app1) [vagrant@localhost django_apps]$ sudo wget http://code.jquery.com/jquery-2.1.4.min.js
```


jQueryのファイルを、さきほどダウンロードしたboostrapの「bootstrap-3.3.5-dist/js/」に移動します。

```
(venv_app1) [vagrant@localhost django_apps]$ mv jquery-2.1.4.min.js bootstrap-3.3.5-dist/js/
```

```
bootstrap-3.3.5-dist
|-- css
|   |-- bootstrap-theme.css
|   |-- bootstrap-theme.css.map
|   |-- bootstrap-theme.min.css
|   |-- bootstrap.css
|   |-- bootstrap.css.map
|   `-- bootstrap.min.css
|-- fonts
|   |-- glyphicons-halflings-regular.eot
|   |-- glyphicons-halflings-regular.svg
|   |-- glyphicons-halflings-regular.ttf
|   |-- glyphicons-halflings-regular.woff
|   `-- glyphicons-halflings-regular.woff2
`-- js
    |-- bootstrap.js
    |-- bootstrap.min.js
    |-- jquery-2.1.4.min.js
    `-- npm.js
```

次に「pj1/app1」のアプリケーションディレクトリ配下に「static」ディレクトリを新規作成し、
さきほどのbootstrap-3.3.5-distディレクトリ配下の全ファイルを を移動させます。

```
(venv_app1) [vagrant@localhost django_apps]$ mkdir pj1/app1/static
(venv_app1) [vagrant@localhost django_apps]$ mv bootstrap-3.3.5-dist/* pj1/app1/static/
```

最終的にこのようなディレクトリ構成になります。

```
pj1
|-- app1
|   |-- __init__.py
|   |-- __pycache__
|   |   |-- __init__.cpython-34.pyc
|   |   |-- admin.cpython-34.pyc
|   |   `-- models.cpython-34.pyc
|   |-- admin.py
|   |-- dumpdata
|   |   `-- ap1_db_dump.json
|   |-- migrations
|   |   |-- 0001_initial.py
|   |   |-- __init__.py
|   |   `-- __pycache__
|   |       |-- 0001_initial.cpython-34.pyc
|   |       `-- __init__.cpython-34.pyc
|   |-- models.py
|   |-- static
|   |   |-- css
|   |   |   |-- bootstrap-theme.css
|   |   |   |-- bootstrap-theme.css.map
|   |   |   |-- bootstrap-theme.min.css
|   |   |   |-- bootstrap.css
|   |   |   |-- bootstrap.css.map
|   |   |   `-- bootstrap.min.css
|   |   |-- fonts
|   |   |   |-- glyphicons-halflings-regular.eot
|   |   |   |-- glyphicons-halflings-regular.svg
|   |   |   |-- glyphicons-halflings-regular.ttf
|   |   |   |-- glyphicons-halflings-regular.woff
|   |   |   `-- glyphicons-halflings-regular.woff2
|   |   `-- js
|   |       |-- bootstrap.js
|   |       |-- bootstrap.min.js
|   |       |-- jquery-2.1.4.min.js
|   |       `-- npm.js
|   |-- tests.py
|   |-- urls.py
|   `-- views.py
|-- manage.py
`-- pj1
    |-- __init__.py
    |-- __pycache__
    |   |-- __init__.cpython-34.pyc
    |   |-- settings.cpython-34.pyc
    |   |-- urls.cpython-34.pyc
    |   `-- wsgi.cpython-34.pyc
    |-- settings.py
    |-- urls.py
    `-- wsgi.py
```

次に、Django からBootsrapを操作するために django-bootstrap-formというPythonパッケージをインストールします。

```
(venv_app1) [vagrant@localhost django_apps]$ pip install django-bootstrap-form

(venv_app1) [vagrant@localhost django_apps]$ pip list
Django (1.8.4)
django-bootstrap-form (3.2)
mysqlclient (1.3.6)
pip (7.1.2)
setuptools (12.0.5)
uWSGI (2.0.11.1)
```

次に、pj1/setting.pyのINSTALLED_APPSに、django-bootstrap-formを追記します。

```
(venv_app1) [vagrant@localhost django_apps]$ vi pj1/settings.py


INSTALLED_APPS = (
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    ##以下を追記
    'bootstrapform',

    'app1'
)
```

### テンプレートを作成
HTMLファイルのテンプレートを作っていきます。DjangoではJinja2というテンプレートエンジンを使い、
HTMLテンプレートに変数を埋め込むことでWebアプリケーションを作成していきます。

テンプレートエンジン Jinja2の使い方は[公式ページ](http://jinja.pocoo.org/docs/dev/)を参照してください。
{{  }} や {%  %} で括った値を変数やプログラミング関数として扱うことができるのがJinja2の特徴です。

今回のIPアドレス管理アプリケーションに合わせて、HTMLテンプレートを作成します。
テンプレートは、「app1/templates/」ディレクトリ配下に配置します。

ここでは、BootstrapのGetting Startedで紹介されている以下のページを参考に作っています。
http://getbootstrap.com/examples/starter-template/

```
(venv_app1) [vagrant@localhost django_apps]$ mkdir pj1/app1/templates
(venv_app1) [vagrant@localhost django_apps]$ vi pj1/app1/templates/ipaddress.html
```

```
{% load staticfiles %}

<!-- Adjust  bootstrap  navbar -->
body { padding-top: 40px; }

<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">

    <title>App1</title>

    <link href="{% static 'css/bootstrap.min.css' %}" rel="stylesheet">
    <link href="{% static 'css/bootstrap-theme.min.css' %}" rel="stylesheet">
    <script src="{% static 'js/jquery-2.1.4.min.js' %}"></script>
    <script src="{% static 'js/bootstrap.min.js' %}"></script>
  </head>

  <body>
    <nav class="navbar navbar-inverse navbar-fixed-top">
      <div class="container">
        <div class="navbar-header">
          <button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#navbar" aria-expanded="false" aria-controls="navbar">
            <span class="sr-only">Toggle navigation</span>
            <span class="icon-bar"></span>
            <span class="icon-bar"></span>
            <span class="icon-bar"></span>
          </button>
          <a class="navbar-brand" href="#">App1</a>
        </div>
        <div id="navbar" class="collapse navbar-collapse">
          <ul class="nav navbar-nav">
            <li class="active"><a href="#">IP address</a></li>
            <li><a href="#">Function 2</a></li>
            <li><a href="#">Function 3</a></li>
          </ul>
        </div><!--/.nav-collapse -->
      </div>
    </nav>

    <div class="container">
        <h1 class="page-header">IP address</h1>
        <table class="table table-striped table-bordered">

            <thead>
                <tr>
                    <td>IP address</td>
                    <td>Status</td>
                    <td>Description</td>
                </tr>
            </thead>
            <tbody>
                {% for ipaddress in ipaddresses %}
                <tr>
                    <td>{{ ipaddress.ipaddress }}</td>
                    <td>{{ ipaddress.status }}</td>
                    <td>{{ ipaddress.description }}</td>
                </tr>
                {% endfor %}
            </tbody>
        </table>

    </div><!-- /.container -->

  </body>
</html>
```

### Viewを定義( IPアドレス表示編 )
作成したテンプレートに合わせて、データベースにあるIPアドレスの情報を表示させるアプリケーションを作ってみます。
pj1/app1/views.py を以下のように修正します。

```
(venv_app1) [vagrant@localhost django_apps]$ vi pj1/app1/models.py
```

```py
from django.http import HttpResponse
from django.shortcuts import render_to_response
from django.template import RequestContext
from app1.models import Ipaddress

def ipaddress(request):
    # データベース上のIPアドレス情報を配列型で取得
    ipaddresses = Ipaddress.objects.all().order_by('id')

    return  render_to_response(
        'ipaddress.html', # テンプレート名を指定
        {'ipaddresses' : ipaddresses }, # 取得したIPアドレス情報をテンプレート内の変数に代入
        context_instance=RequestContext(request)
        )
```

簡易Webサーバを立ち上げて、ホストマシンのWebブラウザで確認してみます。

```
(venv_app1) [vagrant@localhost django_apps]$ python pj1/manage.py runserver 0.0.0.0:8000
```

```
http://192.168.33.15:8000/app1/ipaddress/
```

[django_app_ipaddresslist](./django_app_ipaddresslist.png)

このようにテンプレートファイルとView定義を組み合わせることで、比較的少ない記述でDjangoアプリケーションを作成することができます。

## Viewを定義( IPアドレス管理編 )
次はアプリケーションにIPアドレス情報の変更機能を追加してみます。



# Webサーバとの連動
