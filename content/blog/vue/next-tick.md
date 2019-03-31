---
title: 【Vue.js】nextTick 的理解及應用
date: "2019-03-31T12:00:00Z"
description: ""
draft: false
---

 在目前的專案中，因為需要檢核 form 輸入的內容，但我們的 form 是由很多個 component 組成的，所以是由每個 component 自己檢核自己的 data，那問題來了，當我按下 submit 的時候，Vue 並不會等我檢核完所有的 component，我需要確認檢核結果才能決定接下來的動作，那該怎麼辦？這就是我們今天的主題: **_nextTick_**。

---

在了解 Vue 提供的 API nextTick 之前最好先了解兩樣東西

JavaScript:
1. <a href="https://blog.aruxc.com/javascript/execution_order/" target="_blank" title="javascript">理解執行順序</a>

Vue.js:
1. <a href="https://vuejs.org/v2/guide/instance.html#Instance-Lifecycle-Hooks" target="_blank" title="vue-lifecycle">Instance Lifecycle Hooks</a>
2. <a href="https://vuejs.org/v2/api/#vm-nextTick" target="_blank" title="vue-lifecycle">nextTick API</a> 

執行順序可以參考我的文章，至於 Vue lifecycle 我覺得官方文件已經寫的很清楚了，Vue的文件都寫的很詳細，對於初學者來說非常友善，建議要花時間將官方文件研讀過一次。

這次遇到的狀況是當表單按下 submit 時由表單各自的 input filed 自己驗證自己的 value，然而因為各個 input 實際上是各自獨立的 component，而 component 監聽 submit 去觸發 validate function，然後都驗證完 form 才 return 驗證結果是否通過，所以這明顯會遇上一個問題，form 整體的父 component 該如何等待子 component 完成他們各自的任務才繼續執行接下來的 code?


```javascript
  ...
  this.$store.commit('START_VALIDATE')

  return this.$store.state.invalid
  ...
```

code 大致上長這樣，這樣的結果會變成當子 component 開始驗證時因為這邊是同步 code 的關係，還沒有驗證完就已經 return 結果了，所以這邊我們就得利用 Vue 的 nextTick。

Vue 官方的定義是:

- 在下次 DOM 更新循環結束之後執行 callback。在修改數據之後立即使用這個方法，獲取更新之後的 DOM。

這邊引用官方的範例:


```javascript
Vue.component('example', {
  template: '<span>{{ message }}</span>',
  data: function () {
    return {
      message: '没有更新'
    }
  },
  methods: {
    updateMessage: function () {
      this.message = '更新完成'
      console.log(this.$el.textContent) // => '没有更新'
      this.$nextTick(function () {
        console.log(this.$el.textContent) // => '更新完成'
      })
    }
  }
})
```

Vue 並不會因為你更改 data 之後立刻重新 render DOM 而是在 event loop 清空之後的下一個'tick'更新，這邊的 tick 意思就是我們執行一次的 macrotask 開始下一輪 event loop 所以就是 microtask 已經都被清空 且 DOM 也已完成更新。

回到剛剛我們觸發的驗證 function 是用 watch 去監聽 Vuex 裡面的一個值是否有發生變化，有的話即開始驗證，watch 會將任務緩存到一個異步佇列中，就是我們所說的 microtask，所以在本輪的 event loop 會把我們所有的子 component 的任務都執行完才更新 DOM，那我們所遇到的問題的解法就揭曉了，我們只需要想辦法在本輪 event loop 更新完 DOM 的時候才 return 驗證結果不就好了？

翻了翻 Vue 官方文件發現了這個用法，這個用法省去了許多 code，也非常直觀。

```javascript
await this.nextTick()
```

簡直完美的解法!如同官方的定義所說在下次 DOM 更新循環結束之後執行 callback，但我沒有要執行 callback 且如果是一般的寫法像這樣: 

```javascript
this.nextTick(function() {
  return ...
})
```

promise return 出來的還是 promise 也因為是異步的關係並不會等我執行完，我只是要利用 nextTick 來等待任務完成。

```javascript
...
this.$store.commit('START_VALIDATE')

await this.nextTick()

return this.$store.state.invalid
...
```

這樣就會順利的等到子 component 驗證完才 return 結果，當然也要注意要使用 ES6 的 await 這個 function 前記得要加上 async 哦! 

###參考資料###
- https://vuejs.org/v2/api/#vm-nextTick
- https://ustbhuangyi.github.io/vue-analysis/reactive/next-tick.html