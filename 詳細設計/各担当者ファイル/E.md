
## 10.インフラ・非機能要件詳細設計書

---

### 10.1. 開発環境・実行環境仕様

| 項目               | 詳細                           |
|------------------|------------------------------|
| OS               | Ubuntu 22.04 LTS              |
| Javaバージョン     | OpenJDK 17                   |
| フレームワーク      | Spring Boot 3.0              |
| Webサーバ          | Nginx 1.24                   |
| アプリケーションサーバ | Spring Boot内蔵サーバ           |
| DB               | MySQL 8.0                    |
| DB接続ドライバ      | MySQL Connector/J 8.x        |
| メッセージキュー     | RabbitMQ 3.11                 |
| キャッシュ          | Redis 7.0                    |

---

### 10.2. セキュリティ設計

#### 10.2.1 認証・認可

- Spring Security を利用しセッション管理（Cookieベース）による認証を実装  
- トークンはセッションIDとして管理、セッション有効期限は30分（基本設計書に合わせる）  
- 管理者APIはロールベースのアクセス制御（RBAC）で制限  
- HTTPS通信必須（証明書はLet’s Encryptを使用）  
- パスワードはbcryptでハッシュ化保存

#### 10.2.2 入力値検証

- フロントエンド・サーバーサイド両方でバリデーション実施  
- SQLインジェクション対策としてプリペアドステートメント利用  
- クロスサイトスクリプティング（XSS）対策としてHTMLエスケープを徹底  
- クロスサイトリクエストフォージェリ（CSRF）対策として、トークン方式を導入（特に管理画面で強化）

#### 10.2.3 ログ管理

- ログフォーマットはJSON形式で統一  
- アクセスログ、エラーログ、監査ログを分離管理  
- ログレベルはINFOを基本とし、必要に応じてDEBUG切り替え可能  
- ログはファイル保存を基本とし、必要に応じてFluentd等のログ転送基盤を検討

---

### 10.3. 性能・可用性設計

#### 10.3.1 性能要件

- APIレスポンスタイムは**通常画面の画面遷移・表示で3秒以内**を目標  
- 同時接続数はピーク時で**最大100ユーザー**を想定

#### 10.3.2 負荷対策

- DBに対してはクエリ最適化とインデックス設計を徹底  
- Redisキャッシュを導入し、商品一覧やカテゴリ情報の読み取り負荷を軽減  
- 初期構成は単一構成、将来的な水平スケーリングを検討

#### 10.3.3 可用性

- 主要コンポーネントは基本的に単一構成。将来的に冗長構成を検討  
- アプリケーションサーバは単一台構成を想定（ロードバランシングは将来的検討）  
- 障害検知はZabbixやNagiosで監視し、アラートをメール・Slackに通知

---

### 10.4. 監視・運用設計

#### 10.4.1 監視項目

| 項目               | 監視内容                          | アラート条件                     |
|------------------|-------------------------------|-------------------------------|
| サーバCPU負荷      | CPU使用率                      | 80%以上が5分以上継続             |
| メモリ使用率        | メモリ使用率                   | 80%以上が5分以上継続             |
| ディスク容量        | 空き容量                       | 10%未満                        |
| DB接続数           | 現在の接続数                   | 80%以上の閾値超過               |
| アプリケーションエラー数 | 5xxエラーの発生数              | 5分間で10件以上                  |
| レスポンスタイム    | API平均応答時間                | 3秒超（通常画面の目標値に合わせ） |

#### 10.4.2 ログ監査

- 重要操作（会員情報更新、注文処理等）は監査ログに記録  
- 監査ログは暗号化して保管、アクセス権限を限定

---

### 10.5. バックアップ・リストア

#### 10.5.1 バックアップ方針

- DBは毎日深夜にフルバックアップを自動実行  
- 増分バックアップは週次・月次で実施（基本設計書の運用基準に準拠）  
- バックアップデータは別リージョンの安全なストレージに保管

#### 10.5.2 リストア手順概要

- バックアップデータからのリストアは72時間以内を目標（基本設計書ベース）  
- リストア手順書を整備し定期的にリストア演習を実施