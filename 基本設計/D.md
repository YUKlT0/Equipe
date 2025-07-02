## 5. データ設計

本節では、システムにおける主なエンティティ間の関係と、主要なデータの流れの概要を示す。

### 5.1. 概念データモデル（ER図）
<div class="mermaid">
erDiagram
    CUSTOMER ||--o{ ORDER : "places"
    ORDER ||--|{ ORDER_DETAIL : "has"
    ORDER ||--|{ PAYMENT : "has"
    ORDER_DETAIL }|--|| PRODUCT : "includes"
    PRODUCT }|--|| CATEGORY : "belongs to"
    PRODUCT ||--o{ PRODUCT_IMAGE : "has"
    ORDER ||--|| SHIPPING : "has"
    ADMIN ||--o{ ADMIN_LOG : "makes"

    CUSTOMER {
        string 会員ID PK
        string 氏名
        string メールアドレス
        string ハッシュパスワード
        string 住所
        datetime 登録日時
    }

    ADMIN {
        string 管理者ID PK
        string 氏名
        string メールアドレス
        string パスワード
        string 登録日時
    }

    ORDER {
        string 注文ID PK
        string 会員ID FK
        string 氏名
        string メールアドレス
        string 住所
        datetime 注文日時
        decimal 合計金額
        string 注文ステータス
    }

    ORDER_DETAIL {
        string 注文詳細ID PK
        string 注文ID FK
        string 商品ID FK
        int 数量
        decimal 単価
    }

    PRODUCT {
        string 商品ID PK
        string 商品名
        string 商品説明
        decimal 価格
        string カテゴリID FK
    }

    CATEGORY {
        string カテゴリID PK
        string カテゴリ名
    }

    PRODUCT_IMAGE {
        string 画像ID PK
        string 商品ID FK
        string 画像URL
        int 表示順
    }

    SHIPPING {
        string 配送ID PK
        string 注文ID FK
        string 配送先住所
        string 配送ステータス
    }

    ADMIN_LOG {
        string ログID PK
        string 管理者ID FK
        string 操作内容
        datetime 操作日時
    }

    PAYMENT {
        string 支払いID PK
        string 注文ID FK
        string 支払い方法
        string 支払い状況
    }
</div>

- **エンティティ**:  
  CUSTOMER(顧客), ADMIN(管理者), ORDER(注文), ORDER_DETAIL(注文詳細), PRODUCT(商品), PRODUCT_IMAGE(商品画像), CATEGORY(カテゴリ), SHIPPING(配送情報), ADMIN_LOG(管理者ログ), PAYMENT(決済情報)

- **リレーション**:  
  - 顧客は注文を行う (1対多、CUSTOMER -> ORDER)  
  - 注文は複数の注文詳細を持つ (1対多、ORDER -> ORDER_DETAIL)  
  - 注文詳細は商品を含む (多対1、ORDER_DETAIL -> PRODUCT)  
  - 商品はカテゴリに属する (多対1、PRODUCT -> CATEGORY)  
  - 商品は複数の画像を持つ (1対多、PRODUCT -> PRODUCT_IMAGE)  
  - 商品は在庫を持つ (1対1、PRODUCT -> INVENTORY)  
  - 注文は配送情報を持つ (1対1、ORDER -> SHIPPING_INFO)  
  - 配送情報は配送方法を利用する (多対1、SHIPPING_INFO -> SHIPPING_METHOD)  
  - 管理者は操作ログを複数持つ (1対多、ADMIN -> ADMIN_LOG)


### 5.2. 主要テーブル概要
- **CUSTOMER (顧客テーブル)**
    - ECサイトの会員情報を管理。
    - 会員IDを主キーとし、会員登録者のみ保持。
    - 氏名、メールアドレス、ハッシュ化パスワード、住所、登録日時を保持。
    - 会員登録が任意のため、注文時の氏名・住所等は注文テーブルにも保持し、配送情報と連携。

- **ADMIN (管理者テーブル)**
    - ECサイトの管理者情報を管理。
    - 管理者IDを主キーとして管理。
    - 氏名、メールアドレス、パスワード（ハッシュ化）、登録日時を保持。

- **ORDER (注文テーブル)**
    - 顧客の注文情報を管理。
    - 注文IDを主キーとし、会員IDは外部キーとして関連付ける（非会員はNULL許容）。
    - 注文日時、合計金額、注文ステータス、注文時の氏名・メールアドレス・住所を保持し、配送や請求の基礎情報とする。

- **ORDER_DETAIL (注文詳細テーブル)**
    - 各注文に含まれる商品ごとの詳細情報を管理。
    - 注文詳細IDを主キー、注文IDと商品IDを外部キーとして紐付け。
    - 数量、単価を保持し、複数商品の注文に対応。

- **PRODUCT (商品テーブル)**
    - ECサイトで販売する商品の情報を管理。
    - 商品IDを主キーとし、商品名、商品説明、価格、カテゴリID（外部キー）を保持。
    - 商品画像と関連付ける。

- **PRODUCT_IMAGE (商品画像テーブル)**
    - 商品に紐づく複数の画像情報を管理。
    - 画像IDを主キーとし、商品IDを外部キーとして紐付け。
    - 画像URL、表示順を保持。

- **CATEGORY (カテゴリテーブル)**
    - 商品の分類情報を管理。
    - カテゴリIDを主キーとし、カテゴリ名を保持。

- **SHIPPING (配送情報テーブル)**
    - 注文ごとの配送に関する情報を管理。
    - 配送IDを主キー、注文IDと配送方法IDを外部キーとして紐付け。
    - 配送先住所、配送ステータスを保持。
    - 送料は配送方法テーブルで管理し、注文合計金額に反映される。

- **ADMIN_LOG (管理者ログテーブル)**
    - 管理者による操作履歴を管理。
    - ログIDを主キー、管理者IDを外部キーとして紐付け。
    - 操作内容、操作日時を保持。

- **PAYMENT_INFO (支払い情報テーブル)**
    - 注文に紐づく支払い情報を管理（拡張機能）。
    - 支払いIDを主キー、注文IDを外部キーとして紐付け。
    - 支払い方法、支払い状況を保持。
    - 初期リリースでは振込・代引きを想定。

### 5.3　データフロー概要

#### 例：サイトアクセスから注文完了までのデータフロー（非会員）

##### データフロー（シーケンス図）

<div class="mermaid">
sequenceDiagram
    participant ユーザー
    participant 画面
    participant フロントエンド
    participant バックエンド
    participant DB

    ユーザー->>画面: ECサイトにアクセス
    画面->>フロントエンド: 商品一覧表示要求
    フロントエンド->>バックエンド: 商品情報取得リクエスト
    バックエンド->>DB: PRODUCT, CATEGORY, PRODUCT_IMAGE を検索
    DB-->>バックエンド: 商品情報返却
    バックエンド-->>フロントエンド: 商品情報返却
    フロントエンド-->>画面: 商品一覧を表示

    ユーザー->>画面: 商品を選択し詳細を見る
    画面->>フロントエンド: 商品詳細取得
    フロントエンド->>バックエンド: 商品IDを送信
    バックエンド->>DB: PRODUCT を検索
    DB-->>バックエンド: 商品詳細返却
    バックエンド-->>フロントエンド: 商品詳細返却
    フロントエンド-->>画面: 商品詳細表示

    ユーザー->>画面: 商品をカートに追加
    画面->>フロントエンド: カートに商品情報を保持

    ユーザー->>画面: 購入手続きに進む
    画面->>フロントエンド: ユーザー情報入力（氏名・住所・メールなど）
    フロントエンド->>バックエンド: 入力情報を送信

    バックエンド->>DB: SHIPPING_METHOD から送料取得
    DB-->>バックエンド: 送料情報

    バックエンド->>DB: ORDER を登録
    バックエンド->>DB: ORDER_DETAIL を商品ごとに登録
    バックエンド->>DB: SHIPPING_INFO を登録

    loop 商品ごとに在庫更新
        バックエンド->>DB: INVENTORY を検索
        DB-->>バックエンド: 現在の在庫数
        バックエンド->>DB: 在庫数を減算して更新
    end

    DB-->>バックエンド: 登録・更新成功
    バックエンド-->>フロントエンド: 注文完了情報返却
    フロントエンド-->>画面: 注文完了画面を表示
    画面-->>ユーザー: 注文番号・完了メッセージを表示
</div>

##### **1. サイトアクセス〜商品閲覧**

1. **画面（ユーザー）**  
   ユーザーがブラウザでECサイトにアクセス。トップページや商品一覧ページを開く。

2. **アプリケーション（Frontend）**  
   商品一覧を表示するためのリクエストをバックエンドへ送信。

3. **アプリケーション（Backend）**
   - `PRODUCT` テーブルから商品情報を取得。
   - 必要に応じて `CATEGORY`, `PRODUCT_IMAGE` をJOIN。
   - フロントエンドに商品情報を返却。

4. **画面（ユーザー）**  
   商品一覧が表示され、ユーザーは気になる商品をクリック。

5. **アプリケーション（Frontend） → Backend**
   - 選択された商品IDで `PRODUCT` の詳細を取得し、商品詳細ページを表示。

##### **2. カート操作〜注文確認画面**

1. **画面（ユーザー）**  
   商品詳細ページで数量を選び、「カートに追加」ボタンをクリック。

2. **アプリケーション（Frontend）**
   - 商品ID・数量をローカルセッションに保持。
   - 「カートを見る」ボタンでカート内容表示。

3. **ユーザーが「購入へ進む」ボタンをクリック**

4. **画面**  
   注文者情報入力フォームが表示される（非会員向け）。
   - 入力内容：氏名、メールアドレス、住所、配送方法。

##### **3. 注文確定処理**

1. **画面（ユーザー）**  
   内容を確認し、「注文を確定する」ボタンをクリック。

2. **アプリケーション（Backend）**
   以下の登録・更新処理を順に実施：

   - `ORDER` テーブルに注文基本情報を登録（非会員のため `会員ID` は NULL）。
   - `ORDER_DETAIL` に商品ID・数量・単価を登録。
   - `SHIPPING_INFO` に配送先住所・配送方法を登録。
   - `SHIPPING_METHOD` から送料を取得し、合計金額を算出。
   - `INVENTORY` の在庫数を商品ごとに減算して更新。

3. **データベース（DB）**
   - INSERT：`ORDER`, `ORDER_DETAIL`, `SHIPPING_INFO`
   - UPDATE：`INVENTORY`
   - SELECT：`PRODUCT`, `SHIPPING_METHOD`, `INVENTORY`

4. **アプリケーション（Backend）**
   注文結果をフロントエンドに返却。

5. **画面（ユーザー）**  
   注文完了画面を表示（注文番号・注文情報）。

#### 例：商品情報と在庫情報の新規登録（管理者）

##### データフロー（シーケンス図）
<div class="mermaid">
sequenceDiagram
    participant AdminUser as 管理者ユーザー
    participant Web as 管理者画面
    participant Backend as バックエンドAPI
    participant DB as データベース

    %% ログイン処理
    AdminUser->>Web: 専用URLにアクセスしログイン情報入力
    Web->>Backend: ログイン認証リクエスト（メールアドレス・パスワード）
    Backend->>DB: ADMINテーブルにて認証照合
    DB-->>Backend: 認証結果返却
    Backend-->>Web: ログイン成功レスポンス
    Web-->>AdminUser: 管理者ダッシュボード表示

    %% 商品情報登録
    AdminUser->>Web: 商品マスタ編集画面に遷移
    Web-->>AdminUser: 商品マスタ登録フォーム表示
    AdminUser->>Web: 商品情報入力・送信
    Web->>Backend: 商品登録リクエスト
    Backend->>DB: PRODUCTテーブルに商品データ追加
    DB-->>Backend: 登録成功レスポンス
    Backend-->>Web: 商品登録完了通知
    Web-->>AdminUser: 商品登録完了メッセージ表示

    %% 在庫情報登録
    AdminUser->>Web: ダッシュボードに戻る
    Web-->>AdminUser: ダッシュボード再表示
    AdminUser->>Web: 在庫管理画面に遷移
    Web-->>AdminUser: 在庫登録フォーム表示
    AdminUser->>Web: 在庫情報入力・送信
    Web->>Backend: 在庫登録リクエスト
    Backend->>DB: INVENTORYテーブルに在庫データ追加
    DB-->>Backend: 登録成功レスポンス
    Backend-->>Web: 在庫登録完了通知
    Web-->>AdminUser: 在庫登録完了メッセージ表示
</div>

1. **ログイン処理**
    - 管理者ユーザーが専用URLにアクセスし、メールアドレスとパスワードを入力してログイン。
    - バックエンドはADMINテーブルで認証処理を行い、成功時はダッシュボード画面を表示する。

2. **商品情報登録**
    - 管理者がダッシュボードから商品マスタ編集画面へ遷移。
    - 商品名・説明・価格・カテゴリなどを入力し、登録フォームを送信。
    - バックエンドはPRODUCTテーブルに新規商品情報を登録。
    - 登録完了メッセージが表示される。

3. **在庫情報登録**
    - 管理者が再度ダッシュボードに戻り、在庫管理画面に遷移。
    - 商品と在庫数を入力して在庫登録フォームを送信。
    - バックエンドはINVENTORYテーブルに該当商品の在庫データを登録。
    - 登録完了メッセージが表示される。



<!-- 拡張機能ありバージョン -->
### 5.4　おまけ
#### 拡張機能（レビュー・お気に入り・問い合わせ・支払い情報含む）を含む場合
<div class="mermaid">
erDiagram
    CUSTOMER ||--o{ ORDER : "places"
    ORDER ||--|{ ORDER_DETAIL : "has"
    ORDER_DETAIL }|--|| PRODUCT : "includes"
    PRODUCT }|--|| CATEGORY : "belongs to"
    PRODUCT ||--o{ PRODUCT_IMAGE : "has"
    PRODUCT ||--|| INVENTORY : "has"
    ORDER ||--|| SHIPPING_INFO : "has"
    SHIPPING_INFO }|--|| SHIPPING_METHOD : "uses"
    ADMIN ||--o{ ADMIN_LOG : "makes"

    CUSTOMER ||--o{ REVIEW : "writes"
    PRODUCT ||--o{ REVIEW : "has"

    CUSTOMER ||--o{ FAVORITE : "marks"
    PRODUCT ||--o{ FAVORITE : "is marked"

    CUSTOMER ||--o{ INQUIRY : "makes"

    ORDER ||--|| PAYMENT_INFO : "has"

    CUSTOMER {
        string 会員ID PK
        string 氏名
        string メールアドレス
        string ハッシュパスワード
        string 住所
        datetime 登録日時
    }

    ADMIN {
        string 管理者ID PK
        string 氏名
        string メールアドレス
        string パスワード
        string 登録日時
        string 権限情報
    }

    ORDER {
        string 注文ID PK
        string 会員ID FK
        string 氏名
        string メールアドレス
        string 住所
        datetime 注文日時
        decimal 合計金額
        string 注文ステータス
    }

    ORDER_DETAIL {
        string 注文詳細ID PK
        string 注文ID FK
        string 商品ID FK
        int 数量
        decimal 単価
    }

    PRODUCT {
        string 商品ID PK
        string 商品名
        string 商品説明
        decimal 価格
        string カテゴリID FK
    }

    CATEGORY {
        string カテゴリID PK
        string カテゴリ名
    }

    INVENTORY {
        string 在庫ID PK
        string 商品ID FK
        int 在庫数
    }

    PRODUCT_IMAGE {
        string 画像ID PK
        string 商品ID FK
        string 画像URL
        int 表示順
    }

    SHIPPING_INFO {
        string 配送ID PK
        string 注文ID FK
        string 配送方法ID FK
        string 配送先住所
        string 配送ステータス
        string 追跡番号
    }

    SHIPPING_METHOD {
        string 配送方法ID PK
        string 地域区分
        string 配送業者
        string 配送方法名
        decimal 送料
        int 所要日数
    }

    ADMIN_LOG {
        string ログID PK
        string 管理者ID FK
        string 操作内容
        datetime 操作日時
    }

    PAYMENT_INFO {
        string 支払いID PK
        string 注文ID FK
        string 支払い方法
        string 支払い状況
        string 取引ID
    }

    REVIEW {
        string レビューID PK
        string 会員ID FK
        string 商品ID FK
        int 評価
        string コメント
        datetime 投稿日時
    }

    FAVORITE {
        string お気に入りID PK
        string 会員ID FK
        string 商品ID FK
        datetime 登録日時
    }

    INQUIRY {
        string 問い合わせID PK
        string 会員ID FK
        string メールアドレス
        string 問い合わせカテゴリ
        string 問い合わせ内容
        string 対応ステータス
        datetime 問い合わせ日時
    }
</div>

- **エンティティ**:  
  CUSTOMER(顧客), ADMIN(管理者), ORDER(注文), ORDER_DETAIL(注文詳細), PRODUCT(商品), INVENTORY(在庫), PRODUCT_IMAGE(商品画像), CATEGORY(カテゴリ), SHIPPING_INFO(配送情報), SHIPPING_METHOD(配送方法), ADMIN_LOG(管理者ログ), REVIEW(レビュー), FAVORITE(お気に入り), INQUIRY(問い合わせ), PAYMENT_INFO(支払い情報)

- **リレーション**:  
  - 顧客は注文を行う (1対多、CUSTOMER -> ORDER)  
  - 注文は複数の注文詳細を持つ (1対多、ORDER -> ORDER_DETAIL)  
  - 注文詳細は商品を含む (多対1、ORDER_DETAIL -> PRODUCT)  
  - 商品はカテゴリに属する (多対1、PRODUCT -> CATEGORY)  
  - 商品は複数の画像を持つ (1対多、PRODUCT -> PRODUCT_IMAGE)  
  - 商品は在庫を持つ (1対1、PRODUCT -> INVENTORY)  
  - 注文は配送情報を持つ (1対1、ORDER -> SHIPPING_INFO)  
  - 配送情報は配送方法を利用する (多対1、SHIPPING_INFO -> SHIPPING_METHOD)  
  - 管理者は操作ログを複数持つ (1対多、ADMIN -> ADMIN_LOG)  
  - 顧客は複数のレビューを書く (1対多、CUSTOMER -> REVIEW)  
  - 商品は複数のレビューを持つ (1対多、PRODUCT -> REVIEW)  
  - 顧客は複数の商品をお気に入りに登録する (1対多、CUSTOMER -> FAVORITE)  
  - 商品は複数の顧客にお気に入り登録される (1対多、PRODUCT -> FAVORITE)  
  - 顧客は問い合わせを複数行う (1対多、CUSTOMER -> INQUIRY)  
  - 注文は支払い情報を持つ (1対1、ORDER -> PAYMENT_INFO)
  
  #### 主要テーブル概要

- **CUSTOMER (顧客テーブル)**
    - ECサイトの会員および非会員の顧客情報を管理。
    - 会員IDを主キーとし、会員登録者のみ保持。非会員はNULLまたは未登録の場合あり。
    - 氏名、メールアドレス、ハッシュ化パスワード、住所、登録日時を保持。
    - 会員登録が任意のため、注文時の氏名・住所等は注文テーブルにも保持し、配送情報と連携。

- **ADMIN (管理者テーブル)**
    - ECサイトの管理者情報を管理。
    - 管理者IDを主キーとし、権限情報で操作可能範囲を管理。
    - 氏名、メールアドレス、パスワード（ハッシュ化）、登録日時を保持。

- **ORDER (注文テーブル)**
    - 顧客の注文情報を管理。
    - 注文IDを主キーとし、会員IDは外部キーとして関連付ける（非会員はNULL許容）。
    - 注文日時、合計金額、注文ステータス、注文時の氏名・メールアドレス・住所を保持し、配送や請求の基礎情報とする。

- **ORDER_DETAIL (注文詳細テーブル)**
    - 各注文に含まれる商品ごとの詳細情報を管理。
    - 注文詳細IDを主キー、注文IDと商品IDを外部キーとして紐付け。
    - 数量、単価を保持し、複数商品の注文に対応。

- **PRODUCT (商品テーブル)**
    - ECサイトで販売する商品の情報を管理。
    - 商品IDを主キーとし、商品名、商品説明、価格、カテゴリID（外部キー）を保持。
    - 商品画像や在庫情報と関連付ける。

- **INVENTORY (在庫テーブル)**
    - 商品ごとの在庫数を管理。
    - 在庫IDを主キーとし、商品IDを外部キーとして紐付け。
    - 在庫数を保持し、販売可能数の管理に利用。

- **PRODUCT_IMAGE (商品画像テーブル)**
    - 商品に紐づく複数の画像情報を管理。
    - 画像IDを主キーとし、商品IDを外部キーとして紐付け。
    - 画像URL、表示順を保持。

- **CATEGORY (カテゴリテーブル)**
    - 商品の分類情報を管理。
    - カテゴリIDを主キーとし、カテゴリ名を保持。

- **SHIPPING_INFO (配送情報テーブル)**
    - 注文ごとの配送に関する情報を管理。
    - 配送IDを主キー、注文IDと配送方法IDを外部キーとして紐付け。
    - 配送先住所、配送ステータス、追跡番号を保持。
    - 送料は配送方法テーブルで管理し、注文合計金額に反映される。

- **SHIPPING_METHOD (配送方法テーブル)**
    - 配送方法ごとの詳細（地域区分、配送業者、送料、所要日数）を管理。
    - 配送方法IDを主キーとする。

- **ADMIN_LOG (管理者ログテーブル)**
    - 管理者による操作履歴を管理。
    - ログIDを主キー、管理者IDを外部キーとして紐付け。
    - 操作内容、操作日時を保持。

- **PAYMENT_INFO (支払い情報テーブル)**
    - 注文に紐づく支払い情報を管理。
    - 支払いIDを主キー、注文IDを外部キーとして紐付け。
    - 支払い方法、支払い状況、取引IDを保持。
    - 初期リリースでは振込・代引きを想定し、後にクレジットカード等追加可能。

- **REVIEW (レビュー・評価テーブル)**
    - 顧客による商品のレビュー情報を管理。
    - レビューIDを主キー、会員IDと商品IDを外部キーとして紐付け。
    - 評価（星評価など）、コメント、投稿日時を保持。

- **FAVORITE (お気に入りテーブル)**
    - 顧客がお気に入り登録した商品情報を管理。
    - お気に入りIDを主キー、会員IDと商品IDを外部キーとして紐付け。
    - 登録日時を保持。

- **INQUIRY (問い合わせテーブル)**
    - 顧客からの問い合わせ情報を管理。
    - 問い合わせIDを主キーとし、会員ID（会員の場合のみ）、メールアドレス、問い合わせカテゴリ、問い合わせ内容、対応ステータス、問い合わせ日時を保持。

#### 例：会員による商品のお気に入り登録と購入、注文後の問い合わせと管理者による対応

##### データフロー（シーケンス図）
<div class="mermaid">
sequenceDiagram
    participant User as 会員ユーザー
    participant Web as 画面
    participant Backend as バックエンドAPI
    participant DB as データベース
    participant Admin as 管理者

    %% 顧客がログイン
    User->>Web: ログイン画面でメール・パスワード入力
    Web->>Backend: 認証リクエスト
    Backend->>DB: CUSTOMER テーブルから認証情報照合
    DB-->>Backend: 認証結果
    Backend-->>Web: 認証成功（セッション開始）
    Web-->>User: マイページ or 商品一覧に遷移

    %% お気に入り登録
    User->>Web: 商品をお気に入りに登録
    Web->>Backend: お気に入り登録リクエスト
    Backend->>DB: FAVORITEテーブルに登録
    DB-->>Backend: 登録完了
    Backend-->>Web: 登録成功レスポンス
    Web-->>User: 登録完了メッセージ表示

    %% 注文処理
    User->>Web: 注文を作成・送信
    Web->>Backend: 注文情報送信
    Backend->>DB: ORDERテーブルに注文登録
    Backend->>DB: ORDER_DETAILテーブルに注文詳細登録
    Backend->>DB: INVENTORYテーブルの在庫数更新
    Backend->>DB: PAYMENT_INFOテーブルに支払い情報登録
    DB-->>Backend: 登録完了
    Backend-->>Web: 注文完了レスポンス
    Web-->>User: 注文完了画面表示

    %% 顧客が問い合わせ送信
    User->>Web: 問い合わせフォームに入力・送信
    Web->>Backend: 問い合わせ送信リクエスト
    Backend->>DB: INQUIRYテーブルに新規登録
    DB-->>Backend: 登録成功
    Backend-->>Web: 問い合わせ受付完了
    Web-->>User: 問い合わせ送信完了画面表示
    Backend-->>User: 問い合わせ完了通知メール送信


    %% 管理者が問い合わせ一覧確認
    Admin->>Web: 管理者ダッシュボードから問い合わせ一覧を開く
    Web->>Backend: 問い合わせ一覧取得リクエスト
    Backend->>DB: INQUIRYテーブルから一覧取得
    DB-->>Backend: 問い合わせデータ返却
    Backend-->>Web: 問い合わせ一覧レスポンス
    Web-->>Admin: 問い合わせ一覧表示

    %% 管理者が問い合わせ内容を確認
    Admin->>Web: 問い合わせ詳細をクリック
    Web->>Backend: 問い合わせ詳細取得
    Backend->>DB: INQUIRYテーブルから詳細取得
    DB-->>Backend: 詳細データ返却
    Backend-->>Web: 詳細レスポンス
    Web-->>Admin: 問い合わせ内容表示

    %% 管理者が返答
    Admin->>Web: 回答を入力・送信
    Web->>Backend: 回答登録リクエスト
    Backend->>DB: INQUIRYテーブルに返答内容・ステータス更新
    DB-->>Backend: 更新完了
    Backend-->>Web: 完了通知
    Web-->>Admin: 回答完了メッセージ表示

    %% 顧客に返答通知
    Backend-->>User: 返答内容通知メール送信
</div>

1. **お気に入り登録**
    - 会員ユーザーは気になる商品をお気に入りに登録する。
    - WebサイトからバックエンドAPIへ登録リクエストが送信され、FAVORITEテーブルに記録される。
    - 登録完了後、ユーザーに完了メッセージが表示される。

2. **注文処理**
    - 会員は購入したい商品を注文画面で選択し注文を確定する。
    - バックエンドはORDER、ORDER_DETAIL、INVENTORY（在庫数更新）、PAYMENT_INFO（支払い情報）テーブルに対して処理を行う。
    - 注文完了レスポンスが返され、注文完了画面を表示する。

3. **問い合わせ**
    - 会員は商品や注文に関する問い合わせをフォームから送信する。
    - バックエンドはINQUIRYテーブルに問い合わせ内容を登録し、受付完了をユーザーに通知する。

4. **管理者の問い合わせ対応**
    - 管理者は問い合わせ一覧を取得し、対応状況を確認・更新する。
    - 対応状況はINQUIRYテーブルに反映され、更新完了レスポンスが管理者に返される。