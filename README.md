# ローカル環境の自己署名証明書

## 目的

ローカル環境でSSLができるようにしたい<br>
<br>

## 参考URL

paulczar / omgwtfssl<br>
https://github.com/paulczar/omgwtfssl  
https://hub.docker.com/r/paulczar/omgwtfssl/


自己署名証明書を生成してくれるdockerコンテナOMGWTFSSLを試す<br>
https://mistymagich.wordpress.com/2019/04/15/%E8%87%AA%E5%B7%B1%E7%BD%B2%E5%90%8D%E8%A8%BC%E6%98%8E%E6%9B%B8%E3%82%92%E7%94%9F%E6%88%90%E3%81%97%E3%81%A6%E3%81%8F%E3%82%8C%E3%82%8Bdocker%E3%82%B3%E3%83%B3%E3%83%86%E3%83%8Aomgwtfssl%E3%82%92/


OMGWTFSSLでDocker開発環境をSSL化<br>
https://engrave.work/entry/436


## その他

↓別の方法として **mkcert** というのがある  

【図解付き】開発用オレオレ認証局SSL通信(+dockerコンテナ対応) : 2021
https://qiita.com/kaku3/items/e06a02ae1068de5c0663


↓または、認証局を作成する方法  
Dockerのコンテナでオレオレ認証局を立ててサーバー証明書に署名する
https://qiita.com/papparapa/items/1cef886b2514383157d0


## 謝辞

先人たちの知恵に敬意と多謝。

<br>

---

## 環境

- ホスト
  - Windows10
  - Vagrant
  - VirtualBox
- ゲスト
  - Ubuntu 20.04
    - IPアドレス：192.1698.33.50


### ●ディレクトリ構成

ホスト側設置している **docker_data** を Vagrant の共有機能でゲスト側と共有。


**my_cert/cert_data** に各ドメインのフォルダを作成し、そこへ出力させる。

```
docker_data
└ my_cert
  ├ cert_data
  │  ├ hoge.jp
  │  ├ fuga.jp
  │  ├ ...
  │  └ fazz.jp
  └ README.md

```

<br>

---

## 作成

### ローカルドメインについて

hostsファイルに登録していても、「.localhost」、「.local」は使用しない方がよいらしい。  

●参考URL<br>

hostsを設定してもChromeで反映されない<br>
https://qiita.com/hasht/items/477dd562206db4716cb4

GoogleChromeでhoge.localhostにアクセスできなくなった<br>
https://qiita.com/iida-hayato/items/4e39e5fc43e707a40b5a

chrome subdomain.localhostにアクセスできない<br>
https://okamerin.com/nc/title/684.htm

<br>

### ワイルドカード形式で作成

```
$ docker run -e SSL_SUBJECT="*.hoge.jp" -v /docker_data/my_cert/cert_data/hoge.jp:/certs  paulczar/omgwtfssl
```

## 設定

### ブラウザ

**ca.pem** をOSまたはブラウザに登録する。  
登録する際は「信頼されたルート証明機関」として登録する。


### Nginx

出力された **cert.pem**、**key.pem** を設定ファイルへ記述する。  


```
ssl_certificate /tmp/certs/cert.pem;
ssl_certificate_key /tmp/certs/key.pem;
ssl_session_timeout 5m;
ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
ssl_ciphers         HIGH:!aNULL:!MD5;
```
