## Kubernetesハンズオン
#### DockerとKubernetesを触ってみる

---
## 事前準備

***
### OpenStackから作成した<br>インスタンスにログイン
```bash
$ ssh -i ~~~.pem ubuntu@xx.xx.xx.xx
```

---
### リポジトリをclone
```bash
$ git clone https://github.com/kuesy/kubernetes_training
```

---
### (Dockerを入れていない場合)
```bash
$ sudo apt update && sudo apt install -y docker.io

$ sudo usermod -aG docker ubuntu
$ sudo systemctl restart docker

$ exit
$ ssh -i ...
```

---
### (k8s環境を入れていない場合)
```bash
# minikubeで構築する
$ wget https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
$ sudo chmod +x minikube-linux-amd64
$ sudo mv minikube-linux-amd64 /usr/local/bin/minikube
$ minikube start

# kubectlを入れる
$ curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
$ sudo chmod +x kubectl
$ sudo mv kubectl /usr/local/bin/kubectl
```

***
## Dockerハンズオン

---
### Dockerについて(振り返り)
- コンテナ型の仮想化技術
- どのOSからでも実行できる
- Dockerfile => Image => Containerで動作する
![docker](./img/about_docker.png)

---
### Dockerを触ってみる! - その1
コンテナ環境にnginxを立ち上げてみる

***
### 1. 対象のディレクトリに移動
```bash
$ cd kubernetes_training/01_dockerfile/01_web_server_apache

# Dockerfileとindex.htmlが入っていることを確認
$ ls
```

---
### 2. Dockerfile => Docker Image
イメージ名: apache_test
```bash
$ docker build -t apache_test ./

# dockerイメージが生成されたことを確認
$ docker image ls
```
---
### 3. Dockerfileの中身について
1. Ubuntuを入れて諸々インストール  
2. ローカルのindex.htmlを/var/www/htmlにコピー  
3. (コンテナ起動時)apacheを起動するコマンド実行

```dockerfile
FROM ubuntu # ベースのイメージをdocker hubから取得
MAINTAINER hoge<fuga@piyo.co.jp> # 
RUN apt-get update -y && \
apt-get install -y tzdata && \
apt-get install -y apache2
COPY index.html /var/www/html/
EXPOSE 80
CMD ["apachectl", "-D", "FOREGROUND"]
```

---
### (index.html)
シンプルなやつ:grinning:
```html
<!doctype html>
<html>
<head>
  <meta charset="utf-8">
  <title>Example</title>
</head>
<body>
Dockerテスト
</body>
</html>
```

---
### 4. Dockerイメージ => Dockerコンテナ
ローカルの8000ポートを叩くと対象イメージの
80ポートに向くようにして実行！
```bash
$ docker run -p 8000:80 -d apache_test

# プロセスが動き始めたことを確認
$ docker ps
```

---
### 5. 動作確認
8000ポートにリクエストを投げて返却されたらOK！
```bash
$ curl http://localhost:8000
Dockerテスト
```

---
### 6. 後処理(コンテナの削除)
```bash
# 動作しているコンテナのIDを確認
$ docker ps

# 対象のコンテナを止めて削除
$ docker stop (container id)
$ docker rm (container id)

# 対象のコンテナが消えたことを確認
$ docker ps
```

---
### 7. 後処理(イメージの削除)
イメージ名を指定して消すだけ！
```bash
$ docker rmi apache_test

# apache_testが消えていることを確認
$ docker image ls
```
***
### Dockerを触ってみる! - その2
コンテナ環境にアプリケーションを立ち上げてみる

***
### 1. 対象のディレクトリに移動
```bash
$ cd kubernetes_training/01_dockerfile/02_web_server_app

# Dockerfileとhello.jsが入っていることを確認
$ ls
```

---
### 2. Dockerfile => Docker Image
イメージ名: app_test
```bash
$ docker build -t app_test ./

# dockerイメージが生成されたことを確認
$ docker image ls
```
---
### 3. Dockerfileの中身について
1. 12系のnode.jsをベースとしてインストール
2. ローカルのhello.jsを/hello.jsにコピー
3. (コンテナ起動時)node hello.jsが起動

```dockerfile
FROM node:12
COPY hello.js /hello.js
EXPOSE 3000
CMD ["node", "hello"]
```

---
### (hello.js)
3000ポートにリクエスト投げたらHello Worldが返る:grinning:
```javascript
const http = require('http');

const hostname = '0.0.0.0';
const port = 3000;

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hello World\n');
});

server.listen(port, hostname, () => {
  console.log(`Server running at http://${hostname}:${port}/`);
});
```

---
### 4. Dockerイメージ => Dockerコンテナ
ローカルの3000ポートを叩くと対象イメージの
3000ポートに向けるようにして実行！
```bash
$ docker run -p 3000:3000 -d app_test

# プロセスが動き始めたことを確認
$ docker ps
```

---
### 5. 動作確認
3000ポートにリクエストを投げて返却されたらOK！
```bash
$ curl http://localhost:8000
Hello World
```

---
### 6. 後処理
```bash
# 動作しているコンテナのIDを確認
$ docker ps

# 対象のコンテナを止めて削除
$ docker stop (container id)
$ docker rm (container id)

# イメージの削除
$ docker rmi apache_test
```
***
### Dockerの基本的なコマンドまとめ
```bash
# Dockerイメージの作成
$ docker build -t <イメージ名> <Dockerfileのディレクトリ>
# Dockerイメージの確認
$ docker image ls
# Dockerイメージの削除
$ docker rmi <イメージ名>

# Dockerコンテナの起動
$ docker run -d <イメージ名>
# Dockerコンテナの起動(ローカルからコンテナに叩くようにする設定)
$ docker run -p <ローカルポート>:<コンテナポート> -d <イメージ名>
# Dockerコンテナの確認
$ docker ps 
# Dockerコンテナの停止
$ docker stop <container id>
# Dockerコンテナの削除
$ docker rm <container id>
```

---
## Kubernetesハンズオン

---
### Kubernetesについて(振り返り)
- 
![kubernetes](./hoge.png)


<style type="text/css">
  .reveal h1,
  .reveal h2,
  .reveal h3,
  .reveal h4,
  .reveal h5,
  .reveal h6 {
    text-transform: none;
  }
</style>
