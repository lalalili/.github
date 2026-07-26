# lalalili/.github

組織層級的共用設定：可重用的 GitHub Actions workflow、Renovate 預設、語意化版本契約。

## 可重用 workflow

各套件 repo 不再各自複製 CI 設定，改為呼叫這裡的 workflow。

### PHP 套件 CI

`.github/workflows/ci.yml`：

```yaml
name: CI
on: [push, pull_request]
jobs:
  ci:
    uses: lalalili/.github/.github/workflows/php-package-ci.yml@main
    secrets: inherit
```

可用 inputs：

| input | 預設 | 說明 |
|---|---|---|
| `php-versions` | `['8.3','8.4']` | PHP 版本矩陣 |
| `laravel-versions` | `[""]` | 要釘住的 `laravel/framework` 版本；`[""]` 表示不釘 |
| `php-extensions` | `dom, mbstring, pdo_sqlite, zip` | setup-php 擴充套件 |
| `run-validate` | `true` | 執行 `composer validate --strict` |
| `run-analyse` | `true` | 執行 `composer analyse` |
| `composer-flags` | `--prefer-dist` | 傳給 `composer update` 的額外參數 |

### PHP 套件發版

`.github/workflows/release.yml`：

```yaml
name: Release
on:
  push:
    tags: ['v*']
jobs:
  release:
    uses: lalalili/.github/.github/workflows/php-package-release.yml@main
    secrets: inherit
```

跑完 validate + test + analyse 才會建立 GitHub Release，避免發出壞掉的 tag。

### 前端套件 CI

```yaml
name: CI
on: [push, pull_request]
jobs:
  ci:
    uses: lalalili/.github/.github/workflows/js-package-ci.yml@main
```

## 私有套件存取

`audience-core`、`marketing-automation`、`package-testing-support` 是私有 repo。
依賴它們的套件，其 CI 需要能讀取私有 repo 的 token。

在組織層設定 secret **`COMPOSER_TOKEN`**（Settings → Secrets and variables →
Actions → New organization secret，範圍給 lalalili 的 repo），caller workflow
寫 `secrets: inherit` 即可自動帶入。只依賴公開套件的 repo 不設也能跑。

## Renovate

各 repo 的 `renovate.json`：

```json
{ "extends": ["github>lalalili/.github"] }
```

## 版本契約

發版前請讀 [SEMVER.md](SEMVER.md)，裡面定義了哪些東西算 public API、
deprecation 流程，以及套件之間的發版相依順序。
