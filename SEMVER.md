# lalalili 套件語意化版本契約

所有 `lalalili/*` 套件遵循 [Semantic Versioning 2.0.0](https://semver.org/lang/zh-TW/)。
這份文件定義**什麼算 public API** —— 沒有這個定義，版本號只是數字。

## 版本號語意

| 變更 | 版號 | 例 |
|---|---|---|
| 破壞 public API | **MAJOR** | 移除 contract method、改必填參數、改 config key 名稱 |
| 新增功能且向下相容 | **MINOR** | 新增 contract 的預設實作、新增選填 config key、新增 event |
| 修 bug 且向下相容 | **PATCH** | 修正計算錯誤、補型別標註、效能改善 |

宿主端一律用 caret 約束消費：`"lalalili/commerce-core": "^1.0"`。

> **為什麼 0.x 不夠用**：Composer 對 `^0.1.1` 的解讀是 `>=0.1.1 <0.2.0`。
> 0.x 套件每發一個 minor，所有宿主都得手動改 `composer.json`，否則
> `composer update` 永遠拿不到新版。本生態就曾因此讓 `survey-core`
> 領先宿主 78 個 commit 而無人察覺。

## 算 public API（改動需要 MAJOR）

- `src/Contracts/` 下所有介面的方法簽章
- `src/Services/`、`src/Actions/` 的 **public** 方法簽章與回傳型別
- `src/Events/`、`src/Listeners/` 的事件類別名稱與 public 屬性
- `src/Models/` 的類別名稱、關聯方法名稱、`$fillable` / `$casts` 的語意
- `config/*.php` 的 **key 名稱與型別**（值的預設可在 MINOR 調整）
- 套件發布的 route name 與 Blade view 名稱
- Artisan 指令的 signature
- `composer.json` 中 `php` 與框架版本約束的**下限提高**

## 不算 public API（可在 MINOR / PATCH 改動）

- `src/Support/`、`src/Internal/` 下的一切，除非在 README 明確標示為公開
- `protected` / `private` 方法與屬性
- migration 檔案的內部實作（欄位一旦上線就受保護，但寫法可改）
- 測試、CI 設定、文件
- 依賴套件的 patch / minor 升級

## Deprecation 流程

1. 在要淘汰的成員上加 `@deprecated` PHPDoc，寫明替代方案與預計移除的 MAJOR 版本
2. 在該 MINOR 版的 `CHANGELOG.md` 記錄
3. 保留至少一個 MINOR 週期
4. 下一個 MAJOR 才移除，並在 `UPGRADE.md` 寫遷移步驟

## Release checklist

- [ ] `composer test` 與 `composer analyse` 綠燈
- [ ] `CHANGELOG.md` 已新增本版條目
- [ ] MAJOR 版另外更新 `UPGRADE.md`
- [ ] 打 `vX.Y.Z` tag 後由 `php-package-release.yml` 自動驗證並建立 Release
- [ ] 跨套件連動改版時，依相依順序發版（被依賴者先發）

## 相依順序

發版時被依賴的套件要先發。目前生態的拓撲順序：

```
package-testing-support
  └─ audience-core
       ├─ survey-core ⇄ email-campaign        （雙向整合，需協同發版）
       │    ├─ survey-filament
       │    └─ email-campaign-filament
       └─ marketing-automation

commerce-core、discount、laravelshoppingcart
  └─ commerce-kit
commerce-core
  └─ course-commerce
course-core
  ├─ course-filament
  └─ video-upload
subscription-core
  └─ subscription-filament
```

`survey-core` 與 `email-campaign` 互相依賴（前者有 `src/Integrations/EmailCampaign/`，
後者有 `src/Listeners/HandleSurveyInvitationDispatched.php`）。兩者都設了
`extra.branch-alias`，讓 Composer 在 dep-on-root 情境下仍能解析；發版時需要
兩邊一起推、依序打 tag，中間會有短暫的 CI 紅燈直到 tag 補齊。
