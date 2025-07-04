# 備品管理システム 基本設計書

| ドキュメントバージョン | 1.0                                    |
| :------------------- | :------------------------------------- |
| 作成日               | 2025年4月20日                           |
| 作成チーム           | チーム〇〇                               |
| 承認者               | 〇〇 〇〇様                              |
| 更新履歴             | 2025/04/20: Ver.1.0 初版作成 (チーム〇〇) |

---

## 1. はじめに

### 1.1. 本書の目的

本書は、株式会社〇〇が開発を進める「ＥＣサイト」新規構築プロジェクトにおける基本設計の内容を定義するものである。要件定義書で定められた要件に基づき、システムの全体構成、主要な機能、画面、データ、非機能要件への対応方針などを明確にし、後続の詳細設計、実装、テスト工程のインプットとすることを目的とする。

### 1.2. プロジェクト概要

本プロジェクトは株式会社〇〇のシンプルで質のいい雑貨をより多くの顧客に届けることを目的とし、新規ECサイトを開発・導入するものである。実店舗中心の販売から脱却し、オンラインでの商品販売を可能にすることで、販路と顧客層の拡大を図る。

### 1.3. 前提知識

本書を読むにあたり、以下の知識を有していることを前提とする。

- 本プロジェクトの要件定義書の内容
- Webアプリケーション開発の基本的な知識 (HTTP, HTML, CSS, JavaScript)
- REST API の基本的な概念
- Java および Spring Boot フレームワークの基本的な知識
- リレーショナルデータベースの基本的な知識


## 2. システム概要

### 2.1. システムの目的

本システムは、販売チャネルの確立、商品の魅力訴求、販売業務の効率化を目的とする。


### 2.2. 対象ユーザー

本システムの主な利用者は以下の通り。

- **(1)管理者**:サイトの全体的な管理・運用を行う
  
- **(2)一般消費者（20代後半～40代くらいの男女）**:商品を購入する消費者


### 2.3. システム構成図



<div class="mermaid">
graph LR
  subgraph クライアント環境
    PC[PCブラウザ<br>（Chrome / Edge）]
    SP[スマートフォン<br>（Safari / Chrome）]

  end

  
INTERNET[インターネット]


  subgraph AWS環境[（クラウド）]
    LB[ALB （ロードバランサ）]
    WEB[Web/APサーバ　<br>（Laravel / PHP-FPM）]
    DB[DBサーバ<br>（MySQL 8.0）]
    FS[ファイルストレージ<br>（S3等）]
    WAF[WAF<br>（AWS WAF / Cloudflare）]
  end


  PC -->|HTTPS| INTERNET -->|TLS通信| WAF --> LB --> WEB
  SP -->|HTTPS| INTERNET

  WEB -->|API / DB接続| DB
  WEB -->|画像取得・保存| FS

  style PC fill:#ffffcc,stroke:#333,stroke-width:1px
  style SP fill:#ffffcc,stroke:#333,stroke-width:1px
  style INTERNET fill:#eee,stroke:#999,stroke-width:1px,stroke-dasharray: 5 5
  style WAF fill:#ffcc99,stroke:#333,stroke-width:1px
  style LB fill:#cce5ff,stroke:#333,stroke-width:2px
  style WEB fill:#cce5ff,stroke:#333,stroke-width:2px
  style DB fill:#ccffcc,stroke:#333,stroke-width:1px
  style FS fill:#eeeeee,stroke:#333,stroke-width:1px

</div>

- **クライアント環境**: 
PCブラウザ（Chrome/Edge）あるいはスマートフォン（Safari/Chrome）

- **クラウド**:
Web/APサーバー：ECサイトのアプリケーションをホスト
DBサーバー：商品情報、注文情報などを管理
ファイルストレージ（オプション）：商品画像などの静的ファイルを格納

- **インターネット**:
クライアントからWebサーバーへはHTTPS通信
Webサーバーから各サーバーへは内部接続

### 2.4. 外部インターフェース概要

- **CSVファイル取り込み**  
  - 商品情報の初期登録・一括更新のためにCSVファイルをアップロード可能。
  - 管理画面から操作し、サーバー側でパース・登録処理を行う。

- **認証サーバー（オプション）**  
  - 将来的にSSO（シングルサインオン）や社内AD連携を検討。
  - 管理者ログインの利便性向上を目的とする。

