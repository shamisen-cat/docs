# Symfony

- [Symfony](#symfony)
  - [コマンド一覧](#コマンド一覧)

## コマンド一覧

| 分類         | 操作                       | コマンド                                             |
| ------------ | -------------------------- | ---------------------------------------------------- |
| 基本         | プロジェクト情報の表示     | `php bin/console about`                              |
|              | コマンド一覧の表示         | `php bin/console list`                               |
|              | キャッシュのクリア         | `php bin/console cache:clear`                        |
| デバッグ     | ルート一覧の表示           | `php bin/console debug:router`                       |
|              | サービスの確認             | `php bin/console debug:container`                    |
|              | オートワイヤリングの確認   | `php bin/console debug:autowiring`                   |
| 検証         | コンテナ設定の検証         | `php bin/console lint:container`                     |
|              | Twig テンプレートの検証    | `php bin/console lint:twig`                          |
|              | YAML の検証                | `php bin/console lint:yaml config`                   |
| データベース | データベースの作成         | `php bin/console doctrine:database:create`           |
|              | マイグレーションの生成     | `php bin/console make:migration`                     |
|              | マイグレーションの実行     | `php bin/console doctrine:migrations:migrate`        |
| Tailwind     | CSS のビルド               | `php bin/console tailwind:build`                     |
|              | 変更の監視                 | `php bin/console tailwind:build --watch`             |
| Lucide       | アイコンのインポート       | `php bin/console ux:icons:import lucide:<icon-name>` |
|              | 使用中アイコンのインポート | `php bin/console ux:icons:lock`                      |
