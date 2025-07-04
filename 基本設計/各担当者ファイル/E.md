E
## 6. 非機能要件 対応方針

### 6.1. 性能

- **レスポンスタイム**: 商品一覧、商品詳細、カート操作など、主要画面において通常時（同時接続100ユーザー程度）での応答時間を3秒以内とする。
- **同時接続ユーザー数**: 想定ピーク時の同時接続は最大100ユーザー程度。
- **対応方針**:
    - 商品一覧など大量データが表示される画面では無限スクロール方式を採用。
    - データベースには適切なインデックスを設定し、クエリの最適化を行う。
    - アプリケーション・DBサーバーのスペックは負荷試験結果に応じて選定。クラウド環境のスケーリングも視野に入れる。
    - 必要に応じてキャッシュ導入を検討（例：カテゴリ一覧など更新頻度が低いもの）。
  
### 6.2. セキュリティ

- **認証・認可**:
  - 会員登録は任意。登録時には名前、住所、メールアドレス、パスワードを入力し、サーバー側で安全にハッシュ化して保存。
  - ログイン／ログアウト機能を提供し、マイページで会員情報の編集が可能。
  - セッション管理によりログイン状態を維持し、一定時間操作がない場合は自動ログアウト。
  - ユーザーと管理者のロールを分離し、管理機能（商品管理）は管理者のみ利用可。
  - **二要素認証について**  
  本システムでは現時点で二要素認証は実装しないが、今後のセキュリティ要件の変化やリスク評価に応じて、導入の可能性がある。  
  将来的に導入する場合は別途要件定義および設計検討を行うものとする。
- **通信の暗号化**:
  - Webアプリケーションは常時HTTPS（TLS1.2以上）を利用。
- **入力値の検証**:
  - すべてのユーザー入力についてサーバー側で形式・文字種・長さなどのバリデーションを実施。
  - SQLインジェクション対策としてプリペアドステートメント／エスケープ処理を使用。
  - クロスサイトスクリプティング（XSS）対策として、入力時／出力時にエスケープ処理を実施。
- **アクセス制御**:
  - URL直アクセスによる不正操作防止のため、サーバー側でのロール／権限チェックを実装。
- **その他**:
  - セッションID固定化の防止およびセッションタイムアウトの設定。
  - 使用ライブラリのセキュリティ脆弱性に対する定期的なチェックとアップデート。

### 6.3. 可用性

- **目標稼働率**:  
    - 月間稼働率99.5%以上を目指す。  
  ※ 99.5%の稼働率は約3.65日/月のダウンタイムに相当し、ECサイトとしてはやや低めの水準である。  
  可能であれば99.9%以上（約43分/月）を目指すことが望ましいが、  
  コスト制約や初期フェーズのリスク許容度を踏まえ、本数値を設定している。  
  将来的なサービス拡大に伴い、稼働率向上の検討を行う。  
- **対応方針**:  
    - 初期は単一構成としつつも、障害時のバックアップ復旧手順を整備。  
    - 必要に応じてWeb／DBの冗長化やマネージドサービスの高可用性オプションを検討。  
    - データバックアップの取得とリストア手順を運用設計に含める。

### 6.4. その他（保守性、運用性、拡張性）

- **保守性**:
    - フレームワークの標準的な設計パターンに従い責務分離を実施。
    - 設定値は環境変数や設定ファイルで管理し、ハードコードを避ける。
- **運用性**:
    - システム稼働状況や障害発生をログで監視可能とする。
    - 定期メンテナンス（アップデート／ライブラリ更新）計画を策定。
- **拡張性**:
    - 将来的な機能追加や外部連携に備え、疎結合な設計とする。
    - 商品カテゴリは現時点では単一階層とするが、将来的に多階層カテゴリへの拡張を見据えたテーブル構成とする。
- **マイページ**:  
  会員向けの専用画面を設置し、現時点では会員情報（名前、住所、メールアドレス、パスワード）の閲覧および編集機能を提供する。  
  将来的な機能拡張（注文履歴の確認やポイント管理など）を見据え、拡張しやすい設計とする。  

## 7. 運用・保守設計の概要

### 7.1. ログ設計方針

- **出力ログ**:
    - **アクセスログ**: リクエスト元IP、メソッド、パス、ステータスコード、処理時間など。
    - **アプリケーションログ**: 起動／停止、主要処理開始・終了、エラー（スタックトレース含む）。
    - **操作ログ（監査ログ）**: ユーザーのログイン／ログアウト、商品登録・削除・編集など。
- **ログレベル**:  
    - DEBUG, INFO, WARN, ERROR を定義し、本番環境ではINFO以上を出力。  
    - 【懸念事項】本番環境で詳細なDEBUGログを常時出力すると、システム負荷やパフォーマンス低下の可能性があるため、運用状況に応じてログレベルを切り替えることを推奨。
- **フォーマット**:  
    - タイムスタンプ、ログレベル、メッセージ、ユーザーID、リクエストID等を含む。
