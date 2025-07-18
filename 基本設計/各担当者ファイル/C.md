C
## 4. 画面設計

本章では、想定している主要画面のレイアウトや画面間の遷移など、UI設計の概要を示す。

### 4.1. 画面一覧

- **SC0001: ユーザーログイン画面**
    - メールアドレスとパスワードを入力し、システムにログインする画面。新規会員登録やゲストログインも配置。
- **SC0010: 新規会員登録画面**
    - 氏名、住所、メールアドレス、パスワードを新規登録する画面。
- **SC0110: 商品一覧画面**
    - システムに登録されている商品を一覧表示する画面。カテゴリー絞り込み、納期情報の表示などを配置。ホーム画面の役割を果たす。
- **SC0115: カテゴリ別商品一覧画面**
    - ユーザーが選択したカテゴリで絞り込み、商品一覧画面(SC0110)と同様にシステムに登録されている商品を一覧表示する画面。
- **SC0120: 商品詳細画面**
    - 選択した商品の詳細情報（大きめの商品画像、説明、価格、素材情報など）を表示する画面。
- **SC0125: マイページ画面**
    - 会員登録をしたユーザー専用画面。
- **SC0128: 会員情報編集画面**
    - 会員登録済みのユーザーが登録内容を編集する画面。
- **SC0130: カート画面**
    - カートに追加されている商品を表示・追加・削除する画面。
- **SC0140: 注文情報入力画面**
    - 購入者氏名、購入者住所、購入者メールアドレス、配送先氏名、配送先住所、決済方法を入力・選択する画面。
- **SC0150: 注文情報確認画面**
    - 注文情報入力画面(SC0140)で入力された情報を確認し、確定させる画面。
- **SC0160: 注文完了画面**
    - 注文が正常に完了したことを伝える画面。
- **SC9001: 管理者ログイン画面** 
    - 管理者用ユーザーIDとパスワードを入力し、システムにログインする画面。
- **SC9101: 管理者ダッシュボード画面**
    - 管理者向けのホーム画面。
- **SC9201: 商品情報追加・編集画面** (管理者向け)
    - 新規商品情報の追加や、商品情報の編集を行う画面。
- **SC9301: 注文情報確認画面** (管理者向け)
    - 注文状況を確認する画面。
- **SC9401: 管理者編集画面**
    - 管理者の登録や編集・削除を行う画面。
- (その他、必要に応じて画面を追加)

### 4.2. 画面遷移図

以下に、主要な画面遷移を示す。

<div class="mermaid">
graph TD
    A[SC0001: ユーザーログイン画面] -- ログイン --> C(SC0110: 商品一覧画面);

    A -- 会員登録 --> B(SC0010: 新規会員登録画面);

    B -- 自動でログイン --> C;

    A -- ゲストログイン --> C;

    C -- カテゴリ選択 --> D(SC0115: カテゴリ別商品一覧画面);
    D -- カテゴリ解除 --> C;
    D -- 商品選択 --> E(SC0120: 商品詳細画面);
    E -- 一覧へ戻る --> D;
　
    C -- マイページ選択 --> V(SC0125: マイページ画面);
    V -- 一覧へ戻る --> C;

    V -- 会員情報編集選択 --> W(SC0128: 会員情報編集画面);
    W -- マイページへ戻る --> V;

    C -- 商品選択 --> E(SC0120: 商品詳細画面);
    E -- 一覧へ戻る --> C;

    C -- カートを見る --> F(SC0130: カート画面);
    D -- カートを見る --> F;
    E -- カートを見る --> F;
    F -- 一覧へ戻る --> C;
    F -- 商品へ戻る --> E;
  
    F -- 注文へ進む --> G(SC0140: 注文情報入力画面);
    G -- カートへ戻る --> F;
    G -- 注文確認 --> H(SC0150: 注文情報確認画面);
    H -- 修正 --> G;
    H -- 確定させる --> I(SC0160: 注文完了画面);
    I -- 一覧へ戻る --> C;

    C -- ログアウト --> A;
    D -- ログアウト --> A;
    E -- ログアウト --> A; 

    style A fill:#cce5ff,stroke:#333,stroke-width:1px
    style B fill:#cce5ff,stroke:#333,stroke-width:1px
    style C fill:#d4edda,stroke:#333,stroke-width:1px
    style D fill:#d4edda,stroke:#333,stroke-width:1px
    style E fill:#d4edda,stroke:#333,stroke-width:1px,color:#000
    style F fill:#fff3cd,stroke:#333,stroke-width:1px,color:#000
    style G fill:#fff3cd,stroke:#333,stroke-width:1px,color:#000
    style H fill:#fff3cd,stroke:#333,stroke-width:1px,color:#000
    style I fill:#fff3cd,stroke:#333,stroke-width:1px,color:#000
    style V fill:#e2d6f3,stroke:#333,stroke-width:1px,color:#000
    style W fill:#e2d6f3,stroke:#333,stroke-width:1px,color:#000

</div>

<div class="mermaid">
graph TD
    J[SC9001: 管理者ログイン画面] -- ログイン --> K(SC9101: 管理者ダッシュボード画面);
    K -- 商品情報管理選択 --> L(SC9201: 商品情報登録・編集画面);
    L -- ホームへ戻る --> K;
    K -- 注文情報確認選択 --> M(SC9301: 注文情報確認画面);
    M -- ホームへ戻る --> K;
    K -- 管理者編集選択 --> N(SC9401: 管理者編集画面);
    N -- ホームへ戻る --> K;
    K -- ログアウト --> J;
    L -- ログアウト --> J;
    M -- ログアウト --> J;    

    style J fill:#cce5ff,stroke:#333,stroke-width:1px,color:#000
    style K fill:#e2e3e5,stroke:#333,stroke-width:1px,color:#000
    style L fill:#d4edda,stroke:#333,stroke-width:1px,color:#000
    style M fill:#fff3cd,stroke:#333,stroke-width:1px,color:#000 
    style N fill:#e2d6f3,stroke:#333,stroke-width:1px,color:#000  


</div>

### 4.3. UI/UX基本方針

本システムのユーザーインターフェースは、利用者がストレスなく操作できることを最優先に設計する。以下に、本システムにおけるUI/UX設計の基本方針を示す。

- **直感的な操作性**  
  誰でも迷わず使えるよう、画面レイアウトやナビゲーション、ボタン配置をわかりやすく設計する。

- **統一されたデザインルール**  
  画面間で一貫性のあるデザイン（ボタンの配置・配色、フォント、アイコンなど）を維持し、操作感を揃えることで、ユーザーの混乱を防ぐ。

- **視認性・アクセシビリティの確保**  
  文字サイズや色のコントラストに配慮し、あらゆるユーザーが快適に情報を閲覧できるようにする。特に重要な情報やアクションボタンは視線の導線上に配置する。

- **操作に対する即時フィードバック**  
  ユーザーのアクションに対しては、読み込み中の状態や完了・エラーなどの結果を適切なタイミングと方法で明示し、安心感を与える。

- **スマートフォン対応**  
  スマートフォンで快適に利用できるよう、レスポンシブ対応を行う。タッチ操作も考慮したUI設計とする。
