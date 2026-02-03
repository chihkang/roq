---
title: "Swift data"
description: >-
  利用swift data針對資料處理
tags: ios, code, chat
toc: true
escape: true
---
現在把使用者資料存在本地，不再存在DB。

## Swift Data

簡化相關資料操作，本地儲存相關數量和成本，利用API查詢價格

查詢價格一次可以多筆資料，每一次買入/賣出/替換都是事件

相關資料的異動是由事件累積

## Further
- [x] 目前相關事件的介面不夠完善(字會擠在一起)
- [ ] 持股頁面的排版佈局還有優化空間
- [x] dark mode support (switch)
- [x] 右上方的重新整理(旋轉固定秒數即可，遇到查不到資料應該顯示toast訊息提示)
- [ ] API 如果暫時失效應該要能使用先前的數據(研究本地cache股價的時間)
- [ ] 檢視既有參數，刪除不必要的參數(程式)
- [ ] 更多>趨勢頁面，有閃退問題