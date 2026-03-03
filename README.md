# EnemyDown

制限時間内にランダムで出現する敵を倒してスコアを競う、Minecraft（Spigot）のミニゲームプラグインです。


# はじめに
本リポジトリは、「すずき」（Xアカウント： (@mr_suzuki_works)が作成したMinecraftプラグインに関するものです。
ご利用いただくことによるトラブル等につきましては、一切の責任を負いかねますことを予めご了承ください。


## 目次
- [ゲーム概要](#ゲーム概要)
- [主な機能](#主な機能)
- [コマンド](#コマンド)
- [技術スタック](#技術スタック)
- [プロジェクト構成](#プロジェクト構成)
- [設計のポイント](#設計のポイント)
- [セットアップ](#セットアップ)
- [おわりに](#おわりに)


## ゲーム概要

プレイヤーがコマンドを実行すると、周囲にランダムで敵がスポーンします。制限時間20秒以内にできるだけ多くの敵を倒し、高スコアを目指します。スコアは敵の種類によって異なり、結果はデータベースに保存されます。

## 主な機能

- **難易度選択**: easy / normal / hard の3段階。難易度に応じて出現する敵の種類が変化
- **スコアシステム**: ゾンビ（10点）、スケルトン・ウィッチ（20点）
- **マルチプレイ対応**: 複数プレイヤーが同時にプレイ可能。スコアは個別に管理
- **スコア保存・閲覧**: MySQLデータベースにプレイヤー名・スコア・難易度・日時を記録し、ゲーム内コマンドで一覧表示
- **自動セットアップ**: ゲーム開始時に体力・空腹値の最大化、ネザライト装備の自動装着
- **状態管理**: ゲーム開始・終了時にポーション効果をクリア。終了時にスポーンした敵を自動削除

## コマンド

| コマンド | 説明 |
|---|---|
| `/enemyDown easy` | 難易度easyでゲーム開始（ゾンビのみ出現） |
| `/enemyDown normal` | 難易度normalでゲーム開始（ゾンビ、スケルトン） |
| `/enemyDown hard` | 難易度hardでゲーム開始（ゾンビ、スケルトン、ウィッチ） |
| `/enemyDown list` | スコアの履歴を一覧表示 |

## 技術スタック

| 技術 | 用途 |
|---|---|
| Java 21 | メイン言語 |
| Spigot API | Minecraftサーバープラグイン開発 |
| MySQL | スコアデータの永続化 |
| MyBatis | O/Rマッピング（DB操作の簡素化） |
| Lombok | ボイラープレートコードの削減 |
| Gradle | ビルド・依存関係管理 |

## プロジェクト構成

```
src/main/java/plugin/enemydown/
├── Main.java                          # プラグインのエントリーポイント
├── PlayerScoreDate.java               # DB接続・操作を管理するクラス
├── command/
│   ├── BaseCommand.java               # コマンドの基底クラス（抽象クラス）
│   └── EnemyDownCommand.java          # ゲームのメインロジック
├── date/
│   └── ExecutingPlayer.java           # ゲーム中のプレイヤー情報
└── mapper/
    ├── PlayerScoreMapper.java         # MyBatisマッパー（SQL定義）
    └── date/
        └── PlayerScore.java           # DBテーブルと連動するデータクラス
```

## 設計のポイント

- **BaseCommandによる共通処理の抽出**: プレイヤー判定ロジックを抽象クラスに集約し、各コマンドは固有の処理のみを実装（DRY原則）
- **データクラスの分離**: ゲーム中のプレイヤー情報（ExecutingPlayer）とDB保存用のスコア情報（PlayerScore）を役割ごとに分離
- **MyBatisによるDB操作の簡素化**: アノテーションベースのSQL定義により、JDBC直書きを排除
- **PlayerScoreDateによるDB処理の集約**: DB接続とCRUD操作を専用クラスに切り出し、コマンドクラスの責務を軽量化

## セットアップ

### 前提条件

- Java 21
- Minecraft Spigot Server 1.21
- MySQL 8.0以上

### 手順

1. MySQLでデータベースとテーブルを作成:

```sql
CREATE DATABASE spigot_server;

USE spigot_server;

CREATE TABLE player_score (
    id INT AUTO_INCREMENT PRIMARY KEY,
    player_name VARCHAR(255),
    score INT,
    difficulty VARCHAR(50),
    registered_at DATETIME
);
```

2. `src/main/resources/mybatis-config.xml` のDB接続情報を環境に合わせて編集

3. プロジェクトをビルド:

```
./gradlew build
```

4. `build/libs/EnemyDown-1.0-SNAPSHOT.jar` をSpigotサーバーの `plugins` フォルダに配置

5. サーバーを起動してゲーム内で `/enemyDown easy` を実行

# おわりに
Java学習のアウトプットとして、本リポジトリを公開させていただきました。
ご感想やコメントなどがございましたら、Xまでご連絡いただけますと幸いです。