- **保管期間**:  
    - アクセスログ／アプリケーションログ: 最低1ヶ月。  
    - 操作ログ: 最低1年間。  
    - 【懸念事項】長期間のログ保存はストレージ容量の増加や管理コスト増大の懸念があるため、保存期間の見直しや定期的なログ削除・アーカイブ運用を検討することが望ましい。
- **ログ対象範囲**:  
    - 重要な操作や管理者操作を中心にログ記録を行う。  
    - 【懸念事項】すべてのユーザー操作を詳細に記録すると、プライバシー保護の観点やログ管理負担が増すため、ログ対象の範囲は必要最小限に絞ることを推奨。

### 7.2. 監視設計方針

- **監視項目**:
  - CPU使用率
  - メモリ使用率
  - ネットワークトラフィック
  - ディスク使用率（例: 80%以上でアラート）
  - アプリケーションログのエラー検知
  - データベースの稼働状況
- **監視方法**:
    - Zabbix、Nagios、Prometheus+Grafana 等の監視ツールを導入。
    - 閾値超過や異常発生時には、管理者にメールやチャットツール等で通知。

- #### 7.2.1 閾値の定義（主要項目）

| 項目               | 閾値                        | 備考                                         |
|--------------------|-----------------------------|----------------------------------------------|
| CPU使用率          | 80%以上（5分間継続）        | 短時間のバーストは許容                      |
| メモリ使用率       | 90%以上                     | スワップ発生の前段階で通知                  |
| ディスク使用率     | 85%以上                     | ログやファイルアップロード増加に備えて監視 |
| HTTP応答時間       | 3秒超                        | 商品一覧・詳細ページなど主要画面が対象     |
| エラーログ出力数   | 10件/分以上                 | ERRORレベル以上のログを対象                 |
| HTTP 5xxエラー率   | 5%以上（直近1分平均）       | 500番台エラーの比率                          |
| DB接続失敗回数     | 3回以上/分                  | 瞬間的な断絶も検知                          |
| サーバー再起動検出 | 発生時に即時通知             | 想定外のクラッシュなどを検知                |

- **閾値の調整**:  
  上記のエラーログ出力数（10件/分以上）およびHTTP 5xxエラー率（5%以上）の閾値は、  
  初期設定値として設定しているが、運用開始後の実際のログ状況やシステム負荷を踏まえて柔軟に見直し・調整を行うことを想定する。

### 7.3. バックアップ・リカバリ方針

- **バックアップ対象**: DB全体、アップロードファイル、設定ファイルなど。
- **方式／頻度**:
    - 日次でフルバックアップ、必要に応じてトランザクションログのバックアップ。
    - 取得タイミングは深夜帯を想定。
- **保管場所／期間**:
    - バックアップは本番とは別環境（クラウドストレージ等）に保存。
    - 日次：直近7日、週次：4週分、月次：6ヶ月分を保持。
- **復旧手順**:
    - 復旧手順は文書化し、定期的にリストアテストを実施。
    - RTO（目標復旧時間）および RPO（目標復旧時点）を定める。

## 8. 制約事項・前提条件

### 8.1. 使用技術スタック

- **言語**: Java 17  
- **フレームワーク**: Spring Boot 3.x  
- **データベース**: MySQL 8.0  
- **フロントエンド**: HTML5, CSS3, JavaScript(ES6)  
- **Webサーバー**: Nginx  
- **その他ライブラリ**: Spring Boot標準のスターターやSpring Data JPA等を中心に選定予定    

### 8.2. インフラ環境

- **サーバーOS**: Ubuntu 22.04 LTS（予定）  
- **実行環境**: Java対応のアプリケーションサーバー（例：Tomcat、WildFlyなどを検討中）  
- **稼働場所**: クラウド環境（AWS EC2/RDSなどを想定）  
- **ネットワーク**: インターネット経由でのアクセスを前提とする  
  - 会社内LANや社内限定VPN環境ではなく、外部ユーザーもアクセス可能な環境を想定  
  - セキュリティ対策として以下を実施予定  
    - HTTPS強制（TLS1.2以上の利用）  
    - AWS WAFを利用したWebアプリケーションファイアウォール設置  
    - CloudflareによるDDoS攻撃対策  
    - 認証・認可の強化（ロール分離、アクセス制御の厳格化）  

### 8.3. 開発・運用ルール

- **バージョン管理**: Git（GitHub、GitLab想定）。ブランチ戦略はGit-Flowベースにて運用  
- **コーディング規約**: Java標準コーディング規約に準拠。静的解析ツール（Checkstyle、SpotBugs等）によるコード品質管理を行う  
- **テスト**: JUnitによる単体テストおよび統合テストを実施  
- **デプロイ**: CI/CD環境による自動デプロイを検討・構築予定  

### 8.4. スコープ外

- 外部サービスとのAPI連携（例：決済、配送業者連携）  
- モバイルアプリの開発  
- 詳細なテスト仕様書および運用マニュアルの作成  

