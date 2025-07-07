# シンプル雑貨オンライン詳細設計書（必須機能のみ）

※本バージョンは、基本設計の中から必須機能（商品閲覧、カート操作、非会員としての注文確定）の実装に絞った構成としています。

| ドキュメントバージョン | 1.0                                   |
| :------------------- | :------------------------------------ |
| 作成日               | 2025年7月7日                          |
| 作成チーム           |  Equipe |
| 更新履歴             | 2025/07/07: Ver.1.0 初版作成 (チーム Equipe) |

## 1. はじめに

### 1.1. 本書の目的

本書は、「シンプル雑貨オンライン」新規構築プロジェクトにおける詳細設計の内容を定義するものです。基本設計書 Ver.1.0 で定義された内容に基づき、実装担当者がプログラミング作業を迷いなく進められるように、システムの内部構造、処理フロー、インターフェース、データベース構造、画面項目などを具体的に記述します。

### 1.2. 前提となる基本設計書

本書は、以下の基本設計書の内容を前提としています。

- シンプル雑貨オンライン 基本設計書 Ver.1.0

### 1.3. 対象読者

本書は、以下の担当者を対象としています。

- 本システムのバックエンド開発担当者
- 本システムのフロントエンド開発担当者
- 本システムのテスト担当者
- プロジェクト管理者

### 1.4. 参考文献

- シンプル雑貨オンライン 基本設計書 Ver.1.0
- (チーム内で使用するコーディング規約などがあれば記載)

## 2. システム概要

システムの目的、対象ユーザー、全体構成、外部インターフェースについては、基本設計書 Ver.1.0 の「2. システム概要」に記載の通りです。

**システム構成（再掲）**

<div class="mermaid">
graph LR
    subgraph ユーザー環境
        A[クライアント<br>（PC/スマホ ブラウザ）<br>HTML/CSS/JavaScript]
    end

    subgraph "バックエンド (ローカル or AWS)"
        subgraph "APIサーバー (Spring Boot)"
            direction LR
            B[Controller<br>] --> C(Service<br>);
            C --> D(Repository<br>);
        end
        subgraph DBサーバー
            E[データベース<br>（H2 / PostgreSQLなど）]
        end
        D --> E;
    end

    A -- ① APIリクエスト<br>(HTTP GET, POST...) --> B;
    B -- ② 処理依頼 --> C;
    C -- ③ データアクセス --> D;
    D -- ④ SQL実行 --> E;
    E -- ⑤ データ --> D;
    D -- ⑥ データ --> C;
    C -- ⑦ 処理結果 --> B;
    B -- ⑧ APIレスポンス<br>(JSONデータ) --> A;

    style A fill:#lightyellow,stroke:#333,stroke-width:1px
    style B fill:#lightblue,stroke:#333,stroke-width:1px
    style C fill:#lightgreen,stroke:#333,stroke-width:1px
    style D fill:#lightcoral,stroke:#333,stroke-width:1px
    style E fill:#lightgrey,stroke:#333,stroke-width:1px
</div>