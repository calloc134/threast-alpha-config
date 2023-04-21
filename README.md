# threast-ALpha config

このリポジトリは、threast-ALphaにおける
 - nginxコンフィグファイル
 - 環境変数ファイル
 - Dockerfile
 - docker-compose構成ファイル
などをまとめたものである。

## Dockerfile一覧
### Nest
NestJSの開発環境を構築するためのDockerfile。  
ベースイメージは、node:latest  
起動時には、必要なパッケージ群をインストールしてPrisma Studioを起動してからNestを起動する。
### React
Reactの開発環境を構築するためのDockerfile。  
ベースイメージは、node:latest
ViteでReactのビルドを行うことを想定している。
起動時には、必要なパッケージ群をインストールしてからViteを起動。

## docker composeファイル一覧

### compose-dev.yml
開発環境時に使用するdocker-composeファイル。  
以下のコンテナを使用する。

 - nest-dev
 - react-dev
 - nginx
 - postgresql

また、docker compose内部で使用するネットワークは、以下の通り。

 - docker-network

postgresqlは、docker-networkに接続しており、nest-devより接続をすることができる。  
nest-devでも、同じようにnode_modulesを除くソースコードをマウントし、docker-networkに接続  
また、nest-devではデバッグ用にポート5555を開放し、Prisma Studioにクライアントから接続できるようにしている。  
react-devでは、node_modulesを除くソースコードをマウントし、docker-networkに接続  
nginxでは、docker-networkに接続し、ポート80でHTTP接続を、ポート443でHTTPS接続を開放している。  

### compose-mig.yml
マイグレーション用のdocker-composeファイル。
以下のコンテナを使用する。

 - nest-mig
 - postgresql

nest-migではttyを有効化しているため、コンテナ内でコマンドを実行することができる。
ここで、  
`npx prisma migrate dev`  
のようなコマンドを実行することで、マイグレーションを行うことができる。 

### compose-prod.yml
本番環境時に使用するdocker-composeファイル。
以下のコンテナを使用する。

 - nest-prod
 - react-prod
 - nginx
 - postgresql

docker compose内部で使用するネットワークは、以下の通り。
    
     - docker-network
ここで、react-prodについてはサーバの役割を持たず、reactのビルドを行うだけのコンテナである。  
postgresql, nest-prodの動作は基本的にcompose-dev.ymlと同じである。  
nginxでは、docker-networkに接続し、ポート80でHTTP接続を、ポート443でHTTPS接続を開放している。  
また、上記のdefault.confの設定より、/api以外のパスに対しては、reactのビルド結果出力されたHTMLをサーブするようになっている。