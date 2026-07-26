# lalalili/.github

`lalalili/*` 套件共用的設定：可重用的 GitHub Actions workflow、Renovate 預設、語意化版本契約。

- [PACKAGES.md](PACKAGES.md) — 套件索引，接新宿主前先看這份
- [SEMVER.md](SEMVER.md) — public API 邊界與發版相依順序

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
| `run-test` | `true` | 執行 `composer test`；純工具型套件可關掉 |
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

`lalalili` 是個人帳號而非組織，沒有組織層 secret，因此需要在**每個 repo**
各自設定 secret `COMPOSER_TOKEN`：

```bash
gh secret set COMPOSER_TOKEN -R lalalili/<repo> --body "<token>"
```

caller workflow 寫 `secrets: inherit` 即可帶入。只依賴公開套件的 repo 不設也能跑。

> 更省事的作法是把這三個套件改為 public —— 它們已經是 MIT 授權，
> `package-testing-support` 更只是一個 Testbench 基底類別。改公開後
> 整個生態的 CI 都不需要任何 token。

## Renovate

各 repo 的 `renovate.json`：

```json
{ "extends": ["github>lalalili/.github"] }
```

## 版本契約

發版前請讀 [SEMVER.md](SEMVER.md)，裡面定義了哪些東西算 public API、
deprecation 流程，以及套件之間的發版相依順序。
