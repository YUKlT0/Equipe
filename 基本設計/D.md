## 5. データ設計

本節では、システムにおける主なエンティティ間の関係と、主要なデータの流れの概要を示す。

### 5.1. 概念データモデル（ER図）
<div class="mermaid">
erDiagram
    CUSTOMER ||--o{ ORDER : "places"
    ORDER ||--|{ ORDER_DETAIL : "has"
    ORDER_DETAIL }|--|| PRODUCT : "includes"
    PRODUCT }|--|| CATEGORY : "belongs to"
    PRODUCT ||--o{ PRODUCT_IMAGE : "has"

    CUSTOMER {
        string 会員ID PK
        string 氏名
        string メールアドレス
        string ハッシュパスワード
        string 住所
        datetime 登録日時
        boolean メンバーシップ
    }

    ADMIN {
        string 管理者ID PK
        string 氏名
        string メールアドレス
        string ハッシュパスワード
        string 登録日時
    }

    ORDER {
        string 注文ID PK
        string 会員ID FK
        string 購入者氏名
        string 購入者メールアドレス
        string 購入者住所
        datetime 注文日時
        string 配送先氏名
        string 配送先住所
        decimal 送料
        decimal 合計金額
        string 決済方法
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

</div>

- **エンティティ**
  - CUSTOMER(顧客)
  - ADMIN(管理者)
  - ORDER(注文)
  - ORDER_DETAIL(注文詳細)
  - PRODUCT(商品)
  - PRODUCT_IMAGE(商品画像)
  - CATEGORY(カテゴリ)

- **リレーション**
  - 顧客は注文を行う (1対多、CUSTOMER -> ORDER)  
  - 注文は複数の注文詳細を持つ (1対多、ORDER -> ORDER_DETAIL)  
  - 注文詳細は商品を含む (多対1、ORDER_DETAIL -> PRODUCT)  
  - 商品はカテゴリに属する (多対1、PRODUCT -> CATEGORY)  
  - 商品は複数の画像を持つ (1対多、PRODUCT -> PRODUCT_IMAGE)  


### 5.2. 主要テーブル概要
- **CUSTOMER (顧客テーブル)**
    - ECサイトの会員情報を管理。
    - 会員IDを主キーとし、顧客全員の情報を保持。
    - 氏名、メールアドレス、ハッシュ化パスワード、住所、登録日時、メンバーシップを保持。
    - 非会員の場合は、注文時に入力した購入者情報を注文時にパスワードはNULL、メンバーシップはfalseとして保持。
    - 会員登録が任意のため、注文時の購入者氏名・購入者住所は注文テーブルにも保持。

- **ADMIN (管理者テーブル)**
    - ECサイトの管理者情報を管理。
    - 管理者IDを主キーとして管理。
    - 氏名、メールアドレス、ハッシュ化パスワード、登録日時を保持。

- **ORDER (注文テーブル)**
    - 顧客の注文情報を管理。
    - 注文IDを主キーとし、会員IDは外部キーとして関連付ける。
    - 注文日時、送料、合計金額、決済方法、注文ステータス、購入者氏名、購入者メールアドレス、購入者住所、配送先氏名、配送先住所を保持し配送や請求の基礎情報とする。

- **ORDER_DETAIL (注文詳細テーブル)**
    - 各注文に含まれる商品ごとの詳細情報を管理。
    - 注文詳細IDを主キー、注文IDと商品IDを外部キーとして紐付け。
    - 数量、単価を保持し、複数商品の注文に対応。

- **PRODUCT (商品テーブル)**
    - ECサイトで販売する商品の情報を管理。
    - 商品IDを主キーとし、商品名、商品説明、価格を保持、外部キーとしてカテゴリIDを紐づけ。
    - 商品画像と関連付ける。

- **PRODUCT_IMAGE (商品画像テーブル)**
    - 商品に紐づく複数の画像情報を管理。
    - 画像IDを主キーとし、商品IDを外部キーとして紐付け。
    - 画像URL、表示順を保持。

- **CATEGORY (カテゴリテーブル)**
    - 商品の分類情報を管理。
    - カテゴリIDを主キーとし、カテゴリ名を保持。

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
    画面->>フロントエンド: ユーザー情報入力（氏名・住所・メール・決済情報）
    フロントエンド->>バックエンド: 入力情報を送信

    バックエンド->>DB: ORDER を登録
    バックエンド->>DB: ORDER_DETAIL を商品ごとに登録

    DB-->>バックエンド: 登録・更新成功
    バックエンド-->>フロントエンド: 注文完了情報返却
    フロントエンド-->>画面: 注文完了画面を表示
</div>

##### **1. サイトアクセス〜商品閲覧**

1. **画面（SC0001）**  
   ユーザーがブラウザでECサイトにアクセス、ログイン画面が表示されゲストを選択。

2. **アプリケーション（Frontend）**  
   商品一覧を表示するためのリクエストをバックエンドへ送信。

3. **アプリケーション（Backend）**
   `PRODUCT` テーブルから商品情報を取得。
   必要に応じて `CATEGORY`, `PRODUCT_IMAGE` をJOIN。
   フロントエンドに商品情報を返却。

4. **画面（SC0110）**  
   商品一覧が表示され、ユーザーが商品をクリック。

5. **アプリケーション（Frontend） → Backend**
   選択された商品IDで `PRODUCT` の詳細を取得し、商品詳細ページを表示。

##### **2. カート操作〜注文確認画面**

1. **画面（SC0120）**  
   商品詳細ページで数量を選び、「カートに追加」ボタンをクリック。

2. **アプリケーション（Frontend）**
   商品ID・数量をローカルセッションに保持。
  「カートを見る」ボタンでカート内容表示。

3. **ユーザーが「購入へ進む」ボタンをクリック**

4. **画面（SC0140）**  
   注文者情報入力フォームが表示され、ユーザーが氏名、メールアドレス、住所、配送方法、決済情報を入力。

##### **3. 注文確定処理**

1. **画面（SC0150）**  
   内容を確認し、「注文を確定する」ボタンをクリック。

2. **アプリケーション（Backend）**
   `CUSTOMER`テーブルに氏名、メールアドレス、住所、登録日時、メンバーシップ（false）を登録。
   `ORDER` テーブルに注文基本情報を登録。
   `ORDER_DETAIL` テーブルに商品ID・数量・単価を登録。

3. **データベース（DB）**
   `ORDER`, `ORDER_DETAIL`に情報を登録。

4. **アプリケーション（Backend）**
   注文結果をフロントエンドに返却。

5. **画面（SC0160）**  
   注文完了画面を表示。

   

#### 例：サイトアクセスからログイン、注文完了までのデータフロー（会員）
<div class="mermaid">
sequenceDiagram
    participant ユーザー
    participant 画面
    participant フロントエンド
    participant バックエンド
    participant DB

    ユーザー->>画面: ECサイトにアクセス
    画面->>ユーザー: ログイン画面表示
    ユーザー->>画面: ログイン情報入力・送信
    画面->>フロントエンド: ログイン情報送信
    フロントエンド->>バックエンド: ログイン認証リクエスト
    バックエンド->>DB: CUSTOMERテーブルで認証
    DB-->>バックエンド: 認証結果返却
    バックエンド-->>フロントエンド: 認証結果返却
    フロントエンド-->>画面: ログイン結果表示（成功なら商品一覧へ）

    ユーザー->>画面: 商品一覧閲覧要求
    画面->>フロントエンド: 商品一覧表示要求
    フロントエンド->>バックエンド: 商品情報取得リクエスト
    バックエンド->>DB: PRODUCT, CATEGORY, PRODUCT_IMAGE を検索
    DB-->>バックエンド: 商品情報返却
    バックエンド-->>フロントエンド: 商品情報返却
    フロントエンド-->>画面: 商品一覧表示

    ユーザー->>画面: 商品を選択し詳細を見る
    画面->>フロントエンド: 商品詳細取得要求
    フロントエンド->>バックエンド: 商品ID送信
    バックエンド->>DB: PRODUCTを検索
    DB-->>バックエンド: 商品詳細返却
    バックエンド-->>フロントエンド: 商品詳細返却
    フロントエンド-->>画面: 商品詳細表示

    ユーザー->>画面: 商品をカートに追加
    画面->>フロントエンド: カートに商品情報保持

    ユーザー->>画面: 購入手続きに進む
    フロントエンド->>バックエンド: 会員住所情報取得リクエスト
    バックエンド->>DB: CUSTOMERテーブルから住所情報を取得
    DB-->>バックエンド: 住所情報返却
    バックエンド-->>フロントエンド: 住所情報返却
    フロントエンド-->>画面: 住所選択フォームに表示
    画面->>フロントエンド: ユーザー情報入力
    フロントエンド->>バックエンド: 注文情報送信

    バックエンド->>DB: ORDERを登録（会員ID含む）
    バックエンド->>DB: ORDER_DETAILを商品ごとに登録
    DB-->>バックエンド: 登録成功通知
    バックエンド-->>フロントエンド: 注文完了情報返却
    フロントエンド-->>画面: 注文完了画面表示
</div>

##### **1. サイトアクセス～ログイン**

1. **画面（SC0001）**  
   ユーザーがブラウザでECサイトにアクセスし、ログイン画面を表示。

1. **画面（SC0001）**  
   ユーザーがログイン画面でメールアドレスとパスワードを入力し送信。

2. **アプリケーション（Backend）**  
   送信された情報をもとにDBの会員情報を照合し、認証結果を返す。

##### **2. 商品選択～カート操作**

1. **アプリケーション（Frontend）**  
   商品一覧を表示するためのリクエストをバックエンドへ送信。

2. **アプリケーション（Backend）**
   `PRODUCT` テーブルから商品情報を取得。
   必要に応じて `CATEGORY`, `PRODUCT_IMAGE` をJOIN。
   フロントエンドに商品情報を返却。

3. **画面（SC0110）**  
   商品一覧が表示され、ユーザーが商品をクリック。

4. **アプリケーション（Frontend）**
   選択された商品IDで `PRODUCT` の詳細を取得し、商品詳細ページを表示。

5. **画面（SC0120）**  
   商品詳細ページで数量を選び、「カートに追加」ボタンをクリック。

6. **アプリケーション（Frontend）**
   商品ID・数量をローカルセッションに保持。
  「カートを見る」ボタンでカート内容表示。

##### **3. 注文手続き（会員登録住所の取得）**

1. **画面（SC0140）**  
   注文手続き画面に遷移。

2. **アプリケーション（Frontend）**
   会員が登録している住所情報を取得するリクエストを送信。

3. **アプリケーション（Backend）**
   `CUSTOMER` テーブルからログイン済み会員の住所情報を取得。 

4. **画面（SC0140）**  
   取得した登録住所が注文フォームに表示され、ユーザーが選択。

##### **4. 注文確定処理**

1. **画面（SC0150）**  
   決済情報を入力後、注文内容を確認し「注文を確定する」ボタンをクリック。

2. **アプリケーション（Backend）**  
   `ORDER`テーブルに注文基本情報（会員ID含む）を登録。  
   `ORDER_DETAIL`テーブルに商品ごとの注文情報を登録。  

3. **データベース（DB）**  
   `ORDER`, `ORDER_DETAIL`に情報を登録。

4. **アプリケーション（Backend）**  
   注文完了情報をフロントエンドへ返却。

5. **画面（SC0160）**  
   注文完了画面を表示。



#### 例：商品情報の新規登録（管理者）

##### データフロー（シーケンス図）

<div class="mermaid">
sequenceDiagram
    participant 管理者
    participant 管理画面
    participant フロントエンド
    participant バックエンド
    participant DB

    管理者->>管理画面: ログイン画面にアクセス
    管理画面->>フロントエンド: ログイン情報入力送信
    フロントエンド->>バックエンド: ログイン認証リクエスト
    バックエンド->>DB: ADMIN テーブル照合
    DB-->>バックエンド: 認証結果返却
    バックエンド-->>フロントエンド: 認証結果返却
    フロントエンド-->>管理画面: ログイン結果表示（成功なら管理者ダッシュボード画面へ）

    管理者->>管理画面: 商品情報追加・編集画面に遷移
    管理画面->>フロントエンド: 商品情報・画像URL入力
    フロントエンド->>バックエンド: 商品登録リクエスト

    バックエンド->>DB: PRODUCTテーブルへ商品情報登録
    バックエンド->>DB: PRODUCT_IMAGEテーブルへ画像情報登録
    DB-->>バックエンド: 登録成功レスポンス
    バックエンド-->>フロントエンド: 登録完了レスポンス
    フロントエンド-->>管理画面: 商品登録完了表示
</div>

1. **画面（SC9001）**  
   管理者がログイン画面にアクセスし、IDとパスワードを入力して「ログイン」ボタンをクリック。

2. **アプリケーション（Backend）**  
   ログイン情報を受け取り、`ADMIN`テーブルで認証。  
   認証成功の場合は管理者ダッシュボード画面へ遷移。

3. **画面（SC9101）**  
   管理者ダッシュボード画面に遷移し、商品情報追加・編集画面へのリンクをクリック。

4. **画面（SC9201）**  
   商品情報追加・編集画面で新規商品の情報（商品名、価格、カテゴリ、説明など）を入力し商品画像ファイルと表示順を選択。

5. **アプリケーション（Backend）**  
   受け取った商品情報を`PRODUCT`テーブルに登録。
   画像ファイル情報を`PRODUCT_IMAGE`テーブルに登録。

6. **データベース（DB）**  
   `PRODUCT`, `PRODUCT_IMAGE`に情報を登録。

7. **アプリケーション（Backend）**  
   登録成功の結果を画面に返却。

8. **画面（SC9201）**  
   商品登録完了のメッセージを表示。



### 5.4 拡張機能（決済と配送のステータス管理・在庫管理・購入履歴・レビュー・お気に入り・問い合わせ・管理者変更ログ）

#### 5.4.1 概念データモデル（ER図）

<div class="mermaid">
erDiagram
    CUSTOMER ||--o{ ORDER : "places"
    ORDER ||--|{ ORDER_DETAIL : "has"
    ORDER ||--|| PAYMENT : "includes"
    ORDER_DETAIL }|--|| PRODUCT : "includes"
    PRODUCT }|--|| CATEGORY : "belongs to"
    PRODUCT ||--o{ PRODUCT_IMAGE : "has"
    ORDER ||--|| SHIPPING : "has"
    ADMIN ||--o{ ADMIN_LOG : "makes"

    PRODUCT ||--|| INVENTORY : "has"
    CUSTOMER ||--o{ PURCHASE_HISTORY : "has"
    CUSTOMER ||--o{ REVIEW : "writes"
    CUSTOMER ||--o{ FAVORITE : "favorites"
    CUSTOMER ||--o{ INQUIRY : "makes"
    INQUIRY ||--o{ INQUIRY_RESPONSE : "has"
    ADMIN ||--o{ INQUIRY_RESPONSE : "writes"
    CUSTOMER ||--o{ INQUIRY_RESPONSE : "writes"

    PURCHASE_HISTORY }|--|| ORDER : "refers"
    REVIEW }|--|| PRODUCT : "reviews"
    FAVORITE }|--|| PRODUCT : "favorites"

    CUSTOMER {
        string 会員ID PK
        string 氏名
        string メールアドレス
        string ハッシュパスワード
        string 住所
        datetime 登録日時
        boolean メンバーシップ
    }

    ADMIN {
        string 管理者ID PK
        string 氏名
        string メールアドレス
        string パスワード
        datetime 登録日時
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
        string 氏名
        string 配送先住所
        decimal 送料
        string 配送ステータス
    }

    ADMIN_LOG {
        string ログID PK
        string 管理者ID FK
        string 操作内容
        datetime 操作日時
    }

    PAYMENT {
        string 決済ID PK
        string 注文ID FK
        string 決済方法
        string 決済状況
    }

    INVENTORY {
        string 在庫ID PK
        string 商品ID FK
        int 在庫数
        datetime 最終更新日時
    }

    PURCHASE_HISTORY {
        string 履歴ID PK
        string 会員ID FK
        string 注文ID FK
        datetime 購入日時
        decimal 購入金額
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
        string 内容
        datetime 送信日時
        string 対応状態
    }

    INQUIRY_RESPONSE {
        string 返信ID PK
        string 問い合わせID FK
        string 返信者種別
        string 返信者ID
        string 内容
        datetime 返信日時
    }
</div>

- **エンティティ**
   - CUSTOMER（顧客）
   - ADMIN（管理者）
   - ORDER（注文）
   - ORDER_DETAIL（注文詳細）
   - PRODUCT（商品）
   - CATEGORY（カテゴリ）
   - PRODUCT_IMAGE（商品画像）
   - SHIPPING（配送情報）
   - PAYMENT（決済情報）
   - ADMIN_LOG（管理者ログ）
   - INVENTORY（在庫）
   - REVIEW（レビュー）
   - FAVORITE（お気に入り）
   - INQUIRY（問い合わせ）
   - INQUIRY_RESPONSE（問い合わせ返信）
  
- **リレーション**
  - 顧客は注文を行う（1対多、CUSTOMER → ORDER）  
  - 注文は複数の注文詳細を持つ（1対多、ORDER →  ORDER_DETAIL）  
  - 注文は配送情報を持つ（1対1、ORDER → SHIPPING） 
  - 注文は決済情報を持つ（1対1、ORDER → PAYMENT） 
  - 注文詳細は商品を含む（多対1、ORDER_DETAIL → PRODUCT）  
  - 商品はカテゴリに属する（多対1、PRODUCT → CATEGORY）  
  - 商品は複数の画像を持つ（1対多、PRODUCT → PRODUCT_IMAGE）  
  - 商品は在庫情報を持つ（1対1、PRODUCT → INVENTORY）  
  - 顧客は購入履歴を持つ（1対多、CUSTOMER → PURCHASE_HISTORY）
  - 購入履歴は注文を参照する（多対1、PURCHASE_HISTORY → ORDER）
  - 顧客は商品にレビューを投稿できる（1対多、CUSTOMER → REVIEW、PRODUCT → REVIEW）  
  - 顧客は商品をお気に入り登録できる（1対多、CUSTOMER → FAVORITE、PRODUCT → FAVORITE）  
  - 顧客は問い合わせを送ることができる（1対多、CUSTOMER → INQUIRY）  
  - 管理者は操作ログを記録する（1対多、ADMIN → ADMIN_LOG）
  - 管理者は問い合わせに返答することができる（1対多、ADMIN → INQUIRY_RESPONSE）
  - 顧客は管理者の返答にさらに返答できる（1対多、CUSTOMER → INQUIRY_RESPONSE）
  - 

#### 5.4.2 主要テーブル概要

- **CUSTOMER（顧客テーブル）**
    - ECサイトの会員情報を管理。
    - 会員IDを主キーとし、顧客全員の情報を保持。
    - 氏名、メールアドレス、ハッシュ化パスワード、住所、登録日時、メンバーシップを保持。
    - 非会員の場合は、注文時に入力した情報を注文時にパスワードはNULL、メンバーシップはfalseとして保持。
    - 会員登録が任意のため、注文時の氏名・住所は注文テーブルにも保持し配送情報と連携。

- **ADMIN（管理者テーブル）**
    - ECサイトの管理者情報を管理。
    - 管理者IDを主キーとして管理。
    - 氏名、メールアドレス、ハッシュ化パスワード、登録日時を保持。

- **ORDER（注文テーブル）**
    - 顧客の注文情報を管理。
    - 注文IDを主キーとし、会員IDは外部キーとして関連付ける。
    - 注文日時、合計金額、注文ステータス、注文時の氏名・メールアドレス・住所を保持し配送や請求の基礎情報とする。

- **ORDER_DETAIL（注文詳細テーブル）**
    - 各注文に含まれる商品ごとの詳細情報を管理。
    - 注文詳細IDを主キー、注文IDと商品IDを外部キーとして紐付け。
    - 数量、単価を保持し、複数商品の注文に対応。

- **PRODUCT（商品テーブル）**
    - ECサイトで販売する商品の情報を管理。
    - 商品IDを主キーとし、商品名、商品説明、価格を保持、外部キーとしてカテゴリIDを紐づけ。
    - 商品画像と関連付ける。

- **CATEGORY（カテゴリテーブル）**
    - 商品の分類情報を管理。
    - カテゴリIDを主キーとし、カテゴリ名を保持。

- **PRODUCT_IMAGE（商品画像テーブル）**
    - 商品に紐づく複数の画像情報を管理。
    - 画像IDを主キーとし、商品IDを外部キーとして紐付け。
    - 画像URL、表示順を保持。

- **INVENTORY（在庫テーブル）**
  - 在庫ID（PK）、商品ID（FK）、在庫数、更新日時を保持。
  - 商品との1対1関係を持ち、購入時に自動減少。
  - 入荷の場合は管理画面から在庫数を入力。

- **SHIPPING（配送情報テーブル）**
    - 注文ごとの配送に関する情報を管理。
    - 配送IDを主キー、注文IDを外部キーとして紐付け。
    - 氏名、配送先住所、配送ステータス、送料を保持。

- **PAYMENT（決済情報テーブル）**
    - 注文に紐づく支払い情報を管理。
    - 決済IDを主キー、注文IDを外部キーとして紐付け。
    - 決済方法、決済ステータスを保持。
    - 初期リリースでは振込・代引きを想定。

- **REVIEW（レビュー情報テーブル）**
  - レビューID（PK）、会員ID（FK）、商品ID（FK）、評価（数値）、コメント、投稿日を保持。
  - 会員1名につき同一商品へのレビューは1件までを原則とする。

- **FAVORITE（お気に入りテーブル）**
  - お気に入りID（PK）、会員ID（FK）、商品ID（FK）、登録日時を保持。
  - 重複登録を防ぐため、会員IDと商品IDの組み合わせにユニーク制約を付与可能。

- **INQUIRY（問い合わせテーブル）**
  - 問い合わせID（PK）、会員ID（FK, NULL許容）、メールアドレス、内容、対応ステータス、問い合わせ日時を保持。
  - 未ログインのユーザーも問い合わせ可能。
  
- **INQUIRY_RESPONSE（問い合わせ返信テーブル）**
  - 返信ID（PK）、問い合わせID（FK）、返信者種別（"admin"または"customer"）、返信者ID、内容、返信日時を保持。
  - 問い合わせに対する複数の返信を保持し、顧客・管理者双方が返信可能。

- **ADMIN_LOG（管理者ログテーブル）**
    - 管理者による操作履歴を管理。
    - ログIDを主キー、管理者IDを外部キーとして紐付け。
    - 操作内容、操作日時を保持。
