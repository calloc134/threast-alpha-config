# threast-ALpha config

このリポジトリは、threast-ALphaにおける
 - nginxコンフィグファイル
 - 環境変数ファイル
 - Dockerfile
 - docker-compose構成ファイル

などをまとめたものである。

## Dockerfile一覧
### 開発環境
以下のDockerfileを使用する。  
 - Nest-dev-Dockerfile
   - NestJSの開発環境用のコンテナ
   - ベースは`node:latest`
   - ビルド時に`@nestjs/cli`をインストール
   - 起動時には以下の動作を行う
     - 必要なパッケージ群をインストール
     - Prisma Studioを起動
     - NestをwebpackのHMRオプション有効で起動
 - React-Dev-Dockerfile
   - Reactの開発環境用のコンテナ
   - ベースは`node:latest`
   - 起動時には以下の動作を行う
     - 必要なパッケージ群をインストール
     - Viteサーバを起動

### 本番環境
以下のDockerfileを使用する。
 - Nest-prod-Dockerfile
   - NestJSの本番環境用のコンテナ
   - ベースは`node:latest`
   - ビルド時に`@nestjs/cli`をインストール
   - 起動時には以下の動作を行う
     - 必要なパッケージ群をインストール
     - デプロイ環境としてPrismaをマイグレーション
     - NestJSを本番環境として実行
 - React-prod-Dockerfile
   - Reactの本番環境用のコンテナ
   - ベースは`node:latest`
   - このコンテナはビルドのみを行う
   - 起動時には以下の動作を行う
     - 必要なパッケージ群をインストール
     - Viteをビルドし、distディレクトリに出力

## nginxコンフィグファイル一覧
### 開発環境
nginx-dev内のdefault.confを使用する。
内容は以下の通り。
 - ポート80でHTTP接続を受け付ける
 - ポート443でHTTPS接続を受け付ける
   - ポート443の接続は証明書を必要とする
 - パスが/apiの場合はNestJSのサーバに接続する
   - リバースプロキシとして動作
   - ユーザのIPをX-Forwarded-Forヘッダとその他に追加して送信
 - それ以外の場合はViteサーバに接続する
   - リバースプロキシとして動作
   - HMRを有効にするためにwebsocketも動作するように設定

### 本番環境
nginx-prod内のdefault.confを使用する。
内容は以下の通り。
 - ポート80でHTTP接続を受け付ける
 - ポート443でHTTPS接続を受け付ける
   - ポート443の接続は証明書を必要とする
 - パスが/apiの場合は開発環境に同じ
 - それ以外の場合はビルドされたReactのHTMLファイルをサーブ
   - Viteサーバは存在せずnginxのファイル配信のみで動作
## docker composeファイル一覧

### compose-dev.yml
開発環境時に使用するdocker-composeファイル。  
以下のコンテナを使用する。

 - postgresql
   - docker_networkに接続
 - nest-dev
   - node_modulesを除くソースコードをマウント
   - docker-networkに接続
   - デバッグ用にポート5555をフォワードしPrisma Studioへの接続を許可
 - react-dev
   - node_modulesを除くソースコードをマウント
   - docker-networkに接続
 - nginx
   - docker-networkに接続
   - ポート80でHTTP接続をフォワード
   - ポート443でHTTPS接続をフォワード

また、docker compose内部で使用するネットワークは、以下の通り。

 - docker-network


### compose-mig.yml
マイグレーション用のdocker-composeファイル。
以下のコンテナを使用する。

 - postgresql
   - docker-networkに接続
 - nest-mig
   - docker-networkに接続
   - ttyを有効化しターミナルへの接続を許可
   - ここで`npx prisma migrate dev`などのコマンドを実行できる


### compose-prod.yml
本番環境時に使用するdocker-composeファイル。
以下のコンテナを使用する。

 - nest-prod
   - devに同じ
 - react-prod
   - devとは異なりreactのビルドのみをviteで行う
   - ビルド結果がdistに出力される
 - nginx
   - 前述の通り
 - postgresql
   - devに同じ

docker compose内部で使用するネットワークもdevに同じ
