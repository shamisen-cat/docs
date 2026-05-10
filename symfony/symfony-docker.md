# Symfony: Symfony Docker

- [Symfony: Symfony Docker](#symfony-symfony-docker)
  - [Dev Container の構築](#dev-container-の構築)
    - [Symfony Docker をクローンする](#symfony-docker-をクローンする)
    - [Symfony Docker の設定を調整する](#symfony-docker-の設定を調整する)
      - [Dockerfile](#dockerfile)
      - [.devcontainer/devcontainer.json](#devcontainerdevcontainerjson)
      - [.devcontainer/init-firewall.sh](#devcontainerinit-firewallsh)
      - [.github/workflows/ci.yaml](#githubworkflowsciyaml)
      - [README.md](#readmemd)
    - [Symfony の実行環境を構築する](#symfony-の実行環境を構築する)
    - [Dev Container を起動する](#dev-container-を起動する)

## Dev Container の構築

### Symfony Docker をクローンする

GitHub リポジトリ

[https://github.com/dunglas/symfony-docker](https://github.com/dunglas/symfony-docker)

```bash
git clone git@github.com:dunglas/symfony-docker.git <project-name>
```

ブランチ名を確認するため、リモートリポジトリを表示する。

```bash
git remote -v
```

クローンしたリモートブランチの名前（例: `upstream`）を変更する。

```bash
git remote rename origin upstream
```

クローンしたリモートブランチのプッシュを禁止する。

```bash
git remote set-url --push upstream DISABLED
```

### Symfony Docker の設定を調整する

#### Dockerfile

PHP バージョン（例: PHP 8.4）を変更する。

```dockerfile
FROM dunglas/frankenphp:1-php8.4 AS frankenphp_upstream
```

#### .devcontainer/devcontainer.json

`name` を変更する。

```json
{
  "name": "Project Name"
}
```

VSCode 拡張機能を追加・削除する。

設定例

```json
{
    "customizations": {
        "vscode": {
            "extensions": [
                "bmewburn.vscode-intelephense-client",
                "davidanson.vscode-markdownlint",
                "dbaeumer.vscode-eslint",
                "esbenp.prettier-vscode",
                "gruntfuggly.todo-tree",
                "junstyle.php-cs-fixer",
                "mblode.twig-language-2",
                "streetsidesoftware.code-spell-checker",
                "xdebug.php-debug",
                "yzhang.markdown-all-in-one"
            ]
        }
    }
}
```

Claude Code を使用しない場合には、Claude Code の設定を削除する。

#### .devcontainer/init-firewall.sh

`ipset` に許可対象の URL（例: `data.jsdelivr.com`）を追加する。

```sh
ipset=/github.com/anthropic.com/sentry.io/statsig.com/registry.npmjs.org/packagist.org/cdn.jsdelivr.net/data.jsdelivr.com/marketplace.visualstudio.com/vscode.blob.core.windows.net/update.code.visualstudio.com/allowed-domains
```

#### .github/workflows/ci.yaml

Symfony 構成で Super-Linter を使用する場合、`ext-xml` がなく PHP のバリデーションで失敗するため、`.github/workflows/ci.yaml` を調整する。

- `tests` に `composer validate --strict` を追加し、コンテナ内で検証する。
- `lint` の `VALIDATE_PHP_*` を無効にする。

Symfony のフォーマットルールと相性が悪いため、`prettier` と `codespell` を無効にする。

```yaml
jobs:
  tests:
    steps:
      # ... 省略 ...
      - name: Start services
        run: docker compose up --wait --no-build

      # custom: validate composer integrity inside the app container (where ext-xml is available)
      - name: Validate Composer (inside app container)
        run: docker compose exec -T php composer validate --strict

      - name: Check HTTP reachability
        run: curl -v --fail-with-body http://localhost

  lint:
    steps:
      - name: Lint Code Base
        uses: super-linter/super-linter/slim@v8
        env:
          # ... 省略 ...
          VALIDATE_BIOME_FORMAT: false
          VALIDATE_BIOME_LINT: false

          # custom: avoid composer install inside super-linter (no ext-xml in linter runtime)
          VALIDATE_PHP_BUILTIN: false
          VALIDATE_PHP_PHPCS: false
          VALIDATE_PHP_PHPSTAN: false
          VALIDATE_PHP_PSALM: false

          # custom: prettier/codespell are not used in this project
          VALIDATE_YAML_PRETTIER: false
          VALIDATE_SPELL_CODESPELL: false
```

#### README.md

- 先頭見出しをリポジトリ名などに変更する。
- CI バッジの URL を変更またはバッジを削除する。
- ライセンス表記とクレジットを更新する。

```markdown
# Repository Name

## License

This project is licensed under the MIT License. See [LICENSE](./LICENSE).

## Credits

This project is based on [dunglas/symfony-docker](https://github.com/dunglas/symfony-docker).
Original work by [Kévin Dunglas](https://dunglas.dev) and contributors.
```

### Symfony の実行環境を構築する

Docker イメージをビルドする。

```bash
docker compose build --pull --no-cache
```

- `--pull`: ビルド前に最新のベースイメージをプルする。
- `--no-cache`: 既存のビルドキャッシュを使用せず、すべてのレイヤーをビルドする。

Symfony バージョン（例: Symfony 7.4）を指定し、Docker サービスを起動する。

```bash
SYMFONY_VERSION=7.4.* STABILITY=stable docker compose up --wait
```

- `STABILITY=stable`: 安定版（stable）のみを対象に依存関係を解決する。
- `--wait`: サービス利用可能まで待機する。

`composer.json` の PHP バージョン（例: PHP 8.4）を変更する。

```json
{
    "require": {
        "php": ">=8.4 <8.5"
    }
}
```

`LICENSE` の著作権者表記を更新し、適用ライセンスを併記する。

```text
Copyright (c) Kévin Dunglas and contributors
Copyright (c) 2026 Your Name
```

パッケージのインストールや依存のアップグレードを行わず、`composer.lock` を書き換える。

```bash
docker compose exec php composer update --lock
```

`composer.lock` を基に `vendor` にパッケージをインストールする。

```bash
docker compose exec php composer install
```

PHP の実行環境がパッケージのプラットフォーム条件を満たしているか確認する。

```bash
docker compose exec php composer check-platform-reqs
```

PHP と Symfony のバージョンを確認する。

```bash
docker compose exec php php -v
docker compose exec php php bin/console --version
```

- [https://localhost](https://localhost) にアクセスし、サーバ接続を確認する。
- `src/Controller` にコントローラ（例: `HelloController.php`）を作成し、接続を確認する。

Docker サービスを停止し、関連リソースを削除する。

```bash
docker compose down --remove-orphans
```

- `--remove-orphans`: プロジェクト内の管理外となった孤立コンテナを削除する。

### Dev Container を起動する

コマンドパレット（`Dev Containers: Rebuild Without Cache and Reopen in Container`）などで Dev Container をビルドする
