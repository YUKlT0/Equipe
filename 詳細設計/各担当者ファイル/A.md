# "Welcome to TEINEI "  詳細設計書


| ドキュメントバージョン | 1.0                                   |
| :------------------- | :------------------------------------ |
| 作成日               | 2025年7月7日                          |
| 作成チーム           |  Equipe |
| 更新履歴             | 2025/07/07: Ver.1.0 初版作成 (チーム Equipe) |

## 1. はじめに

### 1.1. 本書の目的

本書は、「" Welcome to TEINEI "」新規構築プロジェクトにおける詳細設計の内容を定義するものです。基本設計書 Ver.1.0 で定義された内容に基づき、実装担当者がプログラミング作業を迷いなく進められるように、システムの内部構造、処理フロー、インターフェース、データベース構造、画面項目などを具体的に記述します。

### 1.2. 前提となる基本設計書

本書は、以下の基本設計書の内容を前提としています。

-  "Welcome to TEINEI " 基本設計書 Ver.1.0


### 1.3. 対象読者

本書は、以下の担当者を対象としています。

- 本システムのバックエンド開発担当者
- 本システムのフロントエンド開発担当者
- 本システムのテスト担当者
- プロジェクト管理者

### 1.4. 参考文献

- " Welcome to TEINEI "   基本設計書 Ver.1.0
- (チーム内で使用するコーディング規約などがあれば記載)

## 2. システム概要

システムの目的、対象ユーザー、全体構成、外部インターフェースについては、基本設計書 Ver.1.0 の「2. システム概要」に記載の通りです。

**システム構成（再掲）**

<div class="mermaid">
graph LR
  subgraph クライアント環境
    PC[PCブラウザ<br>（Chrome / Edge）]
    SP[スマートフォン<br>（Safari / Chrome）]

  end

INTERNET[インターネット]

  subgraph AWS環境[（クラウド）]
    LB[ALB （ロードバランサ）]
    WEB[Web/APサーバ　`<br>`（Java）]
    DB[DBサーバ`<br>`（MySQL 8.0）]
    FS[ファイルストレージ`<br>`（S3等）]
    WAF[WAF`<br>`（AWS WAF / Cloudflare）]
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