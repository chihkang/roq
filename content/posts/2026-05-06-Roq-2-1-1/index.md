---
title: "Roq 2.1.1"
description: >-
  Roq 2.1.1 支援 GitHub Flavored Markdown Alert Blocks，Markdown 裡就能直接寫出提示、警告與注意事項。
tags: code, chat
toc: true
escape: true
---
Roq 更新到 2.1.1，這次我最有感的是 **GFM Alert Blocks**。

簡單說就是可以在 Markdown 裡直接寫 GitHub 那種提示框，不用再自己塞 HTML，也不用為了一段提醒額外發明格式。

```markdown
> [!NOTE]
> 這是一段補充說明。
```

> [!NOTE]
> 升到 Roq 2.1.1 之後，標準的 Alert Blocks 可以直接在 Markdown 裡使用。

## 支援的類型

目前內建五種：

| 類型 | 適合放什麼 |
|---|---|
| `NOTE` | 補充資訊 |
| `TIP` | 小技巧 |
| `IMPORTANT` | 重要提醒 |
| `WARNING` | 可能踩雷的地方 |
| `CAUTION` | 需要特別小心的風險 |

我覺得這個功能很適合寫技術筆記，因為很多內容其實不需要拉成一整段，只是想讓讀者掃過時知道「這句要看一下」。

## 自訂類型

如果五種不夠，也可以在 `application.properties` 裡加自己的類型：

```properties
quarkus.qute.web.markdown.plugin.alerts.custom-types.BUG=Known Bug
quarkus.qute.web.markdown.plugin.alerts.custom-types.SECURITY=Security Notice
```

接著 Markdown 就可以這樣寫：

```markdown
> [!BUG]
> 這是一個已知問題。
```

不過自訂類型預設只有基本結構，顏色與 icon 還是要自己在 CSS 補上。

## 小結

這次更新不算大，但很實用。

如果你的文章常常需要「提醒、補充、警告」這種區塊，Roq 2.1.1 讓格式乾淨很多，Markdown 也比較像真的內容，而不是混著一堆樣式細節。

參考：[GFM Alert Blocks: Styled Callouts in Your Markdown](https://iamroq.dev/posts/gfm-alert-blocks-styled-callouts-in-your-markdown/)
