# Kyudo App - Docker環境

弓道アプリケーションのDocker環境セットアップと使用方法について説明します。

## 📋 概要

このプロジェクトは以下のサービスで構成されています：
- **Frontend**: Vue.js (Vite + TypeScript)
- **Backend**: Spring Boot (Kotlin)
- **Database**: MySQL 8.0

Spring Profilesを使用して、Docker環境とローカル開発環境の両方に対応しています。

## 🔧 前提条件

- Docker
- Docker Compose
- Java 17以上（ローカル開発時）
- Node.js（ローカル開発時）

## ⚙️ 環境設定

### 1. 環境変数ファイル

`.env`ファイルが`docker/`ディレクトリに必要です
.env.sampleを使用してください


### 2. Spring Profiles

バックエンドは以下のプロファイルで動作します：
- `dev`: ローカル開発用（localhost:13306のMySQLに接続）
- `docker`: Docker環境用（mysql:3306のMySQLに接続）

## 🚀 起動方法

### Docker環境（推奨）

**全サービス一括起動:**
```bash
cd docker
docker-compose up -d
```

**ログ確認:**
```bash
docker-compose logs -f
```

**サービス状況確認:**
```bash
docker-compose ps
```

### ローカル開発環境

**1. MySQLのみDocker起動:**
```bash
cd docker
docker-compose up mysql -d
```

**2. バックエンドをローカル起動:**
```bash
cd kyodoApp_backend
./mvnw spring-boot:run -Dspring-boot.run.profiles=dev
```

**3. フロントエンドをローカル起動:**
```bash
cd sample_app/my-kyudo-app
npm run dev
# または yarn dev
```

## 🌐 アクセスURL

| サービス | URL | 説明 |
|---------|-----|------|
| Frontend | http://localhost:5173 | Vue.jsアプリケーション |
| Backend API | http://localhost:8081 | Spring Boot REST API |
| MySQL | localhost:13306 | データベース |


## 📁 プロファイル設定詳細

### application.properties（共通設定）
```properties
server.port=8081
spring.application.name=kyudo_app
spring.jpa.hibernate.ddl-auto=none
jwt.expiration=3600000
```

### application-dev.properties（ローカル開発）
```properties
spring.datasource.url=jdbc:mysql://localhost:13306/kyudo_app
spring.datasource.username=${LOCAL_MYSQL_USER}
spring.datasource.password=${LOCAL_MYSQL_PASSWORD}
logging.level.com.example.kyudo_app=DEBUG
```

### application-docker.properties（Docker環境）
```properties
spring.datasource.url=jdbc:mysql://mysql:3306/kyudo_app
spring.datasource.username=${MYSQL_USER}
spring.datasource.password=${MYSQL_PASSWORD}
logging.level.com.example.kyudo_app=INFO
```

## 🛠️ トラブルシューティング

### コンテナが起動しない場合

**1. ポート競合の確認:**
```bash
lsof -i :5173  # Frontend
lsof -i :8081  # Backend
lsof -i :13306 # MySQL
```

**2. コンテナの再ビルド:**
```bash
docker-compose down
docker-compose build --no-cache
docker-compose up -d
```

**3. ログの確認:**
```bash
docker-compose logs backend
docker-compose logs frontend
docker-compose logs mysql
```

### データベース接続エラー

**MySQLの起動確認:**
```bash
docker-compose ps mysql
```

**データベース接続テスト:**
```bash
mysql -h localhost -P 13306 -u kotaro -p kyudo_app
```

## 🔄 停止・クリーンアップ

**サービス停止:**
```bash
docker-compose down
```

**ボリューム含めて完全削除:**
```bash
docker-compose down -v
```

**不要なイメージ削除:**
```bash
docker system prune -a
```

## 📝 開発の使い分け

| 用途 | 環境 | メリット |
|------|------|----------|
| 本番に近い動作確認 | Docker環境 | 環境の一貫性、デプロイと同じ環境 |
| 開発・デバッグ | ローカル環境 | 高速な起動、詳細ログ、デバッグ容易 |

## 📚 その他

- データベースマイグレーションはFlywayを使用
- JWTによる認証機能を実装
- CORS設定済み（フロントエンドとの連携対応）
- 問題が発生した場合は、以下を確認してください：
    1. Docker Composeバージョン
    2. 各サービスのログ
    3. ポート使用状況
    4. .envファイル設定
