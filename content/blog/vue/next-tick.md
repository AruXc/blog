---
title: 【Vue.js】nextTick 的理解及應用
date: "2019-03-22T12:00:00Z"
description: ""
draft: true
---

 在目前的專案中，因為需要檢核 form 輸入的內容，但我們的 form 是由很多個 component 組成的，所以是由每個 component 自己檢核自己的 data，那問題來了，當我按下 submit 的時候，JavaScript 並不會等我檢核完所有的 component，我需要確認檢核結果才能決定接下來的動作，那該怎麼辦？這就是我們今天的主題: **_nextTick_**。

---

在了解 Vue 提供的 API nextTick 之前最好先了解兩樣東西

1. JavaScript 的運作機制、執行順序
2. Vue 的 Lifecycle

