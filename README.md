# ECSチュートリアル

##　DockerでFlask構築



環境構築はCloud9で行った　よってDockerはインストール済み
(Cloudshellで行おうとしたが、Dockerが入っておらず環境構築が面倒であった)

注意点：日本以外のところで Docker Buildを行うと、タイムゾーンのエラーが発生した

こちらの記事を参考に作成
https://tech-diary.net/how-to-create-flask-container/


~~~
mkdir python-docker
cd python-docker
touch Dockerfile
touch requirement
touch app.py
~~~

- Dockerfile

~~~
FROM ubuntu:20.04
USER root

RUN apt-get update
RUN apt-get install -y python3.10
RUN apt-get install -y python3-pip

COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt

WORKDIR /python
RUN flask run
~~~


- app.py
~~~
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello, World!"
~~~

- requirement
~~~
flask==2.2.2
~~~

dockerのイメージ作成
~~~
docker build -t docker-flask .
~~~


日本に変えてもエラーが出る、そして止まる

~~~
Please select the geographic area in which you live. Subsequent configuration
questions will narrow this down by presenting a list of cities, representing
the time zones in which they are located.

  1. Africa        6. Asia            11. System V timezones
  2. America       7. Atlantic Ocean  12. US
  3. Antarctica    8. Europe          13. None of the above
  4. Australia     9. Indian Ocean
  5. Arctic Ocean  
~~~

解決策は以下の記事を参考

https://tmyoda.hatenablog.com/entry/20210124/1611416396

https://zenn.dev/flyingbarbarian/scraps/1275681132babd


結論、以下Dockerfileを記載
~~~
FROM ubuntu:20.04
USER root


# RUN DEBIAN_FRONTEND=noninteractive apt-get update && apt-get install -y git
# RUN DEBIAN_FRONTEND=noninteractive apt-get update && apt-get install -y tzdata
ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y git
RUN apt-get update && apt-get install -y tzdata
RUN apt-get install -y python3.10
RUN apt-get install -y python3-pip

COPY hello.py hello.py
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt


CMD ["python3", "hello.py"]
~~~

hello.pyも少し編集
~~~
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello_world():
    return "<p>Hello, World!</p>"
    
if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0',port=5000)
~~~

注意1：Docerfileを修正してイメージを作成し直す場合、docker build -t <タグ名>としても、レポジトリ名やタグ名が<none>になってしまう場合があった。Dockerfileないのコマンドにエラーがあるとそのような仕組みなるらしく、さらにDockerfileを修正しても、キャッシュが残っているため修正されなかった。イメージを一度削除すること

注意2:flaskのportを5000で明示しておくと、わかりやすい。host=0.0.0.0にしておかないとおかしくなる（参考：https://qiita.com/amuyikam/items/01a8c16e3ddbcc734a46）


次に作成したイメージからコンテナ作成（なぜかコンテナ名を--nameで指定できずエラーになる。。。）
~~~
docker run -dp 5000:5000 docker-flask:1.1
~~~

以下で動作確認。レスポンスが返って来れば完了。
~~~
curl http://localhost:5000
~~~

以下コマンドでコンテナの実行状況がわかる
~~~
docker ps -a　
~~~

一度コンテナが起動するとともに止まってしまうことがあった。
その際は以下のコマンドを使用すると良い。ただのファイル名のミスだった。
~~~
PowerRole:~/environment/python-docker $ docker logs コンテナID
python3: can't open file 'app.py': [Errno 2] No such file or directory
python3: can't open file 'app.py': [Errno 2] No such file or directory
python3: can't open file 'app.py': [Errno 2] No such file or directory
~~~


NOTE：dockerイメージとコンテナをはっきりと区別すること

Dockerイメージ：Dockerコンテナを動作させるのに使うテンプレートファイル(ファイルと言ってもLinuxファイルシステムなどもあるので、スナップショットに近いもの)
Dockerコンテナ：テンプレートファイルに基づいてアプリケーションを実行する環境・インスタンス

~~~
docker build：イメージの作成
docker images -a:イメージの一覧
dokcer rmi:イメージの削除
docker run：イメージからコンテナの作成と起動
docker start/stop;コンテナの起動・停止
docker ps -a：コンテナの一覧表示
docker rm:コンテナの削除
~~~

参考：
https://www.kagoya.jp/howto/cloud/container/dockerimage/
https://qiita.com/zembutsu/items/24558f9d0d254e33088f




# 全体の参考になるかも
https://dev.classmethod.jp/articles/getting-start-docker-for-platformengineer/#toc-20