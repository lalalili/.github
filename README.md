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
| `matrix-exclude` | `[]` | 要排除的矩陣組合，例：`'[{"php":"8.2","laravel":"13.*"}]'` |
| `php-extensions` | `dom, mbstring, pdo_sqlite, zip` | setup-php 擴充套件 |
| `run-validate` | `true` | 執行 `composer validate --strict` |
| `run-test` | `true` | 執行 `composer test`；純工具型套件可關掉 |
| `run-analyse` | `true` | 執行 `composer analyse` |
| `run-format` | `false` | 執行 `composer format -- --test`；需要套件已提供 Pint |
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

**目前全部 `lalalili/*` 套件都是 public，CI 不需要任何 token。**

`audience-core`、`marketing-automation`、`package-testing-support` 原本是私有
repo，導致依賴它們的 6 個套件無法在 CI 上解析依賴、長期沒有任何自動化測試。
2026-07-27 起改為 public，問題消失。

若日後有套件需要轉回私有，依賴它的 repo 就得各自設定 secret（`lalalili`
是個人帳號，沒有組織層 secret 可用）：

```bash
gh secret set COMPOSER_TOKEN -R lalalili/<repo> --body "<token>"
```

reusable workflow 已經支援 `composer-token` secret，caller 寫
`secrets: inherit` 即可帶入。

## 服務容器

`php-package-ci` 與 `php-package-release` 一律提供 Redis（`REDIS_HOST=127.0.0.1`）。
GitHub Actions 的 `services` 無法用條件式省略，統一提供的成本只有幾秒啟動時間，
換來佇列／快取相關測試不會因為缺少服務而紅燈。

## Renovate

各 repo 的 `renovate.json`：

```json
{ "extends": ["github>lalalili/.github"] }
```

## 版本契約

發版前請讀 [SEMVER.md](SEMVER.md)，裡面定義了哪些東西算 public API、
deprecation 流程，以及套件之間的發版相依順序。
