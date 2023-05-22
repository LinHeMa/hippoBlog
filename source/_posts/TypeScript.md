---
title: TypeScript 5.0 Deopt Explorer
date: 2023-05-22 23:44:54
tags: typescript
---
## Inline Cache
在靜態型別（statically typed）與動態型別（dynamically typed）語言中，透過cache先前的編譯結果，減少runtime以達到優化的方法調用（methods calling ）跟屬性查找（property lookup）。透過遍歷一個物件的原型鍊（prototype chain），並且建立cache，可以在之後對於相同的查找減少查詢時間。補充：javascript VM

透過inline cache，當虛擬機觀察到只有單態（Monomorphic）時，效能會顯著的提升。但是當虛擬機同時觀察到較多型態（Polymorphic、Megamorphic）因為覆寫、重新比較的關係，因此會導致cache的功效開始降低。

Megamorphic
在TypeScript中，大多數的型別都是超多態（Megamorphic），由於程式語言的樹狀結構，理想的情況是進入了越來越細的節點，型態由多態慢慢變成單態進而可以透過cache的建立，減少每次的runtime。

但是現實中總是有特例，由於JavaScript語言的特性，即時是由同一個constructor建構出來的兩個objects，仍然可能是兩種截然不同的型別（例如：初始化的順序不同、有條件的初始化），導致很常發生非預期的Polymorphic。

### Example 1 : 因為順序而導致的多態性
```typescript
function f(x, y) {
  if (x <= y) {
    return { x, y };
  }
  else {
    return { y, x };
  }
}

function g(p) {
  const x = p.x; // polymorphic
}

g(f(0, 1));
g(f(1, 0));
```

> 因為x, y的不同，導致屬性順序的不同，無意間造成了多態。

### Example 2 : 因爲條件初始化、賦值而導致的多態
```typescript
function f(x, y) {
  const p = {};
  p.x = x;
  if (y !== undefined) {
    p.y = y;
  }
  return p;
}

function g(p) {
  const x = p.x; // polymorphic
}

g(f(1));
g(f(1, 0));
```
> y的存有是透過y的值條件賦予的，這樣的行為也會導致多態。

## 識別Megamorphic
在一個龐大的JavaScript程式碼library中，判斷是否具有Megamorphic以及判斷該Megamorphic是否對於效能有負面的影響是困難的事，因為效率以及機會成本，我們應該選擇優先優化真正對於整體運行效率產生負面影響的程式碼。

## Deopt Explorer
使用Deopt Explorer等工具能夠幫助開發人員分析和解決Megamorphic問題。透過可視化的圖表和分析V8引擎的去優化程式碼內部的關聯情況、incline caches，我們能夠深入了解程式碼中超多態性問題的根源，並進行必要的優化。


