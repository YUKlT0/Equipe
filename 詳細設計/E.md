
# 担当E：インフラ・非機能要件詳細設計書

---

## 1. 開発環境・実行環境仕様

| 項目               | 詳細                           |
|------------------|------------------------------|
| OS               | Ubuntu 22.04 LTS              |
| Javaバージョン     | OpenJDK 17                   |
| フレームワーク      | Spring Boot 3.0              |
| Webサーバ          | Nginx 1.24                   |
| アプリケーションサーバ | Spring Boot内蔵サーバ           |
| DB               | PostgreSQL 15                 |
| DB接続ドライバ      | PostgreSQL JDBC Driver 42.x   |
| メッセージキュー     | RabbitMQ 3.11                 |
| キャッシュ          | Redis 7.0                    |
| コンテナ環境       | Docker 23.0                  |
| CI/CD            | GitHub Actions               |

---

## 2. セキュリティ設計

### 2.1 認証・認可

- Spring Security を利用しJWT認証を実装  
- トークン有効期限は1時間、リフレッシュトークンを別途管理  
- 管理者APIはロールベースのアクセス制御（RBAC）で制限  
- HTTPS通信必須（証明書はLet’s Encryptを使用）  
- パスワードはbcryptでハッシュ化保存

### 2.2 入力値検証

- フロントエンド・サーバーサイド両方でバリデーション実施  
- SQLインジェクション対策としてプリペアドステートメント利用  
- クロスサイトスクリプティング（XSS）対策としてHTMLエスケープを徹底

### 2.3 ログ管理

- ログフォーマットはJSON形式で統一  
- アクセスログ、エラーログ、監査ログを分離管理  
- ログレベルはINFOを基本とし、必要に応じてDEBUG切り替え可能  
- ログはFluentd経由でElasticsearchに転送、Kibanaで可視化

---

## 3. 性能・可用性設計

### 3.1 性能要件

- APIレスポンスタイムは99パーセンタイルで500ms以下  
- 同時接続数はピーク時で1000セッションを想定

### 3.2 負荷対策

- DBに対してはクエリ最適化とインデックス設計を徹底  
- Redisキャッシュを導入し、商品一覧やカテゴリ情報の読み取り負荷を軽減  
- 水平スケーリング可能なコンテナ構成

### 3.3 可用性

- 主要コンポーネントは冗長構成（DBはマスタースレーブ構成）  
- アプリケーションサーバは複数台でロードバランシング（Nginxのラウンドロビン）  
- 障害検知はPrometheusで監視し、アラートをSlackに通知

---

## 4. 監視・運用設計

### 4.1 監視項目

| 項目               | 監視内容                          | アラート条件                        |
|------------------|-------------------------------|-------------------------------|
| サーバCPU負荷      | CPU使用率                      | 80%以上が5分以上継続             |
| メモリ使用率        | メモリ使用率                   | 80%以上が5分以上継続             |
| ディスク容量        | 空き容量                       | 10%未満                        |
| DB接続数           | 現在の接続数                   | 80%以上の閾値超過               |
| アプリケーションエラー数 | 5xxエラーの発生数              | 5分間で10件以上                  |
| レスポンスタイム    | API平均応答時間                | 500ms超                        |

### 4.2 ログ監査

- 重要操作（会員情報更新、注文処理等）は監査ログに記録  
- 監査ログは暗号化して保管、アクセス権限を限定

---

## 5. バックアップ・リストア

### 5.1 バックアップ方針

- DBは毎日深夜にフルバックアップを自動実行  
- 増分バックアップは1時間毎  
- バックアップデータは別リージョンの安全なストレージに保管

### 5.2 リストア手順概要

- バックアップデータからのリストアは24時間以内を目標  
- リストア手順書を整備し定期的にリストア演習を実施