### 8.5. その他前提条件

- 会員登録は任意とし、登録時にはユーザーID、名前、住所、メールアドレス、パスワードを入力。パスワードはサーバー側で安全にハッシュ化して保存  
- ユーザーと管理者のロールを明確に分離し、管理機能（商品管理）は管理者のみ利用可能とする  
- 商品一覧画面は無限スクロール方式を採用し、ユーザビリティを向上させる  
- 商品価格は商品ごとに設定可能とし、送料は全国一律700円とする  
- 本システムは受注販売形式を採用し、配送フローが開始される設計とする  
- 初期商品データはCSVファイルによってインポートする  
- 開発・運用時のログ出力は膨大になる可能性があり、将来的にはログ保管コストやサーバーの稼働状況に応じて適切な保守・メンテナンス計画を策定予定  


## 9. （付録）用語集・略語リスト

## 用語集

| 用語／略語           | 定義・説明 |
|----------------------|------------|
| **ECサイト**         | Electronic Commerce（電子商取引）サイトの略。インターネット上で商品やサービスを販売するウェブサイト。 |
| **カート機能**       | ユーザーが購入したい商品を一時的に保存し、まとめて注文できる機能。 |
| **商品詳細ページ**   | 商品の写真、説明、価格、素材、納期情報などの情報を表示するページ。 |
| **スマートフォン対応** | スマートフォンなどのモバイル端末でも快適に閲覧・操作できるようにする対応。 |
| **会員登録機能**     | ユーザーが個人情報を登録し、住所入力の簡略化などの利便性を得られる機能。 |
| **カテゴリ分類**     | 商品を種類ごとに分類し、ユーザーが探しやすくする仕組み。 |
| **注文機能**         | ユーザーが商品を購入するための手続きを行う機能。 |
| **レビュー機能**     | ユーザーが購入した商品に対して評価やコメントを投稿できる機能。 |
| **決済機能**         | クレジットカードや電子マネーなどを使ってオンラインで代金を支払う機能。 |
| **無限スクロール**   | ページの下部までスクロールすると自動的に次のコンテンツが読み込まれる仕組み。ユーザーがページ切り替え操作をする必要がなく、連続的に閲覧できる。 |
| **API**              | Application Programming Interface。アプリケーションが他のソフトウェア機能やサービスを呼び出すためのインターフェース。 |
| **CSV**              | Comma-Separated Values。カンマ区切りのテキスト形式で、表形式データを保存・交換する際によく用いられる。 |
| **DB**               | Database（データベース）。本システムでは MySQL を指し、顧客情報や商品・注文などを管理。 |
| **ER図**             | Entity-Relationship Diagram。システム内のエンティティ（表）とその関係を図示するモデル。 |
| **HTTP / HTTPS**     | Hypertext Transfer Protocol。HTTPSはTLSで暗号化されたHTTP通信。API 呼び出しや画面表示に使用。 |
| **JSON**             | JavaScript Object Notation。軽量なデータ交換フォーマット。REST API の入出力によく使われる。 |
| **Spring Boot**      | Java製のWebアプリケーションフレームワーク。本システムのバックエンド開発に使用。 |
| **RDBMS**            | Relational Database Management System。関係データベース管理システム。MySQL 8.0 を利用。 |
| **REST API**         | Representational State Transfer に基づく API。HTTP メソッドで操作を行う。 |
| **SSO**              | Single Sign-On。一度のログインで複数の関連サービスにアクセス可能とする仕組み。 |
| **UI / UX**          | User Interface / User Experience。ユーザー操作画面の設計と利用体験全体の品質を示す。 |
| **バリデーション**   | 入力値の妥当性チェック。形式・必須・文字数などを確認し、不正入力を防ぐ。 |
| **マイページ**       | 会員ユーザーが自分の登録情報を確認・変更できる専用画面。 |
| **MySQL**            | オープンソースの関係型データベース管理システム。バージョン 8.0 を使用。 |
| **Nginx**            | Web および PHP-FPM 用のリバースプロキシ／HTTP サーバー。 |
| **PHP-FPM**          | PHP FastCGI Process Manager。PHP アプリケーションの実行環境。 |
| **リレーショナルDB**| テーブル間のリレーション（関係）を基本とした、関係モデル形式のデータベース。 |


## 10. 承認

本基本設計書の内容を、以下の関係者が承認する。
後続フェーズでの設計変更が発生した場合、変更管理のプロセスに従い本書を改訂し、バージョン管理を行う。

| 承認者        | 所属・役職          | 日付         | 印 |
| :------------ | :------------------ | :----------- | :- |
|田中　太郎  (甲)   | 株式会社〇〇の事業企画部 部長  | 202X/XX/XX   |    |
|山田　一郎  (乙)   | ベンダー　プロジェクトマネージャー    | 202X/XX/XX   |    |
|佐藤　花子 (乙)   | ベンダー　システムエンジニア | 202X/XX/XX   |    |
