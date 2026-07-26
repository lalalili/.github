# lalalili 套件生態

給 Laravel 13 + Filament 5 宿主應用使用的可重用套件集合。
所有套件以 **GitHub VCS + semver tag** 消費，宿主端用 caret 約束（`^1.0`）。

發版規則見 [SEMVER.md](https://github.com/lalalili/.github/blob/main/SEMVER.md)。

## 電商

| 套件 | 用途 |
|---|---|
| [commerce-core](https://github.com/lalalili/commerce-core) | 商品、訂單、發票、付款紀錄、權益的核心領域層 |
| [commerce-kit](https://github.com/lalalili/commerce-kit) | 把 cart / discount / commerce-core 串起來的整合膠水層 |
| [commerce-payment](https://github.com/lalalili/commerce-payment) | 台灣金流整合（綠界信用卡／銀聯、玉山），含對帳 |
| [discount](https://github.com/lalalili/discount) | 設定驅動的折扣與優惠券計算核心，不綁宿主 model |
| [laravelshoppingcart](https://github.com/lalalili/laravelshoppingcart) | 具名購物車實例、儲存轉接層與購物車事件 |
| [campaign-kit](https://github.com/lalalili/campaign-kit) · [-filament](https://github.com/lalalili/campaign-kit-filament) | 活動頁與 GA4 追蹤 |
| [filament-product-binding](https://github.com/lalalili/filament-product-binding) | Filament / Livewire 商品綁定工作區 |

## 課程與訂閱

| 套件 | 用途 |
|---|---|
| [course-core](https://github.com/lalalili/course-core) · [-filament](https://github.com/lalalili/course-filament) | 線上課程領域層與後台 UI |
| [course-commerce](https://github.com/lalalili/course-commerce) | 課程與 commerce-core 的權益／商品對接 |
| [video-upload](https://github.com/lalalili/video-upload) | 影片上傳 session 與供應商狀態生命週期 |
| [subscription-core](https://github.com/lalalili/subscription-core) · [-filament](https://github.com/lalalili/subscription-filament) | 訂閱制領域層與後台 UI |

## 問卷、名單與行銷

| 套件 | 用途 |
|---|---|
| [survey-core](https://github.com/lalalili/survey-core) · [-filament](https://github.com/lalalili/survey-filament) | 問卷引擎（題型、跳題邏輯、分析）與後台 UI |
| [email-campaign](https://github.com/lalalili/email-campaign) · [-filament](https://github.com/lalalili/email-campaign-filament) | EDM 引擎：模板、變數、寄送與事件記錄 |
| [audience-core](https://github.com/lalalili/audience-core) | 共用名單領域層（AudienceList、匯入批次） |
| [marketing-automation](https://github.com/lalalili/marketing-automation) | 名單分群引擎與多渠道自動化派送 |

## 平台共用

| 套件 | 用途 |
|---|---|
| [report-queue](https://github.com/lalalili/report-queue) | 非同步匯出佇列 + 限時下載中心 |
| [media-manager](https://github.com/lalalili/media-manager) | Filament / Livewire 媒體管理器 |
| [filament-upload-center](https://github.com/lalalili/filament-upload-center) | 瀏覽器直傳 S3 的上傳佇列 UI（Uppy） |
| [text-to-speech](https://github.com/lalalili/text-to-speech) | 語音合成 driver |
| [builder-ui-core](https://github.com/lalalili/builder-ui-core) | 共用前端建構器元件（Vue，規則樹） |
| [package-testing-support](https://github.com/lalalili/package-testing-support) | 各套件共用的 Orchestra Testbench 基底 |

## 消費方式

```jsonc
{
  "repositories": [
    { "type": "vcs", "url": "https://github.com/lalalili/commerce-core.git" }
  ],
  "require": {
    "lalalili/commerce-core": "^1.0"
  }
}
```

不要把套件原始碼 commit 進宿主的 `packages/`，也不要用 `path` repository
搭配硬釘 `versions` 消費 —— 那會讓宿主停在舊版而毫無察覺，且套件本身無法在
CI 上獨立解析依賴。本機跨套件開發時可暫時用 path override，完成後務必還原。
